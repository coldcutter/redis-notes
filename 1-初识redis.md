# 1 初识Redis

## 1.1 Redis是啥？

Redis是一个高性能非关系内存数据库，存储键到5种不同类型值的映射。其他特性包括副本（提高读性能），持久化（到磁盘）以及客户端分片（sharding，提高写性能），能方便地扩展Redis来处理数百GB数据和每秒百万级请求。

### 1.1.1 比较其他数据库

Redis经常被拿来与memcached（高性能key-value缓存服务器）作比较，和memcached一样，Redis也可以存储键值映射，甚至能达到和memcached类似的性能。但相似性到此为止，Redis支持两种不同方式自动写数据到磁盘，并且除了字符串类型，比memcached多4种数据结构，因此也就使得Redis能解决更多问题，既能被用来做主数据库，也能做其他存储系统的辅助数据库。

![](/assets/QQ20160727-1.png)

### 1.1.2 其他特性

当使用像Redis这样的内存数据库时，第一个问题就是：关掉服务器会发生什么？Redis提供两种不同的方式把内存数据写到磁盘。第一种是时间点倾倒（point-in-time dump），特定条件满足（一段时间内多少次写入），或者是两种dump-to-disk命令之一被调用。另一种方法使用一个追加（append-only）文件，把修改数据的命令写入磁盘，你可以配置成从不同步，每秒同步一次或者每次操作结束时同步。

为了提高读性能（同时为了在Redis服务器崩溃时提供容错），Redis支持主\/从复制，slaves连接到master且接收一个全库初始拷贝，之后当在master上执行写操作时，写操作会发往所有连接的slaves，即时修改它们的数据集，这样clients就可以连接任意slave读数据而不必请求master。

### 1.1.3 为什么使用Redis？

如果你之前用过memcached，你可以使用APPEND把数据添加到已存在的字符串末尾，memcached文档说你可以使用APPEND来维护列表。你把项添加到字符串末尾，把它当做列表，但是删除呢？用Redis，你可以直接使用LIST或SET。

数据库插入操作（写到磁盘文件末尾）是比较快的，但是更新一行（随机读写）是相当慢的，但在Redis中，因为数据在内存，所以更新很快（使用原子的INCR命令），而且查询不需要经过查询解析器\/优化器。

Redis非常适合解决实时性问题。

如何安装可参考附录A。

## 1.2 Redis数据结构

![](/assets/QQ20160729-1.png)

### 1.2.1 Strings in Redis

![](/assets/QQ20160801-2.png)

常用命令：GET、SET、DEL：

```
$ redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
```

### 1.2.2 Lists in Redis

![](/assets/QQ20160801-4.png)

常用命令：LPUSH/RPUSH、LPOP/RPOP、LINDEX、LRANGE

```
127.0.0.1:6379> rpush list-key item
(integer) 1
127.0.0.1:6379> rpush list-key item2
(integer) 2
127.0.0.1:6379> rpush list-key item
(integer) 3
127.0.0.1:6379> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"
127.0.0.1:6379> lindex list-key 1
"item2"
127.0.0.1:6379> lpop list-key
"item"
127.0.0.1:6379> lrange list-key 0 -1
1) "item2"
2) "item"
```

### 1.2.3 Sets

![](/assets/QQ20160801-5.png)

常用命令：SADD、SREM、SISMEMBER、SMEMBERS

```
127.0.0.1:6379> sadd set-key item
(integer) 1
127.0.0.1:6379> sadd set-key item2
(integer) 1
127.0.0.1:6379> sadd set-key item3
(integer) 1
127.0.0.1:6379> sadd set-key item
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item"
2) "item2"
3) "item3"
127.0.0.1:6379> sismember set-key item4
(integer) 0
127.0.0.1:6379> sismember set-key item
(integer) 1
127.0.0.1:6379> srem set-key item2
(integer) 1
127.0.0.1:6379> srem set-key item2
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item"
2) "item3"
```

### 1.2.4 Hashes

HASHes存储键值对映射，可存储的值与STRINGs可存储的值相同：strings，或者如果值能表示成数字，则值可被增加或减少。

![](/assets/QQ20160801-6.png)

常用命令：HSET、HGET、HGETALL、HDEL

```
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 1
127.0.0.1:6379> hset hash-key sub-key2 value2
(integer) 1
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 0
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
127.0.0.1:6379> hdel hash-key sub-key2
(integer) 1
127.0.0.1:6379> hdel hash-key sub-key2
(integer) 0
127.0.0.1:6379> hget hash-key sub-key1
"value1"
127.0.0.1:6379> hgetall hash-key
1) "sub-key1"
2) "value1"
```

### 1.2.5 Sorted sets

ZSETs同样存储键值对，键（called members）是唯一的，值（called scores）是浮点数。

![](/assets/QQ20160801-7.png)

![](/assets/QQ20160801-8.png)

```
127.0.0.1:6379> zadd zset-key 728 member1
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"
127.0.0.1:6379> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"
127.0.0.1:6379> zrem zset-key member1
(integer) 1
127.0.0.1:6379> zrem zset-key member1
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```

## 1.3 Hello Redis

近年来有许多网站支持投票功能，比如reddit、Stack Overflow等，本节我们会构建一个类似的基于Redis的简单后端。

假设每天有1000篇文章提交，在这1000篇里，我们希望有大概50篇有趣的文章能在top-100列表里维持至少1天，这50篇文章至少会获得200票。

文章的得分会随时间下降，所以我们定义得分是一个文章发布时间的函数，加上一个常数与得票数的乘积，我们使用Unix时间戳，至于常数，一篇文章维持1天（86400秒）需要200票，所以每一票会获得432分。

我们使用hash来存储一片文章：

![](/assets/QQ20160802-1@2x.png)

常用冒号\(:\)作为分隔符，其他常用符号还包括点\(.\)、斜杠\(\/\)、管道符\(\|\)，保持统一就行了。

我们使用两个ZSET来存放文章得分，一个基于分数，一个基于发布时间：

![](/assets/QQ20160802-2@2x.png)

为了防止重复投票，对于每个文章，我们都需要维护一个用户SET，表示哪些用户投票了：

![](/assets/QQ20160802-3@2x.png)

为了充分节约内存，我们假设超过一周时间用户就不能再投票了，分数固定了，因此过一周我们就可以删掉对应文章的投票用户列表了。

投票的流程是这样的：先检查文章是否在一周内发布，如果是，试图将用户加入文章的投票列表，如果用户之前没投过，增加文章的分数432分，最后更新文章的总投票数加1。
