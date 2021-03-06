# 2 Redis Web应用剖析

本章，我们要解决网上零售商店的一些问题，假设这个网店每天接收来自500万独立用户的1亿次点击，购买超过10万件商品。如果能解决这个大规模的问题，规模更小的会更容易，尽管数据量巨大，但是大部分都能使用1台Redis服务器处理，内存不超过几G。

## 2.1 登录与cookie缓存

有两种常用的方法用于在cookies中存储登录信息：a signed cookie or a token cookie。 _Signed cookies_ 主要存储用户名，也可能是用户ID，还有其他有用的信息，当然包括一个签名，服务器通过签名来确认浏览器发送的信息没有被修改。 _Token cookies_ 使用随机串，服务器通过这个串从数据库中查询对应的用户。随着时间的推移，旧的tokens可以被删除，下面是两种方式的利弊：

![](/assets/QQ20160804-1@2x.png)

我们使用token cookie，在关系数据库表中存用户登录信息，还可以存用户浏览了多长时间，浏览过多少商品等。但是随着用户的浏览会导致大量的数据库写操作，大部分关系数据库会遇到单台服务器每秒插入、更新和删除200~2000行的瓶颈。尽管可以使用批量操作提升性能，但是每次浏览只会有少数几行更新，所以不管用。

现在，由于相对高的负载（平均每秒1200次写，峰值6000次写），我们得用10台数据库服务器来支撑，我们用Redis来替换。

我们使用一个HASH来存储token到登录用户的映射，下面是获取登录用户的函数：

```
def check_token(conn, token):
    return conn.hget('login:', token)
```

更新token，首先将token映射到登录用户，然后记录最近的token，如果用户在访问一个商品，就添加到用户的最近访问ZSET，并保留最近的25条：

```
def update_token(conn, token, user, item=None):
    timestamp = time.time()
    conn.hset('login:', token, user)
    conn.zadd('recent:', token, timestamp)
    if item:
        conn.zadd('viewed:' + token, item, timestamp)
        conn.zremrangebyrank('viewed:' + token, 0, -26)
```

随着时间推移，内存占用越来越多，所以我们想要清理旧数据，我们只保存最近的1千万个session：

```
QUIT = False
LIMIT = 10000000

def clean_sessions(conn):
    while not QUIT:
        size = conn.zcard('recent:')
        if size <= LIMIT:
            time.sleep(1)
            continue

        end_index = min(size - LIMIT, 100)
        tokens = conn.zrange('recent:', 0, end_index - 1)

        session_keys = []
        for token in tokens:
            session_keys.append('viewed:' + token)

        conn.delete(*session_keys)
        conn.hdel('login:', *tokens)
        conn.zrem('recent:', *tokens)
```

这么简单的东西如何能处理每天500万用户？假设每天500万独立用户，那么两天时间，我们就需要清理tokens了，平均每秒不超过58个新session，如果我们每秒运行一下这个清理函数，可以清理最多100个，事实上每秒可以清理上万个session，比我们需要的快数百倍。

## 2.2 购物车

我们只需要用HASH来映射item id和对应的数量就行了：

```
def add_to_cart(conn, session, item, count):
    if count <= 0:
        conn.hrem('cart:' + session, item)
    else:
        conn.hset('cart:' + session, item, count)
```

更新上面的session清理函数，只要加一句：

```
session_keys.append('cart:' + token)
```

## 2.3 页面缓存

虽然可以用模板语言生成动态的页面，但是通过观察，我们的网站95%的页面每天最多改变一次，因此事实上不需要动态生成，几乎所有web应用框架都提供请求处理之前和之后的拦截器，在这一层我们可以加上缓存：如果不能被cache，生成页面并返回，否则我们从缓存中获取，如果缓存中未找到，则生成页面并缓存5分钟。

```
def cache_request(conn, request, callback): 
    if not can_cache(conn, request):
        return callback(request)

    page_key = 'cache:' + hash_request(request)
    content = conn.get(page_key)

    if not content:
        content = callback(request)
        conn.setex(page_key, content, 300)

    return content
```

## 2.4 数据库行缓存

![](/assets/QQ20160814-1.png)

使用两个ZSET，一个是schedule，row\_id作为成员，分数是时间戳，表示下一次该行需要拷贝到Redis的时间，另一个是delay，row\_id作为成员，分数是缓存更新之间等待秒数。

```
def schedule_row_cache(conn, row_id, delay):
    conn.zadd('delay:', row_id, delay)
    conn.zadd('schedule:', row_id, time.time())
```

我们不停取出schedule集合中的第一条，如果为空或者时间没到，等待50毫秒后重试，否则该条就需要更新到Redis，获取delay，如果delay小于等于0，表示该行不需要被缓存了，删除之，否则，从数据库中获取该行，更新缓存并设置下次更新时间为now + delay。

```
def cache_rows(conn):
    while not QUIT:
        next = conn.zrange('schedule:', 0, 0, withscores=True)
        now = time.time()
        if not next or next[0][1] > now:
            time.sleep(.05)
            continue

        row_id = next[0][0]
        delay = conn.zscore('delay:', row_id)
        if delay <= 0:
            conn.zrem('delay:', row_id)
            conn.zrem('schedule:', row_id)
            conn.delete('inv:' + row_id)
            continue

        row = Inventory.get(row_id)
        conn.zadd('schedule:', row_id, now + delay)
        conn.set('inv:' + row_id, json.dumps(row.to_dict()))
```

## 2.5 网页分析

我们在2.1节中的update\_token方法最后加入一行：

```
conn.zincrby('viewed:', item, -1)
```

这样我们就能记录每个商品被浏览的次数，且按照次数从多到少排序。我们只保留前20000条，并定期把计数除以2。

```
def rescale_viewed(conn):
    while not QUIT:
        conn.zremrangebyrank('viewed:', 20000, -1)
        conn.zinterstore('viewed:', {'viewed:': .5})
        time.sleep(300)
```

现在我们更新下can_cache方法：

```
def can_cache(conn, request):
    item_id = extract_item_id(request)
    if not item_id or is_dynamic(request):
        return False
    rank = conn.zrank('viewed:', item_id)
    return rank is not None and rank < 10000
```
