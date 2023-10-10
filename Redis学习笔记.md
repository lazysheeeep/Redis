# Redis学习笔记

<br>

*最近在学redis，整理了一点笔记*  
<br>

---
<br>
<br>

## Redis数据结构介绍
* redis是一个key-value的数据库，key一般是string类型，但是value的类型多种多样

<br>

基础类型
>1.string： hello,world

>2.Hash： {name:"Jack", age:21}

>3.List： [A->B->C->D]

>4.Set： {A,B,C}

>5.SortedSet： {A:1, B:2, C:3}

特殊类型
>6.GEO： {A: (120.5, 30.5) }

>7.BitMap： 0110110101110101011

>8.Hyperlog： 0110110101110101011

<br>

---

<br>

## Redis通用命令
通用指令是部分数据类型的，都可以使用的指令，常见的有：  
* KEYS：查看符合模板的所有key，但是不建议在生产环境使用
* DEL：删除一个指定的key
* EXISTS：判断key是否存在
* EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除
* TTL：查看一个key的剩余有效期

通过help [command] 可以查看一个命令的具体用法

<br>

---

<br>

## String类型
String类型，也就是字符串类型，是Redis中最简单的存储类型。  
其中value字符串可以根据字符串格式不同分成3类：  
* string：普通字符串
* int：整数类型，可以做自增，自减操作
* float：浮点类型，可以做自增，自减操作

不管是那种格式，底层都是字节数组的形式存储，只不过编码方式不同。字符串类型的最大空间不能超过512m

### String类型常见命令
```
·SET:添加或者修改已经存在的一个String类型的键值对

·GET:根据key获取String类型的value

·MSET:批量添加多个String类型的键值对

·MGET:根据多个key获取String类型的value

·INCR:让一个整型的key自增1

·INCRBY:让一个整型的key自增并指定步长，例如:INCRBY k1 2让k1自增2

·INCRBYFLOAT:让一个浮点类型的数字自增并指定步长

·SETNX:添加一个String类型的键值对，前提是该key不存在，若存在，则该命令不会执行

·SETEX:添加一个String类型的键值对，并且指定有效期
```

### key的结构
Redis的key允许有多个单词形成层级结构，多个单词之间用':'隔开，格式如下：
>项目名:业务名:类型:id  

这个格式并非固定，也可以根据自己的需求来删除或添加词条。  
例如我们的项目名称叫 temp，有user和product两种不同类型的数据，我们可以这样定义key：
* user相关的key：temp:user:1
* product相关的key：temp:product:1  

如果Value是一个Java对象，例如一个User对象，则可以将对象序列化为JSON字符串后存储：  

![Alt text](image-1.png)

### 总结
String类型的三种格式：
* 字符串
* int
* float

Redis的key的格式：
* [项目名] : [业务名] : [类型] : id

<br>

---

<br>

## Hash类型
Hash类型又叫散列，其value是一个无序字典  

String结构是将对象序列化为JSON字符串后存储，当其需要修改某个字段时很不方便：  
![Alt text](image-2.png)

Hash结构可以将对象的每个字段独立存储，可以针对单个字段做CRUD
![Alt text](image-3.png)

### Hash常见命令
```
·HSET key filed value：添加或修改hash类型key的field值

·HGET key filed：获取一个hash类型key的field的值

·HMSET：批量添加多个hash类型的key的field值

·HMGET：批量获取hash类型的key的field值

·HGETALL：获取hash类型的key中所有的field和value值

·HKEYS：获取一个hash类型的key中的所有的field

·HVALS：获取一个hash类型的key中所有的value

·HINCRBY：让一个hash类型key的字段自增并指定步长

·HSETENX：添加一个hash类型的key的field值，前提是其不存在，否则不执行该命令
```

<br>

---

<br>

## List类型
Redis中的List类型可以看做一个双向链表结构，既可以支持正向检索也支持反相检索，特征如下：
* 有序
* 元素可以重复
* 插入和删除快
* 查询速度一般
常用来存储一个有序数据，列入：朋友圈点赞列表，列表评论等。

### List常见命令
```
·LPUSH key element...：向列表左侧插入一个或多个元素

·LPOP key：移除并返回列表左侧的第一个元素，没有则返回nil

·RPUSH key element...：向列表右侧插入一个或多个元素

·RPOP key：移除并返回列表右侧的第一个元素，没有则返回nil

·LRANGE key star end：返回一段角标范围内的所有元素

·BLPOP和BRPOP：与LPOP和RPOP类似，只不过在没有元素时等待指定时间内，而不是直接返回nil
```
![Alt text](image-4.png)

### 衍生
>如何用List模拟一个栈？  
入口和出口都在同一边

>如何利用List模拟一个队列？  
入口和出口不在同一边

>如何利用List结构模拟一个阻塞队列？
入口和出口在不同边  
出队时采用BLPOP，BRPOP

<br>

---

<br>

## Set类型
Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：
* 无序
* 元素不可重复
* 查找快
* 支持交集，并集，差集等功能

### Set类型常见命令
```
·SADD key member...：向Set类型的key中添加一个或多个元素

·SREM key member...：移除Set类型的key中的指定元素

·SCARD key：返回Set类型中key的元素个数

·SISMEMBER key member：判断key中是否存在member

·SMEMBERS：获取key中所有元素

·SINTER key1 key2...：求key1与key2的交集

·SDIFF key1 key2...：求key1与key2的差集

·SUNION key1 key2...：求key1与key2的并集
```
![Alt text](image-5.png)

<br>

---

<br>

## SortedSet类型
Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。具有下列特性：  
* 可排序
* 元素不重复
* 查询速度快  

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。

### SortedSet类型常见命令
```
·ZADD key score member：添加一个或多个到sorted set，如果存在则更新其score值

·ZREM key member：删除sorted set中一个指定元素

·ZSCORE key member：获取sorted set中指定元素的score值

·ZRANK key member：获取sorted set中指定元素的排名

·ZCARD key：获取sorted set中元素的个数

·ZCOUNT key min max：统计score值在给定范围内所有元素的个数

·ZINCREBY key increment member：让sorted set中的指定元素自增，步长为increment值

·ZRANGE key min max：按照score排序后，获取指定排名范围内的元素

·ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素

·ZDIFF、ZINTER、ZUNION：求差集，并集，交集
```
<br>

---


暂时只有这么多，晚点完善一下GO-Redis的实战篇
---