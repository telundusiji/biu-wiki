
> 操作系统：CentOS-7.8  
> redis版本：6.0.5  

本篇锤子将和大家一起学习redis的基础知识，文章中只会学一些重要的基础信息，对于redis的详细了解可以浏览redis的官网[https://redis.io/](https://redis.io/)

## 一、概念

Redis是一个开源基于内存的数据结构存储。可以用作数据库、缓存和消息代理。它支持多种数据结构，如字符串(String)、哈希(Hash)、列表(List)、集合(Set)、有序集合(Sorted Set)、位图(Bitmaps)、HyperLogLog、带有半径查询的地理空间索引(geospatial indexes with radius queries)、流(Streams)。Redis内置了副本机制，支持Lua脚本，采用LRU算法作为淘汰算法、支持事务和不同级别数据持久化到磁盘策略，并提供了哨兵机制(Sentinel )和redis集群自动分区机制两种方式来保证服务高可用性

### 1.NoSQL

NoSQL（Not-Only SQL）泛指非关系性数据库。与之相对应的就是关系型数据库，随着互联网的发展，网络用户群体越来越大，数据量的激增以及复杂多变的应用场景，传统的关系型数据库已经暴露出了很多难以克服的问题，比如数据库的高并发读写，海量数据的高效率存储和访问，多变的数据结构等，这些问题的解决方案仅仅使用传统关系型数据库已经无法满足了，于是作为关系型数据库的补充，非关系型数据就扮演了另一个角色来解决这些关系型数据库无法解决的问题。

下面是一个关系型数据库和非关系型数据库的简单对比

![关系型数据库与非关系型数据库简单对比](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E5%9F%BA%E7%A1%80/rdbms_nosql.png)


从图中我们可以简单了解到，非关系型数据库擅长于高并发场景下的数据读写，适合存储大数据量的数据，同时也易于拓展，但是对数据一致性要求高的数据存储时非关系型数据库是不适用的，像银行的交易系统的账户信息，电商平台的支付订单信息等大部分都还是采用关系型数据库来存储。所以在现有的大部分应用中，都是采用关系型数据库和非关系型数据库结合的一种组合来满足复杂的业务场景。

### 2.redis的特点

redis作为非关系型数据库的一种，属于键值型数据存储，因为redis支持高并发的快速数据访问，所以在生产中多用来作为缓存使用，以提高数据读取效率，提高整个应用程序的并发性能。redis主要有以下特点

* 支持多种数据结构

* 支持持久化操作，支持用AOF和RDB两种数据持久化策略把数据持久化到磁盘

* 单线程请求，并发请求情况下不需要考虑数据一致性问题。

* 支持两种高可用方式：哨兵机制(master-slave模式)和redis cluster高可用

* 支持发布与订阅的消息机制(不建议使用redis来做消息中间件)

* 支持简单事务

### 3.redis的线程模型

> 说到redis的线程模型，大部分朋友可能都听说过redis是单线程模型，但其实这个说法是不准确的，整个redis服务当然不可能是一个单线程的，仅仅是因为redis的文件事件分派器是单线程消费事件队列的，所以很多人会称Redis是单线程模型。其实在redis服务中还有ServerSocket监听、IO多路复用程序等，这些模块都至少需要一个线程来工作，所以在学习的时候，我是不会直接称呼redis是单线程模型，这样会让不是很了解redis的朋友产生误会，不能理解一个线程怎么实现的整个服务，这里仅做一个说明，避免读者朋友误会，接下来是正文

#### 线程模型概述

整个处理模型由4部分组成：多个套接字、IO多路复用器、文件事件分派器、文件事件处理器

如下图是redis的文件事件处理器简略表示

![线程模型](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E5%9F%BA%E7%A1%80/process_model.png)

我们大概描述一下这个流程

* 首先，在redis服务启动后，服务中会有一个ServerSocket来负责监听服务端口

* 有一个客户端连接到Redis的监听端口，客户端与服务端建立通信在服务端会生成一个Socket，这个Socket是与客户端进行数据交换的通道

* 通信建立后，端口监听程序将Socket进行业务封装交给IO多路复用程序，

* IO多路复用程序根据自己的逻辑(不同的多路复用实现，具体操作不一样)会对每个客户端通信的消息进行监听，根据客户端发送过来不同的请求消息，将消息与通信封装成不同的事件，再将事件放入事件队列。

* 文件事件分派器消费事件队列，从队列中出队一个事件，则根据事件类型不同，将事件交由不同的事件处理器处理（这个过程是单线程的），处理完一个事件后，再去队列出队下一个，重复下去....

* 当客户端断开与服务端的通信时，IO多路复用器检测到与客户端的通信已断开，也就将该客户端信息移除，就不再监听该客户端信息了

以上是一个简略的描述过程，对非阻塞IO有一定了解的朋友或许会看的更明白一些，其核心就是IO多路复用与事件队列的单线程消费。

#### IO多路复用器

常见的IO多路复用器有select、epoll、kqueue等，Redis的IO多路复用程序是通过包装这些常见的IO多路复用器来实现的，下面简单说一下多路复用IO

多路复用IO有时也被称为事件驱动IO，而IO多路复用器就是多路复用IO的实现，其代表实现有select、epoll、kqueue（上面也提到过），使用这些IO多路复用器的好处在于单个process就可以同时处理多个网络连接的IO，这样合理的使用多个IO多路复用器就可以较小资源实现高并发网络编程。通俗的理解IO多路复用器就是利用单个线程或者少量线程，管理大量的Socket通信，从而减少了因为IO等待(阻塞IO下，当对方操作数据时，当前线程就处于阻塞等待状态)带来的资源浪费。

在这三个代表性的IO多路复用器中，select是采用轮询机制来检测多个Socket中是否有数据交互产生，而epoll和kqueue则是使用的callback的方式。当Socket比较多的时候，select要通过遍历每个Socket来完成调度,不论当前活跃的Socket有多少，都会全部遍历一遍，这种设计会导致CPU时间的浪费，所以在后面的epoll和kqueue多路复用器中就换了另外一种方式，它们会给Socket注册回调函数，当Socket活跃的时候，会自动完成数据处理相关操作，这样就避免了轮询代理的CPU时间浪费

#### 文件事件处理器

* 连接应答处理器——对客服端连接服务器进行应答

* 命令请求处理器——接受客户端传来的命令请求

* 命令回复处理器——向客户端返回命令执行结果

### 4.redis持久化

Redis有两种持久化策略RDB（Redis DataBase）和AOF（Append Only File），这两只持久化策略可以选择单独使用，也可以结合使用，当然也可以不使用。如果在redis中不使用持久化，则redis的中缓存的数据的生命时长最长就是redis运行的时间，即redis停止后缓存的数据就全部丢失了

#### RDB

RDB持久化是以指定的时间间隔执行数据集的时间点快照，通俗讲就是每隔一段时间，redis会把把内存中的数据写入磁盘的临时文件，作为快照，等需要恢复的时候再把快照文件读进内存

#### AOF

AOF持久化是记录服务器接收到的每个写入操作，这些操作将在服务器启动时再次加载，重建原始数据集，命令的记录以追加的方式，使用的格式与Redis协议本身的格式相同，Redis能够在日志太大的时候重写这些记录文件

#### 优缺点对比

![RDB与AOF对比](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E5%9F%BA%E7%A1%80/rdb_aof.png)


## 二、基础操作

### 1.简单安装

redis单机单实例安装参考[《学习必备——redis安装》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483935&idx=1&sn=0b4caf3c05e39271f2b1a551fb386929&chksm=e8d7ada8dfa024be82dc0aeab16790877a663632993e742c2e5b9c0c4d12b5b4a793b00ef3ab&token=440071545&lang=zh_CN#rd)


### 2.命令行操作使用

使用 redis-cli 连接redis服务

```

#-h 连接的redis主机名
#-p 连接的redis端口号
#-a 连接的redis密码，也可以不指定密码，在redis命令行中使用 'auth 密码' 来认证
redis-cli -h localhost -p 6379 -a 123456

```

#### 基础命令

```

#列出匹配的key，可以使用通配符
keys *
#设置一个key-value
set key value
#根据key获取一个value
get key
#删除一个key
del key
#切换数据库，默认总共16个
select index
#删除当前数据库下数据
flushdb
#删除所有数据库下数据
flushall

```

#### String类型

字符串类型，限制最大值为512M，包含以下操作

```

#设置key-value，若key已经存在，则覆盖原有数据
set key value
#设置key-value，若key已经存在，则不进行任何更改
setnx key data
#设置带过期时间的key，时间单位为秒
setex key second value
#设置key-value，使用过期时间，ex 后面跟时间时单位为秒，px后面跟时间单位是毫秒
set key value ex|px time
#查看key过期剩余时间
ttl key
#获取key的value的字符串长度
strlen key
#截取字符串，end为-1时，代表截取到最后
getrange key start end
#字符串替换，将newdata替换由于值的start位置开始
setrange key start newdata
#将新的value追加到原有key的value后面，若key不存在，则直接创建key将value设置进去
append key value
#当key的value可以格式成数字类型时，value累加1，否则报错
incr key
#当key的value可以格式成数字类型时，value累减1，否则报错
decr key
#当key的value可以格式成数字类型时，value累加指定的num值，否则报错
incrby key num
#当key的value可以格式成数字类型时，value累加指定的num值，否则报错
decrby key num
#批量设置key-value值
mset [key value]
#批量获取
mget [key]

```

#### Hash类型

Hash是一个键对应多个属性和value，大小无限制（视内存大小而定），包含操作如下：

```

#设置key和其对应的多个属性和value
hset key [field value]
#示例
#hset user id 1 name 锤子

#设置对象中的多个属性键值对，存在的属性键值不会被更改
hmsetnx key [field value]

#获取key下的属性field 的值
hget key field 

#获取key下多个属性field 的值
hmget key [field]

#或者整个key下所有属性和值
hgetall key

#对key下指定属性累加整数值num
hincrby key field num

#对key下指定属性累加浮点数值num
hincrbyfloat key field num

#查看key下属性个数
hlen key

#判断key下属性是否存在
hexists key field

#获取key下所有属性名称
hkeys key
#获取key下所有属性值
hvals key

```

#### List类型

List类型即列表类型，redis中的list实现是基于链表，包含操作如下：

```

#构建一个list，从左边开始存入数据
lpush key [data]
#实例lpush test 1 2 3 4
#则从左依次放入后结果为：[4 3 2 1]

#构建一个list，从右边开始存入数据
rpush key [data]
#实例rpush test 1 2 3 4
#则从左依次放入后结果为：[1 2 3 4]

#获取数据，指定其实位置，end为-1代表为最后
lrange key start end

#从左侧开始拿出一个数据
lpop key
#从右侧开始拿出一个数据
rpop key

#查看list长度
llen key

#或者指定索引下标的值
lindex key index

#替换指定下标的值
lset key index value

#在list的指定索引下标的元素前面或者后面插入值
linsert key before/after index value

#移除指定数量和指定值的列表元素
lrem key count element

#对list进行剪裁，不在指定范围的元素会被删除
ltrim key start end

```

#### Set类型

Set类型是无序集合，通过hash表实现的，包含如下操作：

```

#添加集合和元素，会自动去除重复的member
sadd key [member]

#查看集合所有成员
smember key

#查看集合成员数量
scard key

#判断成员是否在集合中，1代表存在，0代表不存在
sismember key member

#删除指定成员
srem key member

#移除并返回集合中指定数量的元素
spop key count

#将成员从一个集合移动到另一个集合
smove sourcekey targetkey member

#对多个集合做差集
sdiff key1 [key]

#多个几个做交集
sinter key1 [key]

#多个集合做并集
sunion key1 [key]

```

#### Zset类型

Zset类型，有序集合类型，使用HashMap和跳跃表(SkipList)来保证数据的存储和有序，HashMap里放的是成员到score的映射，而跳跃表里存放的是所有的成员，排序的依据是HashMap里存的score

```

#设置集合并添加成员和对应的分数
zadd key [score member]

#查看集合中内容，当使用withscores时也会返回分数
zrange key start end [withscores]

#查询指定member的索引下标
zrank key member

#查询指定member的分数
zscore key member

#获取集合成员个数
zcard key

#统计分数在指定区间的成员个数
zcount key startscore endscore

#获取分数在指定区间的成员，当在startscore和endscore前面加上'(' 则表示不包含起止分数，
#不加时默认包含起止分数
zrangebyscore key startscore endscore

#获取分数在指定区间的成员，当在startscore和endscore前面加上'(' 则表示不包含起止分数，
#不加时默认包含起止分数，在获取的结果中再使用start和end下标限制
zrangebyscore key startscore endscore limit start end

#删除成员
zrem key member

```

#### 消息发布与订阅操作

订阅操作

```

#指定订阅的频道
subscribe [channel] 

```


发布操作

```

#向指定频道发布消息
publish channel message

```

### 总结

在本篇文章中主要是对redis概念进行了讲解，包括redis是什么以及redis的一些简单原理和机制，然后就是熟悉一些redis的基础操作，通过本篇学习，我们可以对redis有一个简单的认识，在熟悉其操作后，在去更进一步理解


> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片


> 觉得不错就点个赞叭QAQ