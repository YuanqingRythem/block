---
title: Redis笔记
date: 2021-07-09 12:07:09
tags: 学习
---

## Nosql概述

2020年大数据时代，大数据：一般的数据库无法进行分析处理。



1.单机MySQL的年代app----DAL（数据库访问层）----Mysql

90年代，一个基本的网站访问量一般不会太大，单个数据库完全足够！

那个时候，更多的去使用静态网页html，服务器根本没有太大压力

思考：整个网站的瓶颈是多少

* 1.数据量如果太大，一台机器放不下了！

* 2.数据的索引（B+树），一个机器内存也放不下

* 3.访问量（读写混合），一个服务器承受不了

只要你开始出现以上的三种情况之一，你就必须要晋级！



2.Memcached（缓存）+MySQL+垂直拆分（读写分离）

网站80%的情况下都是在读！每次都要去查询数据库的话就十分的麻烦！所以说我们希望减轻服务器的压力，我们可以使用缓存来保证效率

发展过程：优化数据结构和索引——>文件缓存（IO）——>Memcached（当时最热门的技术！）

![image-20200902101526970](/typora-user-images/image-20200902101526970.png)



3.分库分表+水平拆分+MySQL集群

技术和业务在发展的同时，对人的要求也越来越高！

本质：数据库（读，写）

早些年MyISAM：使用表锁（十分影响效率！高并发下就会出现严重的锁问题）

Innodb：行锁

慢慢的就开始使用分库分表来解决写的压力！MySQL在那个年代推出了表分区，这个并没有多少公司使用！

MySQL的集群，三个集群各放三分之一的数据

![image-20200902103551116](/typora-user-images/image-20200902103551116.png)

MySQL等关系型数据库不够用了数据量很多变化很快

MySQL有的使用它来存储一些比较大的文件，博客，图片！数据库表很大，效率就低了！如果有一种数据库来专门处理这种数据，MySQL压力就变得十分小了（研究如何处理这些问题！）大数据的IO压力下，表几乎没法更大！

企业架构：

![image-20200902104858989](/typora-user-images/image-20200902104858989.png)

为什么要用NoSQL！？

用户的个人信息，社交网络，地理位置，用户自己产生的数据，用户日志等等爆发式增长！

这时候我们就需要使用NoSQL数据库的，NoSQL可以很好处理以上的问题

NoSQL=not only SQL（不仅仅是SQL）

关系型数据库：表格，行，列（POI）

泛指非关系型数据库，随着web2.0互联网诞生，传统关系型数据库很难对付2.0时代！尤其是超大规模的高并发的社区！暴露出来很多难以克服的问题，NoSQL在当今大数据环境下十分迅速，且当下必须掌握



很多的数据类型用户的个人信息，社交网络，地理位置，这些数据类型的存储不需要一个固定的格式！不需要多余的操作就可以横向扩展 !Map<String,Object> 使用键值对来控制！

NoSQL特点：

1.方便扩展（数据之间没有关系，很好扩展！）

2.大数据量高性能（Redis一秒可以写八万次，读取是十一万次，NoSQL的缓存记录级的，是一种细粒度的缓存，性能会比较高）

3.数据类型是多样型的！（不需要事先设计数据库！随取随用！如果数据量十分大的表，很多人就无法设计了！）

4.传统RDBMS和NoSQL区别

RDBMS：

* 结构化组织

* SQL

* 数据和关系都存在单独的表中
* 严格的一致性
* 基础的事务

NoSQL

* 不仅仅是数据
* 没有固定的查询语言
* 键值对存储，列存储，文档存储，图形数据库
* 最终一致性
* CAP定理和BASE（异地多活）

3V+3gao

* 高性能（保证用户体验和性能），高可用，高可扩（机器不够了可以扩展机器）
* 海量Volume，多样Variety，实时Velocity

# Redis：

远程字典服务，是一个开源的使用ANSI C语言编写，支持网络，可基于内存亦可持久化的日志型，key-value数据库，并提供多种语言的API，也被人们称为结构化数据库

redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并在此基础上实现了master-slave主从同步

能干吗？

* 内存存储，持久化，内存中是断电即失，所以持久化很重要（rdb，aof）

* 效率高，可用于高速缓存

* 发布订阅系统

* 地图信息分析

* 计时器，计数器（浏览量）

特效

* 多样的数据类型
* 持久化
* 集群
* 事务

mac上安装redis

切换到src文件下

sudo make

sudo make test

sudo make install

redis-server启动redis服务

redis-cli连接redis

首先你得确认安装redis，并且安装成功......

好吧，话不多说，上步骤

1.第一步进入redis的安装目录里面

   命令：cd /Users/barbed/java/redis-3.2.12

   **/Users/barbed/java/redis-3.2.12 这是你的redis的路径，因人而异哈**

2.执行启动

   命令：src/redis-server redis.conf

3.创建客户端，新建一个终端，进入redis的src目录下

   命令：cd /Users/barbed/java/redis-3.2.12／src   

4.执行连接

   命令：redis-cli -h 127.0.0.1 -p 6379

5.输入密码

   命令：auth

6.退出客户端但不退出redis

   命令：exit

7.退出redis

   查看redis的pid 命令：ps aux|grep redis

   结束进程      命令：kill -15 pid

redis-benchmark是性能测试工具

## 基础知识

redis默认有16个数据库

![image-20200903112506843](/typora-user-images/image-20200903112506843.png)

默认使用第0个

使用select 3 切换到第三个数据库

keys *代表查看所有的key

flushdb清除当前数据库

FLUSHALL清除所有数据库

为什么redis的端口号是6379？因为作者是一个女明星的粉丝，而女明星的名字通过手机输入正好是6379

Redis是单线程的！官方表示，redis是基于内存操作，cpu不是redis性能瓶颈，redis的瓶颈是根据机器的内存和网络带宽，既然可以用单线程来实现，就是用单线程了！所以就是用单线程了！

Redis是C语言写的，官方提供的数据为100000+QPS，完全不比同样是使用key-value的memecache差，峰值时间每秒请求数(QPS)

**redis为什么单线程还这么快？**

误区：高性能的服务器一定是多线程的？

误区2：多线程（cpu上下文切换！会消耗资源）一定比单线程的效率高！

CPU>内存>硬盘的速度要有所了解！

核心：redis是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，多线程上下文切换耗时的操作，对于内存系统来说，如果没有上下文切换效率就是最高的！多次读写都是在一个CPU上的，内存情况下，这就是最佳的方案

官网全段翻译： 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

**Redis-key**

expire key 10设置key的过期时间，过期以后就无法get到了

ttl key查看当前key的剩余时间

exists key判断当前key是否存在

move key移除当前key

type key查看当前key的类型

### String类型

set key v1 设置值

get key 获得值

APPEND key “值” 像java一样可以在当前key的值以后追加字符串，如果当前key不存在就相当于setkey

STRLEN key 获取字符串长度



incr key 加一（当前key对应的值加一）

decr key 减一（当前key对应的值减一）

步长

INCRBY key 10 一次增长十个数

DECRBY key 10 一次减少十个数

字符串范围 range

GETRANGE key 0 3 查看当前key对应的字符串从0个看到3个，截取字符串

GETRANGE key 0 -1 获取全部字符串和get key 是一样的

SETRANGE key  1 xx 替换key对应的字符串，替换1位xx

setex （set with expire）设置过期时间

setnx（set if not exist）不存在，再设置（在分布式锁中会常常使用！）

setex key 30 "hello" 设置key的值为hello，30秒后过期

setnx key "redis" 如果key不存在，则创建，若存在则创建失败

mset key1 v1 key2 v2 key3 v3 批量设置多个值

mget key1 key2 key3 批量获取多个值

msetnx key1 v1 key4 v4 如果不存在则创建是个原子性操作，要么一起成功要么一起失败

对象

set user:1 {name:zhangsan,age:3}设置一个user：1的对象，值为json字符串来保存一个对象！

这里的key是个巧妙地设计user:{id}:{filed}，如此设计在redis中是完全OK的

mset user:1:name zhangsan user:1age 2

mget user:1:name user:1:age

getset 先get在set

getset db redis 如果不存在值则返回null，存在值则获取原来的值，并设置新的值

未来的话，jedis与Java连接

### List

基本数据类型，列表

所有的List命令都是用L开头的

LPUSH list one 放入列表的头部

LRANGE list 0 -1 获取全部的值

LRANGE list 0 1 获取从零到一的两个值

Rpush list right 插到队列的最后一个位置，放到列表的尾部

Lpop list 从左面移出一个元素

Rpop list 从右面移出一个元素

Index list 0 通过下标获得某一个值

Llen list 返回列表长度

取关移出固定的值

lrem list 1 one 移出list集合中指定个数的value，精确匹配

通过下标截取制定长度，该list的已经被改变了，只剩下了截取的元素

127.0.0.1:6379> keys *

(empty array)

127.0.0.1:6379> Rpush mylist "hello1"

(integer) 1

127.0.0.1:6379> Rpush mylist "hello2"

(integer) 2

127.0.0.1:6379> Rpush mylist "hello3"

(integer) 3

127.0.0.1:6379> ltrim mylist 1 2

OK

127.0.0.1:6379> Lrange mylist 0 -1

1) "hello2"

2) "hello3"



rpoplpush 移出列表的最后一个元素，并将他移动到新的列表中！

127.0.0.1:6379> rpoplpush mylist myotherlist

"hello3"

127.0.0.1:6379> lrange mylist 0 -1

1) "hello2"

127.0.0.1:6379> lrange myotherlist 0 -1

1) "hello3"



lest 将列表中指定下标的值替换成另一个值，更新操作

如果不存在列表我们去更新就会报错

如果存在，更新当前下标的值

lest list 1 other



将某一个具体的value插入到列表中某个元素的前面或者后面

127.0.0.1:6379> Rpush mylist word

(integer) 2

127.0.0.1:6379> lrange mylist 0 -1

1) "hello2"

2) "word"

127.0.0.1:6379> linsert mylist before word other

(integer) 3

127.0.0.1:6379> lrange mylist 0 -1

1) "hello2"

2) "other"

3) "word"

> 小结

* 它实际上是一个链表，before Node after，left ，right 都可以插入值
* 如果key不存在，创建新的链表
* 如果key存在，新增内容
* 如果移除了所有的值，空链表，也代表不存在!
* 在两边插入或者改动值，效率最高！中间元素，相对来说效率会低一点



### set

set中的值是不能重复的！

127.0.0.1:6379> sadd myset hello           set集合中添加值

(integer) 1

127.0.0.1:6379> sadd myset yuanqing

(integer) 1

127.0.0.1:6379> sadd myset loveyuanqing

(integer) 1

127.0.0.1:6379> SMEMBERS myset 查看指定的set所有值

1) "loveyuanqing"

2) "hello"

3) "yuanqing"

127.0.0.1:6379> SISMEMBER myset hello 判断某一个值是不是在set中

(integer) 1



127.0.0.1:6379> scard myset 获取set集合中的内容元素个数

(integer) 3



127.0.0.1:6379> scard myset

(integer) 3

127.0.0.1:6379> srem myset hello 移除set集合中的指定元素

(integer) 1

127.0.0.1:6379> scard myset

(integer) 2

127.0.0.1:6379> smembers myset

1) "loveyuanqing"

2) "yuanqing"

127.0.0.1:6379> 



set 无需不重复集合，抽随机！

127.0.0.1:6379> SRANDMEMBER myset

"yuanqing"

127.0.0.1:6379> SRANDMEMBER myset

"loveyuanqing"

127.0.0.1:6379> SRANDMEMBER myset 2 随机抽选出指定个数的元素



删除制定的key，随机删除制定的key！

127.0.0.1:6379> smembers myset

1) "Wx"

2) "yuanqing"

3) "loveyuanqing"

127.0.0.1:6379> spop myset

"yuanqing"

127.0.0.1:6379> smembers myset

1) "Wx"

2) "loveyuanqing"



将一个指定的值移到另外一个key中

127.0.0.1:6379> sadd myset hello

(integer) 1

127.0.0.1:6379> sadd myset word

(integer) 1

127.0.0.1:6379> sadd myset yuanqing

(integer) 1

127.0.0.1:6379> sadd myset2 set2

(integer) 1

127.0.0.1:6379> smove myset myset2 yuanqing

(integer) 1

127.0.0.1:6379> smembers myset

1) "word"

2) "hello"

127.0.0.1:6379> smembers myset2

1) "yuanqing"

2) "set2"



微博，b站，共同关注！（并集）共同爱好

数字集合类：

——差集sdiff

——交集sinter

——并集sunion

127.0.0.1:6379> sadd key1 a

(integer) 1

127.0.0.1:6379> sadd key1 b

(integer) 1

127.0.0.1:6379> sadd key1 c

(integer) 1

127.0.0.1:6379> sadd key2 c

(integer) 1

127.0.0.1:6379> sadd key2 d

(integer) 1

127.0.0.1:6379> sdiff key1 key2 差集 

1) "b"

2) "a"

127.0.0.1:6379> sinter key1 key2 交集

1) "c"

127.0.0.1:6379> sunion key1 key2 并集

1) "d"

2) "b"

3) "a"

4) "c"

### Hash（哈希）

Map集合，Key-Map集合!------------>key-<key-value>这时候值对应一个map集合!本质上和String类型没有太大区别，还是一个简单的key-value！

127.0.0.1:6379> hset myhash filed1 yuanqing # set一个具体的key-value

(integer) 1

127.0.0.1:6379> hget myhash filed1 #获取一个key-value

"yuanqing"

127.0.0.1:6379> hmset myhash filed1 hello filed2 word #set多个key-value

OK

127.0.0.1:6379> hmget myhash filed1 filed2 #获取多个 key-value

1) "hello"

2) "word"

127.0.0.1:6379> hgetall myhash #获取全部数据

1) "filed1"

2) "hello"

3) "filed2"

4) "word"

127.0.0.1:6379> hdel myhash filed1 #删除hash制定的key字段！

(integer) 1

127.0.0.1:6379> hgetall myhash

1) "filed2"

2) "word"

127.0.0.1:6379> hlen myhash #获取hash表字段数量

(integer) 1

127.0.0.1:6379> hexists myhash field2 #判断hash中的指定字段是否存在

(integer) 0

127.0.0.1:6379> hkeys myhash # 只获得所有的field

1) "filed2"

127.0.0.1:6379> hvals myhash # 只获得所有的value

1) "word"



127.0.0.1:6379> hset myhash field3 5 #指定增量

(integer) 1

127.0.0.1:6379> hincrby myhash field3 1

(integer) 6

127.0.0.1:6379> hincrby myhash field3 -1

(integer) 5

127.0.0.1:6379> hsetnx myhash field4 hello #如果不存在则可以设置

(integer) 1

127.0.0.1:6379> hsetnx myhash field4 hello #如果存在则不能设置

(integer) 0

hash变更数据，user name age，尤其是用户信息之类的，经常变更的信息！hash更适合于对象的存储，String更加适合字符串存储

### Zset（有序集合）

在set的基础上增加一个值，set k1 v1 zset k1 score v1

127.0.0.1:6379> zadd myset 1 one #添加一个值

(integer) 1

127.0.0.1:6379> zadd myset 2 two 3 three #添加多个值

(integer) 2

127.0.0.1:6379> zrange myset 0 -1  #查看值范围

1) "one"

2) "two"

3) "three"

127.0.0.1:6379> zadd salary 2500 xiaohong #添加三个用户

(integer) 1

127.0.0.1:6379> zadd salary 5000 hanggao

(integer) 1

127.0.0.1:6379> zadd salary 500 yuanqingsese

(integer) 1

127.0.0.1:6379> zrangebyscore salary -inf +inf #从小到大进行排序

1) "yuanqingsese"

2) "xiaohong"

3) "hanggao"

127.0.0.1:6379> zrevrange salary 0 -1 #从大到小进行排序

1) "hanggao"

2) "yuanqingsese"

127.0.0.1:6379> zrangebyscore salary -inf +inf withscores #显示全部用户并附带成绩

1) "yuanqingsese"

2) "500"

3) "xiaohong"

4) "2500"

5) "hanggao"

6) "5000"

127.0.0.1:6379> zrangebyscore salary -inf 2500 withscores #显示工资小于2500员工的降序排列

1) "yuanqingsese"

2) "500"

3) "xiaohong"

4) "2500"

127.0.0.1:6379> zrange salary 0 -1

1) "yuanqingsese"

2) "xiaohong"

3) "hanggao"

127.0.0.1:6379> zrem salary xiaohong #移除有序集合中的指定元素

(integer) 1

127.0.0.1:6379> zrange salary 0 -1

1) "yuanqingsese"

2) "hanggao"

127.0.0.1:6379> zcard salary #获取有序集合中的个数

(integer) 2

127.0.0.1:6379> zadd myset 1 hello

(integer) 1

127.0.0.1:6379> zadd myset 2 word

(integer) 1

127.0.0.1:6379> zcount myset 1 3 #获取指定区间的成员数量

(integer) 2

set的排序版，可以存储班级成绩表，工资表排序！

普通消息1，重要消息2，带权重进行判断！

排行榜应用实现

## 三大数据类型

### geospatial 地理位置

朋友的定位，附近的人，打车距离计算？

Redis的Geo 在Redis3.2版本就推出了！这个功能可以推算地理位置的信息，两地之间的距离，方元几里的人！

getadd 添加地理位置

两极无法直接添加,有效的经度为-180到+180

纬度为-85到+85

参数key  值（纬度，经度，名称）

127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing

(integer) 1

127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai

(integer) 1

127.0.0.1:6379> geoadd china:city 114.05 21.23 dalian 113.23 22.32 liancheng

(integer) 2

获得当前定位：一定是一个坐标值！

127.0.0.1:6379> geopos china:city shanghai

1) 1) "121.47000163793563843"

  2) "31.22999903975783553"

两人之间的距离geodist

- **m** 表示单位为米。
- **km** 表示单位为千米。
- **mi** 表示单位为英里。
- **ft** 表示单位为英尺。

127.0.0.1:6379> geodist china:city beijing shanghai #查看上海到北京的直线距离

"1067378.7564"

127.0.0.1:6379> geodist china:city beijing shanghai km #查看重庆到北京的直线距离

"1067.3788"

我附近的人？（获得所有附近的人的地址，定位！）通过半径来查询！

获得指定数量的人，200

127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km #获取当前位置110 30 下1000km内的数据

1) "liancheng"

127.0.0.1:6379> GEORADIUS china:city 110 30 2000 km #获取当前位置110 30 下2000km内的数据

1) "liancheng"

2) "shanghai"

3) "beijing"

4) "dalian"

127.0.0.1:6379> GEORADIUS china:city 110 30 2000 km withdist #显示到中心距离的位置

1) 1) "liancheng"

  2) "912.9047"

2) 1) "shanghai"

  2) "1105.9098"

3) 1) "beijing"

  2) "1245.2858"

4) 1) "dalian"

  2) "1056.3843"

127.0.0.1:6379> GEORADIUS china:city 110 30 2000 km withcoord #获取当前位置110 30 下2000km内的数据的经度纬度定位信息

1) 1) "liancheng"

  2) 1) "113.22999805212020874"

   2) "22.3200004495938984"

2) 1) "shanghai"

  2) 1) "121.47000163793563843"

   2) "31.22999903975783553"

3) 1) "beijing"

  2) 1) "116.39999896287918091"

   2) "39.90000009167092543"

4) 1) "dalian"

  2) 1) "114.04999762773513794"

   2) "21.22999937888568667"

127.0.0.1:6379> GEORADIUS china:city 110 30 2000 km withcoord count 1 #筛选出指定的用户

1) 1) "liancheng"

  2) 1) "113.22999805212020874"

   2) "22.3200004495938984"

127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1200 km #找出位于指定元素周围的其他元素

1) "beijing"

2) "shanghai"

127.0.0.1:6379> GEOHASH china:city beijing #将二维的经纬度转化为一纬的字符串降纬打击，如果两个字符串越接近，那么距离越近

1) "wx4fbxxfke0"

> GEO 底层实现原理其实就是Zset！我们可以使用Zset命令来操作GEO

127.0.0.1:6379> ZRANGE china:city 0 -1 #查看地图中全部元素

1) "dalian"

2) "liancheng"

3) "shanghai"

4) "beijing"

127.0.0.1:6379> ZREM china:city beijing #移除指定元素！

(integer) 1

### Hyperloglog

> 什么是基数？

A{1,3,5,7,9,7} B{1,3,5,7}

基数（不重复的元素）= 5，可以接受误差

Redis Hyperloglog 基数统计的算法！

一个人访问一个网站多次，但还是算作一个人 

传统的方式，set保存用户的ID，然后就可以统计set中元素数量作为标准判断！

这个方式如果保存大量的用户id，就会比较麻烦！我们的目的是为了计数，而不是保存用户id；

优点：占用内存是固定的，2^64不同的元素的技术，只需要12kb的内存

127.0.0.1:6379> PFADD mykey a b c d e f g h i j #创建第一组元素

(integer) 1

127.0.0.1:6379> PFCOUNT mykey #统计mykey数量

(integer) 10

127.0.0.1:6379> PFADD mykey2 i j k x c v d #创界第二组元素mykey2

(integer) 1

127.0.0.1:6379> PFMERGE mykey3 mykey mykey2 #合并两组mykey mykey2=>mykey3并集

OK

127.0.0.1:6379> PFCOUNT mykey3 #查看并集数量！

(integer) 13

如果允许容错，那么一定可以使用hyperloglog

### Bitmap

> 位运算

统计疫情感染人数

统计用户信息，活跃，不活跃！登录，未登录！两个状态的，都可以使用bitmaps！

位图，数据结构！都是操作二进制位来进行记录，只有0 和 1两个状态！

365=265bit 1字节=8bit 46字节左右！

> 测试

使用bitmap来记录周一到周日的打卡

周一:1 周二:0 周三:1

127.0.0.1:6379> setbit sign 0 1

(integer) 0

127.0.0.1:6379> setbit sign 2 0

(integer) 0

127.0.0.1:6379> setbit sign 3 1

(integer) 0

127.0.0.1:6379> setbit sign 4 1

(integer) 0

127.0.0.1:6379> setbit sign 5 1

(integer) 0

127.0.0.1:6379> setbit sign 6 1

查看某一天是否打卡

127.0.0.1:6379> getbit sign 4

(integer) 1

127.0.0.1:6379> getbit sign 2

(integer) 0

统计打卡的天数

127.0.0.1:6379> bitcount sign 

(integer) 5

### 事务

事务本质：一组命令的集合！一个事务中的所有命令都会被序列化，在事务的执行过程中，会按照顺序执行

要么同时成功，要么同时失败，原子性！

Redis事务没有隔离级别的概念！

所有的命令在事务中并没有被执行！只要发起执行命令的时候才会执行！Exec

Redis单条命令是保证原子性的，redis的事务是不保证原子性！

redis的事务:

* 开启事务（multi）
* 命令入队（）
* 执行事务（exec）

> 正常执行事务！

127.0.0.1:6379> multi

OK

127.0.0.1:6379> set k1 v1

QUEUED

127.0.0.1:6379> set k2 v2

QUEUED

127.0.0.1:6379> get k2

QUEUED

127.0.0.1:6379> exec #执行事务

1) OK

2) OK

3) "v2"

> 取消事务

127.0.0.1:6379> multi #开启事务

OK

127.0.0.1:6379> set k5 v5

QUEUED

127.0.0.1:6379> DISCARD #取消事务

OK

127.0.0.1:6379> get k5

(nil)

> 编译性异常（代码有问题！命令有错！），事务中所有的命令都不会被执行

命令写错了

> 运行时异常（1/0），如果事务队列中存在语法性，那么执行命令的时候，其他命令是可以正常执行的

127.0.0.1:6379> set k1 "v1"

OK

127.0.0.1:6379> multi

OK

127.0.0.1:6379> incr k1

QUEUED

127.0.0.1:6379> set k2 v2

QUEUED

127.0.0.1:6379> exec

1) (error) ERR value is not an integer or out of range #虽然第一条命令报错了，但依旧正常执行成功了

2) OK

> 监控！Watch（面试常问）

悲观锁：

* 很悲观，认为什么时候都会出问题，无论做什么都会加锁

乐观锁

* 很乐观，认为什么时候都不会出现问题，所以不会上锁！更新数据的时候去判断一下，在此期间是否有人修改过这个数据，version！

* 获取version
* 更新的时候比较version

127.0.0.1:6379> set moeny 100

OK

127.0.0.1:6379> set out 0

OK

127.0.0.1:6379> WATCH money #监视moeny对象

OK

127.0.0.1:6379> MULTI #事务正常结束，数据期间没有发生变动，这个时候就能正常执行成功

OK

127.0.0.1:6379> DECRBY moeny 20

QUEUED

127.0.0.1:6379> INCRBY out 20

QUEUED

127.0.0.1:6379> EXEC

1) (integer) 80

2) (integer) 20

如果同时有多个线程操作一个数据，通过watch监测money字段，若事执行失败，就先解锁unwatch，获取最新的值，再次监视（在此获取最新的version），事务结束，比对监视的值是否发生了变化，如果没有变化则执行成功，如果改变则执行失败，获取最新值就好

## Jedis

使用java语言来操作redis

> 是redis官方推荐java连接和开发的工具！使用java操作redis的中间件

## springboot整合

springboot操作数据：spring-data jpa jdbc mongodb redis

springboot在2.x之后，原来使用的jedis被替换为了lettuce？

jedis：采用的直连，多个线程操作的话，是不安全的如果想要避免不安全的，使用jedis pool连接池！更像BIO模式

lettuce：采用netty，实例可以再多个线程中进行分享，不存在线程不安全的情况！可以减少线程数据了，更像NIO模式

源码分析：

```java
	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")// 我们可以自己定义一个redistemplate来替换这个默认的！
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
    // 默认的redistemplate没有过多地设置，redis对象都是需要序列化的
    // 两个泛型都是object，object的类型，我们使用需要进行强制转换
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean// 由于String是redis最常使用的类型，所以单独提出来一个bean
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
```

测试：

```java
        package com.yuanqing;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisTemplate;

@SpringBootTest
class SbprojectApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {

        // redisTemplate 操作不同的数据类型
        // opsForValue 操作字符串 类似String
        // opsForList 操作List 类似于List
        // opsForSet
        // opsForZSet
        // opsForHash
        // opsForGeo
        // opsForHyperLogLog

        // 除了基本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务和基本的crud
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.flushAll();
//        connection.flushDb();
        redisTemplate.opsForValue().set("园青","太屌了");
        System.out.println(redisTemplate.opsForValue().get("园青"));

    }
}
```

![image-20201015120759544](/typora-user-images/image-20201015120759544.png)

上述红框为序列化配置，默认通过JDK去序列化我们现在要JSON来序列化，那么就需要自定义配置

序列化的方式有很多在企业中，我们所有的pojo 都会序列化！

```java
@Configuration
public class RedisConfig {
    //编写我们自己的redisTemplate
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置具体的序列化方式
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Object> objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper =new ObjectMapper();
        // 第一个参数是序列化的域包括set，get，filed方法，第二个参数是修饰符的范围包括public private
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 第一个参数的意思是不做任何验证允许所有子类型，第二个参数是指定序列化输入类型必须是非final修饰的类型
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL);
        objectJackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // String的序列化
        StringRedisSerializer stringRedisSerializer =new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key
        template.setHashKeySerializer(stringRedisSerializer);
        // value采用Jackson2的序列化方式
        template.setValueSerializer(objectJackson2JsonRedisSerializer);
        // hash的value
        template.setHashKeySerializer(objectJackson2JsonRedisSerializer);
        return template;
    }
}
```

## redis配置文件详解

1.在配置文件中可以看出来对大小写不敏感，以及存储单位

* ```bash
  # 1k => 1000 bytes
  # 1kb => 1024 bytes
  # 1m => 1000000 bytes
  # 1mb => 1024*1024 bytes
  # 1g => 1000000000 bytes
  # 1gb => 1024*1024*1024 bytes
  #
  # units are case insensitive so 1GB 1Gb 1gB are all the same.
  ```

2.可以包含其他配置文件

* ```bash
  # include /path/to/local.conf
  # include /path/to/other.conf
  ```

3.NETWORK

* ```bash
  bind 127.0.0.1 #绑定的IP
  protected-mode yes #保护模式
  port 6379 #端口设置
  ```

4.GENRAL

* ```bash
  daemonize yes #以守护进程的方式开启
  pidfile /var/run/redis_6379.pid #如果以后台的方式允许，我们就需要指定一个pid文件
  # Specify the server verbosity level.
  # This can be one of:
  # debug (a lot of information, useful for development/testing)
  # verbose (many rarely useful info, but not a mess like the debug level)
  # notice (moderately verbose, what you want in production probably)生成环境
  # warning (only very important / critical messages are logged)
  loglevel notice #日志
  logfile "" #日志的文件名
  databases 16 #数据库的数量，默认是16个
  always-show-logo yes #是否总是显示logo
  ```

5.快照

持久化，在规定的时间内，执行了多少次操作，则会持久化到文件.rdb.aof

redis是内存数据库，如果没有持久化那么数据断电即失去

```bash
# 900s内，如果至少有1个key进行了修改，我们进行持久化操作
save 900 1
# 300s内，如果至少有10个key进行了修改，我们进行持久化操作
save 300 10
# 60s内，如果至少有10000个key进行了修改，我们进行持久化操作
save 60 10000

stop-writes-on-bgsave-error yes #持久化出错是否继续工作

rdbcompression yes #是否压缩rdb文件，需要消耗一些CPU资源！

rdbchecksum yes #保存rdb文件的时候，进行错误的检查校验！
```

6.安全SECURITY设置密码

```bash
config get requirepass
config set requirepass "123456"

127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "123456"
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> auth 12345
(error) WRONGPASS invalid username-password pair
127.0.0.1:6379> auth 
(error) ERR wrong number of arguments for 'auth' command
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> 
```

```bash
# maxclients 10000 设置最大客户端数量

# maxmemory <bytes> # redis 配置最大的内存容量

# maxmemory-policy noeviction #内存到达上限后的处理策略，例如移除一些过期的key
1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
2、allkeys-lru ： 删除lru算法的key   
3、volatile-random：随机删除即将过期key   
4、allkeys-random：随机删除   
5、volatile-ttl ： 删除即将过期的   
6、noeviction ： 永不过期，返回错误
```

```bash
APPEND ONLY MODE aof配置

appendonly no #默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下rdb完全够用！

appendfilename "appendonly.aof" #持久化的文件的名字

# appendfsync always # 每次修改都会同步（sync）消耗性能
appendfsync everysec # 没秒执行一次 sync，可能会丢失这1s的数据！
# appendfsync no # 不执行sync，这个时候操作系统自己同步数据，速度最快！
```

## Redis持久化

在指定的时间间隔内将内存中的数据集体写入磁盘，也就是快照(snapshot)，恢复的时候将快照文件直接读到内存里。

单独的创建一个fork子进程来持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感,RDB可能要比AOF更加高效，RDB的缺点是最后一次持久化后的数据可能会丢失。默认就是RDB

保存的文件时dump.rdb

在主从复制中，rdb在从机上备用

在我们配置文件中的快照进行配置

![image-20201027180035396](/typora-user-images/image-20201027180035396.png)

配置文件中

```bash
save 900 1 #900秒有1次修改key操作进行RDB持久化
save 300 10 #以此类推
save 60 10000
```

> 触发机制

1.save的规则满足的情况下，会自动触发rdb规则

2.执行flushall命令，也会触发我们的rdb规则！

3.退出redis，也会产生rdb文件！

备份就自动生成一个dump.rdb文件

> 如何恢复rdb文件！

1.只需要将rdb文件放到我们启动目录就可以了，redis启动的时候就会自动检查rdb文件恢复其中的数据

2.查看需要存放的位置

```bash
1) "dir"

2) "/Users/doinb/usr/local/redis-6.0.7" #如果在这个目录下存在dump.rdb文件，启动就会恢复其中的数据
```

优点：

1.适合大规模的数据恢复！dump.rdb

2.如果你对数据完整性要求不高！

缺点：

1.需要一定的时间间隔！，如果redis意外宕机了，最后一次修改数据就没有了

2.fork进程的时候，会占用一定的内容空间！！

### AOF（Append Only File）

将我们所有的命令都记录下来，history，恢复的时候把这个文件全部执行一遍

![image-20201030175255063](/typora-user-images/image-20201030175255063.png)

以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来（读操作不记录），只需追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话，就会根据日志文件将”写“指令从头到尾执行一遍，以完成数据的恢复



默认是不开启的我们需要手动进行配置:vim redis.conf之后/APPEND，找到appendonly no改为yes

![image-20201118110010531](/typora-user-images/image-20201118110010531.png)

redis-cli

shutdown

redis-server redis.conf

再去输入命令ll查看即可看到appendonly.aof

如果在aof文件中随便写点东西的话或是有错误，redis是无法启动的

我们需要修复这个aof文件，redis给我们提供了一个工具

redis-check-aof --fix appendonly.aof

如果文件正常，重启就可以恢复了

一种是全丢，只丢错误的数据

> ​    如果AOF文件大于64m，太大了！fork一个新的进程来将我们的文件进行重写！
>
> ​	AOF默认就是文件无限追加，文件会越来越大

> ​	优点和缺点

```bash
# appendfsync always #每次都会sync，消耗性能
appendfsync everysec #每秒执行一次sync，可能会丢失这1s的数据
# appendfsync no     #不执行sync，这个时候操作系统自己同步数据，速度最快

# rewirte 重写
```



优点：

每一次修改都同步，文件的完整性会更加好

每秒同步一次，可能会丢失一秒的数据

从不同步，效率是最高的

缺点

相对数据文件来说，aof远远大于rdb，修复的速度也比rdb慢

aof运行效率也要比rdb慢，所以redis默认的持久化是aof，不是rdb



总结：

![image-20201208111542828](/typora-user-images/image-20201208111542828.png)

性能建议：

![image-20201208111615697](/typora-user-images/image-20201208111615697.png)

## redis 发布订阅

### 发布订阅模式中的角色

- 发布者（publisher）
- 订阅者（subscriber）
- 频道（channel）

如图所示：

![img](/typora-user-images/image-20250109220645561.png)

发布者发布消息到频道，订阅了频道的订阅者可以收到消息，订阅者可以订阅不同的频道。

### 通信模型

- RedisServer中可以创建若干channel
- 一个订阅者可以订阅多个channel
- 当发布者向一个频道中发布一条消息时，所有的订阅者都将会收到消息
- Redis的发布订阅模型没有消息积压功能，即新加入的订阅者收不到发布者之前发布的消息
- 当订阅者收到消息时，消息内容如下
  - 第一行：固定内容message
  - 第二行：channel的名称
  - 第三行：收到的新消息



订阅者代码：

```bash
127.0.0.1:6379> subscribe yuanqing
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "yuanqing"
3) (integer) 1
#等待推送信息
1) "message" #消息
2) "yuanqing" #那个频道的消息
3) "jingrishaonianliubowen" #具体内容
```

发布者代码

```bash
127.0.0.1:6379> publish yuanqing jingrishaonianliubowen
(integer) 1
```

通过SUBSCRIBE命令订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个频道！而字典的值则是一个链表，链表中保存了所有订阅这个channel的客户端。SUBSCRIBE命令的关键，就是将客户端添加到给定的channel的订阅链表中。

![image-20201209123846582](/typora-user-images/image-20201209123846582.png)

通过PUBLISH命令向订阅者发送消息，redis-server会使用给定的频道作为键，在它维护的channel字典中查找记录了订阅这个频道的素有客户端的链表，遍历这个链表，将消息发布给所有订阅者

PUB/SUB从字面上理解就是发布与订阅，在redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值进行了消息发布后，所有订阅它的客户端都会收到相应的消息。

实时消息，聊天

### 发布订阅的 API

|                 命令                  |                含义                |
| :-----------------------------------: | :--------------------------------: |
|        publish channel message        |     向指定的channel中发布消息      |
|   subscribe channel1 [channel2...]    |   订阅给定的一个或多个渠道的消息   |
|  unsubcribe [channel1 [channel2...]]  | 取消订阅给定的一个或多个渠道的消息 |
|   psubscribe pattern1 [pattern2...]   |  订阅一个或多个符合给定模式的频道  |
| punsubscribe [pattern1 [pattern2...]] |       退订所有给定模式的频道       |
|            pubsub channel             |     列出至少有一个订阅者的频道     |
|      pubsub numsub [channel...]       |      列出给定频道的订阅者数量      |

## 主从复制

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器，前者称为主节点，后者称为从节点，数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave以读为主，一个主节点可以有多个从节点，一个从节点只能有一个主节点，其实就是读写分离，减缓服务器的压力！架构总经常使用，一主二从的模式

作用如下：

![image-20201222165143823](/typora-user-images/image-20201222165143823.png)

一般来说要将Redis运用于工程项目中，只使用一台redis是万万不能的（宕机），原因如下：

从结构上，单个redis服务器会发生单点故障，并且一台服务器所需要的处理所有的请求负载，压力较大

从容量上，单个redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作redis存储内存，一般来说，单台redis最大使用内存不应该超过20G。

电商网站上的商品，一般都是一次上传，无数次浏览的，多读少写

![image-20201224102610344](/typora-user-images/image-20201224102610344.png)

环境配置只配置从库不用配置主库

主从配置：永久配置从配置文件中进行配置

### 安装

```shell
【linux12(10.0.0.12)】:
yum install epel-release
yum install redis


【linux14(10.0.0.14)】:
yum install epel-release
yum install redis
Copy
```

### 修改配置文件

```shell
【linux12(10.0.0.12)】:
vi /etc/redis.conf
修改daemonize no为daemonize yes，快速找到daemonize no的命令为【/daemonize no】
修改bind 127.0.0.1为bind 10.0.0.12，快速找到bind 127.0.0.1的命令为【/bind 127.0.0.1】


【linux14(10.0.0.14)】:
vi /etc/redis.conf
修改daemonize no为daemonize yes，快速找到daemonize no的命令为【/daemonize no】
在# slaveof <masterip> <masterport>下面添加slaveof 10.0.0.12 6379，快速找到# slaveof <masterip> <masterport>的命令为【/# slaveof <masterip> <masterport>】
修改# repl-ping-slave-period 10，取消注释，修改为repl-ping-slave-period 10
Copy
```

### 启动redis服务

```shell
【linux12(10.0.0.12)】:
redis-server /etc/redis.conf

【linux12(10.0.0.14)】:
redis-server /etc/redis.conf
Copy
```

### 启动redis客户端

```shell
【linux12(10.0.0.12)】:（因前面绑定了ip，所以要指定-h 10.0.0.12）
redis-cli -h 10.0.0.12

【linux12(10.0.0.14)】:
redis-cli
Copy
```

### 同期情况确认

```shell
【linux12(10.0.0.12)】:
INFO replication

【linux12(10.0.0.14)】:
INFO replication
Copy
```

### 主从同步测试

```shell
【linux12(10.0.0.12)】:
10.0.0.12:6379> set test 1

【linux12(10.0.0.14)】:
127.0.0.1:6379> get test
```

或是单机多集群：

复制3个配置文件，修改对应的信息，

1，端口

2，pid 名字

```bash
pidfile /var/run/redis_6379.pid
```

3，log文件名字

```bash
logfile "6479.log"
```

4，dump.rdb名字

```bash
dbfilename dump.rdb
```

如果要配置主从，只要在从机上认老大即可

SLAVEOF 127.0.0.1 6379 # 认端口号为6379的主机当老大

info replication # 查看信息

**主机可以写，从机不能写只能读!**

主机断开连接，从机依旧连接到主机的，但是没有写操作，这个时候，主机如果回来了，从机依旧可以直接获取到主机写的信息

如果是使用命令行，来配置的主从，这个时候如果重启了，就会变回主机！只要变为从机，立马就能从主机中获取值！

> 复制原理

Slave启动成功连接到master后会发送一个sync同步命令

master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集的命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

增量复制：master继续将新的所有收集到的修改命令一次传给slave，完成同步

但是只有重新连接master，一次完全同步（全量复制）将被自动执行！我们数据一定可以在从机中看到！

> 层层链路

上一个M连接下一个S

![image-20201231123218259](/typora-user-images/image-20201231123218259.png)

这时候也可以完成我们的主从复制

> 如果没有老大了，这个时候能不能选择一个老大出来呢？手动！

谋朝篡位

如果主机断开了连接，我们可以使用 SLAVEOF no one 让自己变成主机！其他的节点就可以手动连接到最新的这个主节点（手动）如果这个时候老大修复了，那就只得重新连接

## 哨兵模式

主从切换技术的方法是：当服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费时费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式

谋朝篡位的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，他会独立运行，其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控多个Redis实例。**

![image-20201231155313634](/typora-user-images/image-20201231155313634.png)

![image-20201231155526609](/typora-user-images/image-20201231155526609.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover(故障转移)过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象称为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover（故障转移）。切换成功后通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

1，配置哨兵

vim sentinel.conf

```bash
#sentinel monitor 被监控名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面的数字1代表主机挂了，slave投票看让谁接替成为主机，票数最多的，就会成为主机！

2，启动哨兵

redis-sentinel sentinel.conf（哨兵配置文件）

如果Master节点断开了，这个时候就会从从机中随机选择一个服务器！（这里面有一个投票算法）

哨兵日志

![image-20201231185027821](/typora-user-images/image-20201231185027821.png)

由此得出最先的主机是6381

**如果主机此时回来了，只能归并到新的主机下，当做从机，这就是哨兵模式的规则！**

> 哨兵模式

优点：

1，哨兵集群，基于主从复制模式，所有的主从配置优点，他全有

2，主从可以切换，故障可以转移，系统的可用性就会更好

3，哨兵模式就是主从模式的升级，手动到自动，更加健壮！

缺点：

1，Redis不好在线扩容，集群容量一旦到达上限，在线扩容就十分麻烦!

2，实现哨兵模式的配置其实是很麻烦的，里面有很多选择

 其他配置文件详解

```bash
# 配置端口,默认26379
port 26379

# 以守护进程模式启动
daemonize yes

# 日志文件名
logfile "sentinel_26379.log"

# 存放备份文件以及日志等文件的目录
dir "/opt/redis/data"

# sentinel monitor <master-name> <master-ip> <master-port> <quorum>
# master-name 监控主节点的名称，只能由字母A-z、0-9、.、-、_ 这些符号组成
# quorum 配置多少个sentinel哨兵同意认为Master主节点失联，那么这时客观上认为主节点失联了
sentinel monitor mymaster 192.168.14.101 6379 2

# sentinel down-after-milliseconds <master-name> <milliseconds>
# 30秒ping不通主节点的信息，主观认为master宕机
sentinel down-after-milliseconds mymaster 30000

# sentinel auth-pass <master-name> <password> 
# 当在redis实例开启了requirepass foobared 授权密码，这样所有连接redis的客户端都要提供密码，
# 设置哨兵连接主从的密码，注意： 必须为主从设置一样的密码
sentinel auth-pass mymaster 123456

# sentinel parallel-syncs <master-name> <numslaves>
# 指定了在发生failover主备切换是最多可以有多少个slave同时对新的master进行同步，
# 这个数字越小，完成failover的时间越长，但是数字越大，意味着越多的slave因为replication不可用，可以通过将这个值设为1，来保证# # 每次只有一个slave处于不能处理命令请求的状态
sentinel parallel-syncs mymaster 1

# 故障转移开始，三分钟内没有完成，则认为转移失败，默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
# 故障转移的超时时间，可以用在以下方面
# 1、同一个sentinel对同一个master两次failover之间的时间间隔；
# 2、当一个slave从一个错误的master那里同步数据开始计算时间，直到slave被纠正为向正确的master那里同步数据时间；
# 3、当想要取消一个正在进行的failover所需要的时间；
# 4、当failover时，配置所有slaves指向新的master所需要的最大时间，不过即使过了这个超时，slaves依然会被正确配置为指向master,但是就不按parallel-syncs所配置的规则来了。
sentinel failover-timeout mymaster 180000

# sentinel notification-script <master-name> <script-path>
# 配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时，发送邮件通知相关人员
# 对于脚本的运行结果有以下规则：
# 1、若脚本返回1，该脚本稍后将会被再次执行，重复次数默认为10；
# 2、若脚本返回≥2，脚本将不会重复执行；
# 3、如果脚本在执行过程中收到系统终端信号被终止了，则与返回值为1的行为相同；
# 4、一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个叫SIGKILL信号终止，之后重新执行。

# 如通知型脚本：当sentinel有任何告警几倍的时间发生时（比如redis实例的主、客观失效等等），将会去调用这个脚本，此时脚本将通过邮件等方式通知系统管理员，调用该脚本时，将传给脚本两个参数：一是时间类型，二是事件的描述。如果sentinel.conf配置文件配置了这个脚本路径，那么必须保证这个脚本存在这个路径，并且是可执行的，否则sentinel无法正常启动成功。
sentinel notification-script mymaster /var/redis/notify.sh

# sentinel client-reconfig-script <master-name> <script-path>
# 客户端重新配置主节点参数脚本，当一个master由于failover发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息
# 以下参数将会在调用脚本时传给脚本：
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前state总是 failover
# role 是 leader 或者 observer 中的一个
# <from-ip> <from-port> <to-ip> <to-port> 是用来和旧的master 和新的 master(即旧的slave)通信的
# 具备通用信，多次调用
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```

## 缓存穿透和雪崩

在这里不会详细分析解决方案的底层

Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中最要害的问题，就是**缓存穿透，缓存雪崩，和缓存击穿**。

### 缓存穿透(查不到)

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败，当用户很多的时候，缓存都没有命中（秒杀），于是都去请求了持久层数据库，这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

![image-20210107190957720](/typora-user-images/image-20210107190957720.png)

如何解决？

1.在请求的时候加过滤器

**布隆过滤器**

多所有可能的查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力；

2.假设缓存没有命中那么就给赋一个空值，用户请求出来就是空的

![image-20210108174221043](/typora-user-images/image-20210108174221043.png)

**缓存空对象**

会存在两个问题：

1，如果空值能够被缓存起来，这就意味着缓存需要更多地空间存储更多地键，因为这当中可能会有很多的空值的键；

2，即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对需要保持一致性的业务会有影响。例如存储层已经有了但缓存没有

![image-20210108174048056](/typora-user-images/image-20210108174048056.png)

### 缓存击穿(量太大)

例如微博有一个热搜所有人都去访问这个时候并发量是巨大的，假如这个key设置了60秒过期，60.1秒后请求数据库在设到redis缓存中，在这0.1秒的瞬间可能会有大量的请求直接请求数据库，被称为缓存击穿，这可能会导致微博宕机。

也就是当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般都是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且写回缓存，会导致使数据库压力瞬间过大

> 解决方案

**设置热点数据不过期**

**加互斥锁**

分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需哟等待即可

![image-20210108182849169](/typora-user-images/image-20210108182849169.png)

### 缓存雪崩

> 概念

缓存雪崩，是指在某一时间段，缓存集中过期失效。或者Redis宕机！

产生雪崩的原因之一，比如在写文本的时候，马上就要到双12零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时，那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性压力波峰。于是所有请求都会到达存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。

![image-20210111103514953](/typora-user-images/image-20210111103514953.png)

其实集中过期，倒不是非常致命的，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网，因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的，无非就是对数据库产生周期性的压力而已，而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

双11：停掉一些服务，（保证主要的服务可用），比如在双11的时候退款，直接告诉你，当天业务繁忙不能退款

> 解决方案

**redis高可用**

这个思想的含义是,既然redis有可能挂掉,那我多增设几台redis ,这样一台挂掉之后其他的还可以继续工作,其实就是搭建的集群。(异地多活! )

**限流降级**

这个解决方案的思想是, 在缓存失效后,通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存,其他线程等待。

**数据预热**

数据加热的含义就是在正式部署之前,我先把可能的数据先预先访问一遍,这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key ,设置不同的过期时间,让缓存失效的时间点尽量均匀。

