# 3 Redis命令

## 3.1 Strings

Redis中，String用来存储字符串、整数和浮点数，数值类型可以被增减：

**Table 3.1** **Increment and decrement commands in Redis**

| **Command** | **Example use and description** |
| --- | --- |
| INCR | INCR key-name—Increments the value stored at the key by 1 |
| DECR | DECR key-name—Decrements the value stored at the key by 1 |
| INCRBY | INCRBY key-name amount—Increments the value stored at the key by the provided integer value |
| DECRBY | DECRBY key-name amount—Decrements the value stored at the key by the provided integer value |
| INCRBYFLOAT | INCRBYFLOAT key-name amount—Increments the value stored at the key by the provided float value \(available in Redis 2.6 and later\) |

只要值可以被解析成10进制的数值，就可以做加减操作，如果key不存在或者是空串，Redis把它的值当做0来操作，如果值不是数值，会报错。

![](/assets/QQ20160818-1.png)

当使用SETRANGE和SETBIT写字符串时，如果字符串不够长，Redis会在更新之前自动用nulls扩充字符串。使用GETRANGE读字符串时，超过字符串末尾的不会返回，但使用GETBIT读取位时，超过末尾的被当做0。

![](/assets/QQ20160818-2.png)

## 3.2 Lists

![](/assets/QQ20160818-3.png)

![](/assets/QQ20160818-4.png)

![](/assets/QQ20160818-5.png)

## 3.3 Sets

![](/assets/QQ20160819-1.png)

![](/assets/QQ20160819-2.png)

![](/assets/QQ20160819-3.png)

![](/assets/QQ20160819-4.png)

## 3.4 Hashes

![](/assets/QQ20160819-5.png)

![](/assets/QQ20160819-6.png)

## 3.5 Sorted sets

![](/assets/QQ20160822-1.png)

![](/assets/QQ20160822-2.png)

## 3.6 发布\/订阅

![](/assets/QQ20160822-3.png)

## 3.7 其他命令

### 3.7.1 排序

SORT可以排序LISTs，SETs和ZSETs中的数据，甚至是存储在HASHes中的数据。

![](/assets/QQ20160822-4.png)

![](/assets/QQ20160822-5.png)

### 3.7.2 Basic Redis transactions

对于涉及多个keys的操作，Redis提供5种命令帮助我们进行没有中断的操作：WATCH, MULTI, EXEC, UNWATCH, DISCARD。Redis基本事务的意思就是当一个客户端执行多个操作A，B，C，...的时候，其他客户端不能中断它们。这不同于关系数据库事务，可以部分执行，之后回滚或者提交。Redis中，每个基本MULTI\/EXEC事务中的命令顺序执行直到完成，完成后，其他客户端才可以执行它们的命令。

### 3.7.3 Expiring keys

![](/assets/QQ20160822-6.png)







