# 3 Redis命令

## 3.1 Strings

Redis中，String用来存储字符串、整数和浮点数，数值类型可以被增减：

**Table 3.1** **Increment and decrement commands in Redis**

| **Command** | **Example use and description** |
| :--- | :--- |
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







