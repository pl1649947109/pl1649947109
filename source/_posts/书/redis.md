---
title: redis蹒跚学步
id: 1
date: 2019-10-31 20:00:00
tags: redis
comment: true
---

该篇内容来源：https://www.cnblogs.com/pyyu/p/9843950.html

## 菜单redis基础

### Nosql

```
学名（not only sql）

特点：
存储结构与mysql这一种关系型数据库完全不同，nosql存储的是KV形式
nosql有很多产品，都有自己的api和语法，以及业务场景

产品种类：
Mongodb
redis
Hbase hadoop
```

### Nosql和sql的区别

```
应用场景不同，sql支持关系复杂的数据查询，nosql反之
sql支持事务性，nosql不支持
```

<!----more---->

### Redis特性

```
Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件redis是c语言编写的，支持数据持久化，是key-value类型数据库。应用在缓存，队列系统中redis支持数据备份，也就是master-slave模式
```

### Redis优势

```
性能高，读取速度10万次/秒
写入速度8万次每秒
所有操作支持原子性

用作缓存数据库，数据放在内存中
替代某些场景下的mysql，如社交类app
大型系统中，可以存储session信息，购物车订单
```

### yum安装redis

1.yum安装

```
#前提得配置好阿里云yum源，epel源
#查看是否有redis包
yum list redis
#安装redis
yum install redis -y
#安装好，启动redis
systemctl start redis
```

2.检测redis是否工作

```
redis-cli    #redis 客户端工具
#进入交互式环境后，执行ping，返回pong表示安装成功
127.0.0.1:6379> ping
PONG（表示通了）
```

### redis可执行文件

```
./redis-benchmark //用于进行redis性能测试的工具
./redis-check-dump //用于修复出问题的dump.rdb文件
./redis-cli //redis的客户端
./redis-server //redis的服务端
./redis-check-aof //用于修复出问题的AOF文件
./redis-sentinel //用于集群管理
```

### redis配置文件

```
redis.conf 
```

### redis.conf 的核心配置

```
绑定ip，如需要远程访问，需要填写服务器ip
bind 127.0.0.1  

端口，redis启动端口
port 

守护进程方式运行
daemonize yes

rdb数据文件
dbfilename dump.rdb

数据文件存放路径
dir /var/lib/redis/

日志文件
logfile /var/log/redis/redis-server.log

主从复制
slaveof
```

### 使用redis客户端

```
#执行客户端命令即可进入
./redis-cli  
#测试是否连接上redis
127.0.0.1:6379 > ping
返回pong代表连接上了

//用set来设置key、value
127.0.0.1:6379 > set name "chaoge"
OK
//get获取name的值
127.0.0.1:6379 > get name
"chaoge"
```

### redis数据类型

```
redis是一种高级的key：value存储系统，其中value支持五种数据类型
字符串（strings）
散列（hashes）
列表（lists）
集合（sets）
有序集合（sorted sets）
```

### 基本命令

```
keys *         查看所有key
type key      查看key类型
expire key seconds    过期时间
ttl key     查看key过期剩余时间        -2表示key已经不存在了
persist     取消key的过期时间   -1表示key存在，没有过期时间

exists key     判断key存在    存在返回1    否则0
del keys     删除key    可以删除多个
dbsize         计算key的数量
```

### strings数据类型

- set  设置key
- get   获取key
- append  追加string
- mset   设置多个键值对
- mget   获取多个键值对
- del  删除key
- incr  递增+1
- decr  递减-1

```shell
127.0.0.1:6379> set name 'yu'   #设置key
OK
127.0.0.1:6379> get name　　　　#获取value
"yu"
127.0.0.1:6379> set name 'yuchao'　　#覆盖key
OK
127.0.0.1:6379> get name　　　　#获取value
"yuchao"
127.0.0.1:6379> append name ' dsb' 　　#追加key的string
(integer) 10
127.0.0.1:6379> get name　　#获取value
"yuchao dsb"


127.0.0.1:6379> mset user1 'alex' user2 'xiaopeiqi'　　　　#设置多个键值对
OK
127.0.0.1:6379> get user1　　　　#获取value
"alex"
127.0.0.1:6379> get user2　　　　#获取value
"xiaopeiqi"
127.0.0.1:6379> keys *　　　　　　#找到所有key
1) "user2"
2) "name"
3) "user1"

127.0.0.1:6379> mget user1 user2 name   #获取多个value
1) "alex"
2) "xiaopeiqi"
3) "yuchao dsb"
127.0.0.1:6379> del name　　　　　　　　#删除key
(integer) 1
127.0.0.1:6379> get name　　　　　　　　#获取不存在的value，为nil
(nil)
127.0.0.1:6379> set num 10　　　　#string类型实际上不仅仅包括字符串类型，还包括整型，浮点型。redis可对整个字符串或字符串一部分进行操作，而对于整型/浮点型可进行自增、自减操作。
OK　　　　
127.0.0.1:6379> get num
"10"
127.0.0.1:6379> incr num　　　　#给num string 加一 INCR 命令将字符串值解析成整型，将其加一，最后将结果保存为新的字符串值，可以用作计数器
(integer) 11
127.0.0.1:6379> get num　　
"11"

127.0.0.1:6379> decr num　　　　　　#递减1  
(integer) 10
127.0.0.1:6379> decr num　　　　#递减1
(integer) 9
127.0.0.1:6379> get num
"9"
```

### list数据类型

- lpush         从列表左边插
- rpush         从列表右边插
- lrange          获取一定长度的元素  lrange key  start stop
- ltrim               截取一定长度列表
- lpop                 删除最左边一个元素
- rpop                     删除最右边一个元素
- lpushx/rpushx                key存在则添加值，不存在不处理

```shell
lpush duilie 'alex' 'peiqi' 'ritian'  #新建一个duilie，从左边放入三个元素

llen duilie  #查看duilie长度

lrange duilie 0 -1  #查看duilie所有元素

rpush duilie 'chaoge'   #从右边插入chaoge

lpushx duilie2  'dsb'  #key存在则添加 dsb元素，key不存在则不作处理

ltrim duilie 0 2  #截取队列的值，从索引0取到2，删除其余的元素

lpop #删除左边的第一个
rpop #删除右边的第一个
```

### sets集合类型

- sadd/srem   添加/删除 元素
- sismember   判断是否为set的一个元素
- smembers    返回集合所有的成员
- sdiff             返回一个集合和其他集合的差异
- sinter           返回几个集合的交集
- sunion          返回几个集合的并集

```shell
sadd zoo  wupeiqi yuanhao  #添加集合，有三个元素，不加引号就当做字符串处理

smembers zoo  #查看集合zoo成员

srem zoo  wupeiqi #删除zoo里面的alex

sismember zoo wupeiqi  #返回改是否是zoo的成员信息，不存在返回0，存在返回1

sadd zoo wupeiqi   #再把wupeiqi加入zoo

smembers zoo  #查看zoo成员

sadd zoo2 wupeiqi mjj #添加新集合zoo2

sdiff zoo zoo2 #找出集合zoo中有的，而zoo2中没有的元素

sdiff zoo2  zoo  #找出zoo2中有，而zoo没有的元素

sinter zoo zoo1   #找出zoo和zoo1的交集，都有的元素

sunion  zoo zoo1  #找出zoo和zoo1的并集，所有的不重复的元素
```

### 有序集合

**都是以z开头的命令**：用来保存需要排序的数据，例如排行榜，成绩，工资等。

- 利用有序集合的排序，排序学生的成绩

```shell
127.0.0.1:6379> ZADD mid_test 70 "alex"
(integer) 1
127.0.0.1:6379> ZADD mid_test 80 "wusir"
(integer) 1
127.0.0.1:6379> ZADD mid_test 99 "yuyu"
```

- 排行榜，zreverange 倒叙   zrange正序

```shell
127.0.0.1:6379> ZREVRANGE mid_test 0 -1 withscores
1) "yuyu"
2) "99"
3) "wusir"
4) "80"
5) "xiaofneg"
6) "75"
7) "alex"
8) "70"
127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
1) "alex"
2) "70"
3) "xiaofneg"
4) "75"
5) "wusir"
6) "80"
7) "yuyu"
8) "99"
```

- 移除有序集合mid_test中的成员，xiaofeng给移除掉

```shell
127.0.0.1:6379> ZREM mid_test xiaofneg
(integer) 1
127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
1) "alex"
2) "70"
3) "wusir"
4) "80"
5) "yuyu"
6) "99"
```

- 返回有序集合mid_test的基数

```shell
127.0.0.1:6379> ZCARD mid_test
(integer) 3
```

- 返回成员的score值

```shell
127.0.0.1:6379> ZSCORE mid_test alex
"70"
```

- zrank返回有序集合中，成员的排名。默认按score，从小到大排序。

```shell
127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
1) "alex"
2) "70"
3) "wusir"
4) "80"
5) "yuyu"
6) "99"
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> ZRANK mid_test wusir
(integer) 1
127.0.0.1:6379> ZRANK mid_test yuyu
(integer) 2
```

### 哈希数据结构

- hset 设置散列值
- hget  获取散列值
- hmset  设置多对散列值
- hmget  获取多对散列值
- hsetnx   如果散列已经存在，则不设置（防止覆盖key）
- hkeys     返回所有keys
- hvals     返回所有values
- hlen      返回散列包含域（field）的数量
- hdel     删除散列指定的域（field）
- hexists    判断是否存在

```shell
redis hash是一个string类型的field和value的映射表

语法  hset key field value  

hset news1   title "first news title" #设置第一条新闻 news的id为1，添加数据title的值是"first news title"

hset news1 content "news content"    #添加一个conntent内容

hget news1 title   #获取news:1的标题

hget news1  content  #获取news的内容

hmget news1  title content   #获取多对news:1的 值

hmset news2 title "second news title" content "second Contents2"   #设置第二条新闻news:2 多个field

hmget news2 title  content #获取news:2的多个值

hkeys news1   #获取新闻news:1的所有key

hvals news1   #获取新闻news:1的所有值

hlen news1    #获取新闻news:1的长度

hdel news1 title   #删除新闻news:1的title

hlen news1     #看下新闻news:1的长度

hexists news1 title    #判断新闻1中是否有title，不存在返回0，存在返回1
```

## redis发布订阅（异步队列）

引子：

**使用 Redis 中的 List 作为队列**

**Redis 通过 PUBLISH 、 SUBSCRIBE 等命令实现了订阅与发布模式。**

使用上文所说的 Redis 的数据结构中的 List 作为队列 Rpush 生产消息，LPOP 消费消息。

**缺点：**LPOP 不会等待队列中有值之后再消费，而是直接进行消费。

**弥补：**可以通过在应用层引入 Sleep 机制去调用 LPOP 重试。



**使用 BLPOP key [key…] timeout**

BLPOP key [key …] timeout：阻塞直到队列有消息或者超时。

**缺点：**按照此种方法，我们生产后的数据只能提供给各个单一消费者消费。能否实现生产一次就能让多个消费者消费呢？

整体：

**Pub/Sub：主题订阅者模式**

![](https://img2018.cnblogs.com/blog/1132884/201811/1132884-20181124213748132-1685790439.png)

![](https://img2018.cnblogs.com/blog/1132884/201811/1132884-20181124215053216-993504589.png)

Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。

通过SUBSCRIBE命名订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个channel，而字典的值则是一个链表，链表中保存了多有订阅这个channel的客户端。SUBSCRIBE命令的关键，就是将客户端添加到给定channel的订阅链表中。

通过 PUBLISH 命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。

### 订阅一个频道

启动两个窗口（用于订阅）

```shell
#执行命令
SUBSCRIBE pl(这个是订阅频道)
```

启动一个窗口（用于发布消息）

```shell
#执行消息发布
PUBLISH pl meg(发布消息的内容)
```

上面的就是一个简单的发布订阅例子的实现。

### 订阅多个频道

启动两个窗口（用于订阅）

```shell
#执行命令
PUSBSCRIBE p*(订阅以p开头的频道)
```

启动一个窗口（用于发布消息）

```shell
#发布两个频道（都可以将消息发送到上面订阅的客户端）
PUBLISH pl "asasa"
PUBLISH pl23 "plps"
```

### 命令总结

```shell
#将消息发送到指定的频道channel
PUBLISH channel msg  
#订阅频道，可以同时订阅多个频道
SUBSCRIBE channel [channel...]
#订阅一个或多个符合给定模式的频道，每个模式以*作为匹配符
PSUBSCRIBE pattern [pattern...]
#取消订阅指定的频道，如果不指定，则会取消订阅所有频道
UNSUBSCRIBE [channel...]
#退订指定的规则, 如果没有参数则会退订所有规则
PUNSUBSCRIBE [pattern [pattern ...]]
#查看订阅与发布系统状态
PUBSUB subcommand [argument [argument ...]]

#注意：使用发布订阅模式实现的消息队列，当有客户端订阅channel后只能收到后续发布到该频道的消息，之前发送的不会缓存，必须Provider和Consumer同时在线
```

**Pub/Sub模式的缺点：**消息的发布是无状态的，无法保证可达。对于发布者来说，消息是“即发即失”的。

此时如果某个消费者在生产者发布消息时下线，重新上线之后，是无法接收该消息的，要解决该问题需要使用专业的消息队列，如 Kafka…

## kafka

### 为什么使用消息队列

```
比如在一个企业里，技术老大接到boss的任务，技术老大把这个任务拆分成多个小任务，完成所有的小任务就算搞定整个任务了。
那么在执行这些小任务的时候，可能有一个环节很费时间，并且优先级很低，推迟完成也不影响整个任务运转，那么技术老大就会将这个很费时间，且不重要的任务，丢给他的小弟去解决，自己继续完成其他任务。

那个技术老大就是一个 程序系统，那个小弟就是消息队列。
当程序系统发现某些任务耗费时间且优先级较低，迟点完成也不影响整个任务，就把这个任务丢给消息队列。
```

### 应用场景

```
在程序系统中，例如外卖系统，订单系统，库存系统，优先级较高
发红包，发邮件，发短信，app消息推送等任务优先级很低，很适合交给消息队列去处理，以便于程序系统更快的处理其他请求。
```

### 消息队列工作流程

```
消息队列一般有三个角色：
队列服务端
队列生产者
队列消费者

消息队列工作流程就如同一个流水线，有产品加工，一个输送带，一个打包产品
输送带就是 不停运转的消息队列服务端
加工产品的就是 队列生产者
在传输带结尾打包产品的 就是队列消费者
```

### 队列产品

```
RabbitMQ：Erlang编写的消息队列产品，企业级消息队列软件，支持消息负载均衡，数据持久化等。
ZeroMQ ：saltstack软件使用此消息，速度最快。
Redis：key-value的系统，也支持队列数据结构，轻量级消息队列
Kafka：由Scala编写，目标是为处理实时数据提供一个统一、高通量、低等待的平台
```

### 一个app系统消息队列工作流程

```
消费者，一个后台进程，不断的去检测消息队列中是否有消息，有消息就取走，开启新线程去处理业务，如果没有一会再来
```

### 消息队列的作用

```
1）程序解耦

允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2）冗余：

消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。

许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

3）峰值处理能力：

(大白话，就是本来公司业务只需要5台机器，但是临时的秒杀活动，5台机器肯定受不了这个压力，我们又不可能将整体服务器架构提升到10台，那在秒杀活动后，机器不就浪费了吗？因此引入消息队列)

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。

如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。

使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

4）可恢复性：

系统的一部分组件失效时，不会影响到整个系统。

消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

5）顺序保证：

在大多使用场景下，数据处理的顺序都很重要。

大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka保证一个Partition内的消息的有序性）

6）缓冲：

有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

7）异步通信：

很多时候，用户不想也不需要立即处理消息。比如发红包，发短信等流程。

消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。
```

### kafka架构

```
1）Producer ：消息生产者，就是向kafka broker发消息的客户端。

2）Consumer ：消息消费者，向kafka broker取消息的客户端

3）Topic ：主题，可以理解为一个队列。

4） Consumer Group （CG）：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制-给consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。

5）Broker ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。

6）Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。

7）Offset：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka
```



## redis持久化RDB与AOF

### redis的持久化

Redis是一种内存型数据库，一但服务器进程退出，数据库的数据就会丢失，为了解决这个问题，Redis提供了两种持久化的方案，将内存中的数据保存到磁盘中，避免数据的丢失。

### RDB持久化

**手动**：`redis`提供了`RDB持久化`的功能，这个功能可以将`redis`在内存中的的状态保存到硬盘中，它可以**手动执行。**

**自动**：也可以再`redis.conf`中配置，**定期执行**。

**RDB持久化产生的RDB文件是一个经过压缩的二进制文件，这个文件被保存在硬盘中，redis可以通过这个文件还原数据库当时的状态。**

**实战**







### AOF持久化

记录服务器执行的多有变更操作（例如set del），并在服务器启动时，通过重新执行这些命令来还原数据集。

AOF 文件中的命令全部以redis协议的格式保存，新命令追加到文件末尾。

优点：最大程度保证数据不丢

缺点：日志记录非常大





### 总结

- 持久化方式有哪些？有什么区别？

  rdb：基于快照的持久化，速度更快，主从复制也是依赖于rdb持久化功能。

  aof：以追加的方式记录redis操作日志的文件，可以最大程度的保证redis数据安全。

- **RDB 优点：**全量数据快照，文件小，恢复快。

- **RDB 缺点：**无法保存最近一次快照之后的数据。

- **AOF 优点：**可读性高，适合保存增量数据，数据不易丢失。

- **AOF 缺点：**文件体积大，恢复时间长。

## redis主从同步

主从复制的作用：数据备份、读写分离、分布式集群、实现高可用、宕机容错机制

- redis复制功能一类是主数据库，一类是从数据库，主数据库可以进行读写操作；当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的。在redis里面只有一个主。
- 通过redis的复制功能可以很好的实现数据库的读写分离，提高数据库的负载能力。主数据库主要进行写操作，而从数据库负责读操作。

### 原理

- Slave 发送 Sync 命令到 Master。
- Master 启动一个后台进程，将 Redis 中的数据快照保存到文件中。
- Master 将保存数据快照期间接收到的写命令缓存起来。
- Master 完成写文件操作后，将该文件发送给 Slave。
- 使用新的 AOF 文件替换掉旧的 AOF 文件。
- Master 将这期间收集的增量写命令发送给 Slave 端。

### 实战

**准备环境**

6380.conf

```shell
1、环境：
准备两个或两个以上redis实例

mkdir /data/638{0..2}  #创建6380 6381 6382文件夹

配置文件示例：
vim   /data/6380/redis.conf
port 6380
daemonize yes
pidfile /data/6380/redis.pid
loglevel notice
logfile "/data/6380/redis.log"
dbfilename dump.rdb
dir /data/6380
protected-mode no
```

6381.conf

```shell
port 6382
daemonize yes
pidfile /data/6382/redis.pid
loglevel notice
logfile "/data/6382/redis.log"
dbfilename dump.rdb
dir /data/6382
protected-mode no
```

6382.conf

```shell
port 6382
daemonize yes
pidfile /data/6382/redis.pid
loglevel notice
logfile "/data/6382/redis.log"
dbfilename dump.rdb
dir /data/6382
protected-mode no
```

启动三个redis实例

```shell
redis-server /data/6380/redis.conf
redis-server /data/6381/redis.conf
redis-server /data/6382/redis.conf
```

主从规划

```
主节点：6380（master）
从节点：6381、6382 （slave）
```

**配置主从同步**

配置从

```shell
redis-cli -p 6381   
SLAVEOF 127.0.0.1 6380  #指明主的地址

redis-cli -p 6382
SLAVEOF 127.0.0.1 6380  #指明主的地址
```

检查从库状态：

```shell
127.0.0.1:6382> info replication
127.0.0.1:6381> info replication

#或者直接使用info命令也是可以的
```

检查主库状态：

```shell
127.0.0.1:6380> info replication
```

**测试**

```shell
#主
127.0.0.1:6380> set name pl
OK

#从
127.0.0.1:6381> get name
"pl"
```

### 手动进行主从切换

**关闭主库6380**

```shell
#关闭主库6380
redis-cli -p 6380
shutdown
```

**检查主从信息**

```
info replication

# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:dowm   （这条消息就说明主挂掉了）
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:14
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ba54af40bb025b0850643417991864b301a9a8bf
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
```

**选新的主**

**关闭6381的从库身份**

```shell
slaveof no one
```

**将6382设为6381的从库**

```
6382连接到6381：
[root@db03 ~]# redis-cli -p 6382
127.0.0.1:6382> SLAVEOF no one
127.0.0.1:6382> SLAVEOF 127.0.0.1 6381
```

完成。

## redis哨兵机制

Redis-Sentinel是redis官方推荐的高可用性解决方案。（哨兵机制需要主从复制的支持）

```
当用redis作master-slave的高可用时，如果master本身宕机，redis本身或者客户端都没有实现主从切换的功能。
而redis-sentinel就是一个独立运行的进程，用于监控多个master-slave集群，
自动发现master宕机，进行自动切换slave > master。
```

**sentinel（哨兵）主要功能**：

- **监控**：redis是否良好运行，如果结点不可达就会对结点继续下线标识
- **提醒**：当被监控的某个redis出现问题时，哨兵可以通过API向管理员或者其他程序发送通知
- **自动故障转移**：如果被标识的是主结点sentinel就会和其他的sentinel结点“协商”，如果其他结点也认为主结点不可达，就会选举出一个sentinel结点来完成**自动故障转移**
- 在master-slave进行切换后，master_redis.conf、slave_redis.conf、和sentinel.conf的内容都会发生变化，即即master_redis.conf中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换

**Sentinel的工作原理**

```
每个Sentinel以每秒钟一次的频率向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令
 

如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel 标记为主观下线。

如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。

当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线

在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令

当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次

若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除。

若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

主观下线和客观下线

主观下线：Subjectively Down，简称 SDOWN，指的是当前 Sentinel 实例对某个redis服务器做出的下线判断。
客观下线：Objectively Down， 简称 ODOWN，指的是多个 Sentinel 实例在对Master Server做出 SDOWN 判断，并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后，得出的Master Server下线判断，然后开启failover.

SDOWN适合于Master和Slave，只要一个 Sentinel 发现Master进入了ODOWN， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对下线的主服务器执行自动故障迁移操作。

ODOWN只适用于Master，对于Slave的 Redis 实例，Sentinel 在将它们判断为下线前不需要进行协商， 所以Slave的 Sentinel 永远不会达到ODOWN。
```

### 主从复制的背景

Redis主从复制可将主结点数据同步给从结点，从结点此时有两个作用：

- 一旦主结点宕机，从结点作为主结点的备份可以随时顶上来
- 扩展从结点的读功能，分担主结点读压力
- 一旦主结点宕机，从结点上位，那么就需要人为修改所有一个应用方主结点地址（改为新的master地址），还需要命令所有从结点复制新的主结点（这个问题就是Sentinel可以解决的）

### 主从复制架构

![](https://img2018.cnblogs.com/blog/1132884/201809/1132884-20180928114913282-867816204.png)

![](https://img2018.cnblogs.com/blog/1132884/201809/1132884-20180928115021716-2071889796.png)

### Redis Sentinel架构

![](https://img2018.cnblogs.com/blog/1132884/201811/1132884-20181124181822218-1125753774.png)

![](https://img2018.cnblogs.com/blog/1132884/201809/1132884-20180928115711245-1503702880.png)

![](https://img2018.cnblogs.com/blog/1132884/201809/1132884-20180928145626051-1360651849.png)

![](https://img2018.cnblogs.com/blog/1132884/201809/1132884-20180928145734973-1288883859.png)

### 命令总结

```shell
官网地址：http://redisdoc.com/

redis-cli info #查看redis数据库信息

redis-cli info replication #查看redis的复制授权信息

redis-cli info sentinel   #查看redis的哨兵信息
```

### 实战

**配置3台服务器，一主二从**

6379使用本机的redis.conf做主

```
不改
```

从/etc/redis-6380.conf

```shell
port 6380
daemonize yes   #这个必须设置
loglevel notice
dbfilename dump.rdb
protected-mode no
slaveof 127.0.0.1 6379  #从连接主
```

从/etc/redis-6381.conf

```shell
port 6381
daemonize yes   #这个必须设置
loglevel notice
dbfilename dump.rdb
protected-mode no
slaveof 127.0.0.1 6379  #从连接主
```

**配置3台sentinel哨兵**

主的哨兵/etc/redis-sentinel-26379.conf

```shell
# Sentinel节点的端口
port 26379  
dir /var/redis/data/
logfile "26379.log"

# 当前Sentinel节点监控 127.0.0.1:6379 这个主节点
# 2代表判断主节点失败至少需要2个Sentinel节点节点同意
# mymaster是主节点的别名
sentinel monitor mymaster 127.0.0.1 6379 2 #监视6379端口

#每个Sentinel节点都要定期PING命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过30000毫秒30s且没有回复，则判定不可达
sentinel down-after-milliseconds mymaster 30000

#当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，
原来的从节点会向新的主节点发起复制操作，限制每次向新的主节点发起复制操作的从节点个数为1
sentinel parallel-syncs mymaster 1

#故障转移超时时间为180000毫秒
sentinel failover-timeout mymaster 180000
```

从哨兵的配置和上面的一样，就是改一下端口

现在，我们的所有的配置文件如下：

```shell
[root@master tmp]# ll /etc/redis-*
/etc/redis-6379.conf
/etc/redis-6380.conf
/etc/redis-6381.conf
/etc/redis-sentinel-26379.conf
/etc/redis-sentinel-26380.conf
/etc/redis-sentinel-26381.conf
```

**启动哨兵**

```shell
redis-sentinel /etc/redis-sentinel-26379.conf
redis-sentinel /etc/redis-sentinel-26380.conf
redis-sentinel /etc/redis-sentinel-26381.conf
```

**查看哨兵**

```shell
[root@master ~]# redis-cli -p 26379  info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
#看到最后一条信息正确即成功了哨兵，哨兵主节点名字叫做mymaster，状态ok，监控地址是127.0.0.1:6379，有两个从节点，3个哨兵
```

**现在，杀死6379端口**

```shell
[root@iZpiur4diu91c9Z ~]#  redis-cli -p 26379  info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6381,slaves=2,sentinels=3
#我们发现现在的主变成了6381端口了，我们没有做操作，这里面的操作都是哨兵帮我们来做的。
```

**恢复6379端口，查看其状态**

```shell
# Replication
role:slave   #现在他变成了从
master_host:127.0.0.1
master_port:6381     #现在他的主就是6381
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:318129
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:bf049d11cdbb94782c744f4c27ba748abdba3c4e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:318129
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:312745
repl_backlog_histlen:5385
```

完成。

## redis-cluster配置

问题

```
redis官方生成可以达到 10万/每秒,每秒执行10万条命令
假如业务需要每秒100万的命令执行呢？
```

一台服务器内存正常是16~256G，假如你的业务需要500G内存，

**新浪微博作为世界上最大的redis存储，就超过1TB的数据，去哪买这么大的内存条？各大公司有自己的解决方案，推出各自的集群功能，核心思想都是将数据分片（sharding）存储在多个redis实例中，每一片就是一个redis实例。**

```
各大企业集群方案：
twemproxy由Twitter开源
Codis由豌豆荚开发，基于GO和C开发
redis-cluster官方3.0版本后的集群方案
```

### 客户端分片

redis3.0集群采用P2P模式，完全去中心化，将redis所有的key分成了16384个槽位，每个redis实例负责一部分slot，集群中的所有信息通过节点数据交换而更新。

**redis实例集群主要思想是将redis数据的key进行散列，通过hash函数特定的key会映射到指定的redis节点上**

**图例**

![](https://img2018.cnblogs.com/blog/1132884/201811/1132884-20181125134928388-1528161304.png)

### 数据分布理论

分布式数据库首要解决把整个数据集按照分区规则映射到多个节点的问题，即把数据集划分到多个节点上，每个节点负责整个数据的一个子集。

**常见的分区规则有哈希分区和顺序分区**。`Redis Cluster`采用哈希分区规则，因此接下来会讨论哈希分区规则。

- 结点取余分区
- 一致性哈希 分区
- 虚拟槽分区（**redis-cluster采用的方式)**





## python操作redis集群

### 简单操作

**使用strictRedis连接redis**

```python
python3
>>>import redis
>>>r = redis.StrictRedis(host='localhost', port=6379, db=0,password='xxx')
>>>r.set('name', 'pl')
True

#增删改查
>>> conn=redis.StrictRedis()
>>>
>>>
>>> conn.set("name1","alex1")
True
>>> conn.set("name2","wupeiqi")
True
>>>
>>>
>>> conn.set("name1","alex666")
True
>>> conn.delete("name2","name1")
2
>>> conn.keys()
[b'name3', b'name2', b'name1']


2、sentinel集群连接并操作

[root@db01 ~]# redis-server /data/6380/redis.conf
[root@db01 ~]# redis-server /data/6381/redis.conf
[root@db01 ~]# redis-server /data/6382/redis.conf  
[root@db01 ~]# redis-sentinel /data/26380/sentinel.conf
--------------------------------
## 导入redis sentinel包
>>> from redis.sentinel import Sentinel  
##指定sentinel的地址和端口号
>>> sentinel = Sentinel([('localhost', 26380)], socket_timeout=0.1)  
##测试，获取以下主库和从库的信息
>>> sentinel.discover_master('mymaster')  
>>> sentinel.discover_slaves('mymaster')  
##配置读写分离
#写节点
>>> master = sentinel.master_for('mymaster', socket_timeout=0.1)  
#读节点
>>> slave = sentinel.slave_for('mymaster', socket_timeout=0.1)  
###读写分离测试   key     
>>> master.set('oldboy', '123')  
>>> slave.get('oldboy')  
'123'

3、python连接rediscluster集群测试
使用

python3
>>> from rediscluster import StrictRedisCluster  
>>> startup_nodes = [{"host": "127.0.0.1", "port": "7000"}]  
### Note: decode_responses must be set to True when used with python3  
>>> rc = StrictRedisCluster(startup_nodes=startup_nodes, decode_responses=True)  
>>> rc.set("foo", "bar")  
True  
>>>   
'bar'
```

### redis储存session

安装模块

```
安装模块
pip3 install django-redis-sessions或者
pip3 install django-redis
```

配置settings.py文件

```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "PASSWORD": "",
#             "PARSER_CLASS": "redis.connection.HiredisParser",
#             "SOCKET_TIMEOUT": 10,
#             "CONNECTION_POOL_CLASS_KWARGS": {
#                 "max_connections": 2,
#             }
        }
    }
}
  
#SESSION_COOKIE_AGE = 30 * 60 #设置session过期时间为30分钟
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
```

编写路由

```python
from django.contrib import admin
from django.urls import path
from app01 import views
urlpatterns = [
    path('set_session/',views.set_session),
    path('get_session/',views.get_session),
    path('admin/', admin.site.urls),
]
```

编写视图

```python
from django.shortcuts import render,HttpResponse

def set_session(request):
    request.session['username']='chaoge'
    request.session['age']=18
    return HttpResponse("设置sesson成功")

def get_session(request):
    username=request.session['username']
    age = request.session['age']
    return HttpResponse(username+":"+str(age))
```

启动项目

```python
python3 manage.py runserver 0.0.0.0:8000
```

检查redis数据库，是否存在一条key

```python
127.0.0.1:6379> keys *
1) ":1:django.contrib.sessions.cachep220moqvxclz2hyjqmbybqs3v8ck2i39"

获取这个key的值
127.0.0.1:6379> get :1:django.contrib.sessions.cachep220moqvxclz2hyjqmbybqs3v8ck2i39
"\x80\x04\x95!\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\busername\x94\x8c\x06chaoge\x94\x8c\x03age\x94K\x12u."
```

