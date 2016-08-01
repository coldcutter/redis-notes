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

![](/assets/QQ20160801-3.png)

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







