
> 操作系统：CentOS-7.8  
> redis版本：6.0.5  

本篇锤子将和大家一起学习redis高可用，文章中会介绍和演示两种redis高可用服务的基本原理和搭建方式，帮助大家快速学习搭建使用redis服务。对redis不太熟悉的朋友可以参考上一篇[《一起学习Redis基础》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483943&idx=1&sn=21640da9e60e0544bb4d11451e6c9617&chksm=e8d7ad90dfa02486c102a5ae49b3cf62339eeb0a44ac2708a374e765acd8e34ffbf2a78fd1a3&token=571377584&lang=zh_CN#rd)

## 一、高可用方案

redis 服务实现高可用主要有两种方式：主从复制(Replication-Sentinel)和Redis集群(Redis Cluster)。下面对两种高可用方案进行原理介绍

### 1.主从复制与哨兵机制

单纯Redis主从复制并不是一个高可用方案，Redis主从复制模式下MASTER节点宕机后是没有重新选主机制，也就意味的服务部分功能不可用，而哨兵机制Redis中提供了监控节点健康状态的机制，所以主从复制模式结合哨兵机制才是一个真正高可用服务方案

#### 主从复制

主从复制模式下redis多个节点分为两种类型：主节点（Master）和从节点（Slave）。主节点负责读写工作，而从节点仅负责读工作，所以当主节点故障后，则服务就不可写了。

主从复制模式下有以下特点：

* Master节点既可以读也可以写，Slave节点只能读，不可以写

* Master实例与Slave实例之间的数据是实时同步的，即 Master中有数据变更时会与Slave进行交互，以便于将自身数据的变更复制给 Slave

* Slave与Master断开连接，过一段时间后再次连接Master时会同步自己断开时间段中Master中数据的更改部分，若无法部分同步数据，则Slave会请求全量同步数据

* 服务故障时，若Slave节点宕机，则读写服务可继续提供，若Master节点宕机，则只有读服务可继续提供

* 整个服务的读性能高，写性能受Master节点性能限制

* 整个服务存储上限受Master节点机器存储限制（多个节点数据互为副本，Master存储满了就无法继续存储）

#### 哨兵机制

哨兵机制（Redis Sentinel）是Redis社区提供的高可用解决方案，哨兵是一个独立的进程，它会实时监控Redis节点的健康状态，当Master不可用时会触发选主机制，从多个Slave节点中选出一个节点作为新的Master。

由于哨兵是独立的进程，所以在使用哨兵机制实现高可用时是需要部署两套集群即：哨兵集群和 Redis主从复制集群。

哨兵集群是由多个哨兵组成的分布式集群，主要有以下特点

* 监控redis节点的健康状况，发现节点故障

* 发现Redis节点故障时，可以发送提醒通知

* 当发现Master节点故障时，会进行故障自动转移，从Slave节点中重新选主，保证Redis服务集群可用

* 哨兵节点个数需满足奇数个，且至少为3个

* 一个哨兵集群可监控多套Redis服务集群

哨兵模式下还有两个概念：主观下线和客观下线

主观下线是在指定时间内哨兵（Sentinel） 没有收到目标节点的有效回复，则会判定该节点为主观下线，当大于等于半数个哨兵都将目标节点标记为主观下线，此时会判定目标节点客观下线，对于客观下线的节点若是主节点则会进行故障转移

### 2.集群模式

Redis集群模式是社区推出的 Redis 分布式集群解决方案，这样的一个集群是由多个主从节点群组成的分布式服务器群，它具有复制、高可用和分片特性。

相对于主从复制模式来说，Redis cluster模式是多主节点，且各个主节点的数据也不一样，数据采用分片的方式由多个主节点共同管理，每个主节点只管理部分数据。Redis默认划分了16384个hash槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每主节点只负责一部分hash槽的管理，这样也可以达到负载均衡的目的。

在Redis Cluster模式下的主节点提供读写操作，从节点作为备用节点不接收服务请求，只作为故障转移使用，在主节点与从节点之间也是采用主从复制机制，即一个主节点可以有多个从节点，当主节点故障时，会从当前主节点下的从节点中选主成为主节点继续服务。

Redis Cluster 的一个简略示意图如下所示

![redis cluster模式](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/redis_cluster.png)

从图中看出，Redis Cluster模式下是有多个主节点，每个主节点可以有多个从节点，且Redis自身在Cluster模式也做了节点数限制，节点最小配置 6 个节点（3 主 3 从）。

### 3.优缺点对比

在了解完上面两种模式后，我们来对比一下这两种模式各自的优缺点

![sentinel和cluster对比](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/sentinel_cluster.png)

## 二、搭建与配置演示

### 1.Replication-Sentinel

#### 环境准备

三台主机：RedisMaster（192.168.56.91）、RedisSlave1（192.168.56.92）、RedisSlave2（192.168.56.93）

主节点一个为RedisMaster，从节点两个为RedisSlave1和RedisSlave2

三个节点redis先进行基础安装（下载源码，解压，编译，基础配置等），基础安装参照[《学习必备——redis安装》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483935&idx=1&sn=0b4caf3c05e39271f2b1a551fb386929&chksm=e8d7ada8dfa024be82dc0aeab16790877a663632993e742c2e5b9c0c4d12b5b4a793b00ef3ab&token=440071545&lang=zh_CN#rd)

#### 配置操作

##### a）主从复制配置

三个节点的redis.conf配置如下

```

# 服务器端口号，默认就是6379
port 6379 
# 运行所有ip访问服务
bind 0.0.0.0 
# reids的密码 
requirepass "123456" 
# 主节点redis器密码 
masterauth "123456"

# 下面配置只需要配置slave节点
# 配置主服务器地址、端口号 
replicaof 192.168.56.91 

```

三个节点配置完成后，启动redis，使用客户端连接redis，查看三个节点服务信息如下：

![主从复制节点信息](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/replication.png)

从图中可以看出，我们配置三个节点正确运行，第一个为Mater节点后两个为slave节点

##### b）哨兵配置

在redis的编译目录下有 sentinel.conf 文件，三个节点配置sentinel.conf如下

```

#哨兵监听端口
port 26379
#哨兵启动后运行在后台
daemonize yes
# 配置哨兵，主节点名称（可自定义，但要与后面配置与其他节点哨兵一致），主节点ip，主节点redis端口，2个以上哨兵认为主节点不可用时进行故障转移
sentinel monitor redis-master 127.0.0.1 6379 2
# redis认证密码
sentinel auth-pass redis-mater 123456
# 被sentinel认定为不可用的间隔时间，单位毫秒
sentinel down-after-milliseconds redis-master 30000
# 剩余的slaves重新和新的master做同步时，进行同步的slave的并行个数
sentinel parallel-syncs redis-master 1
# 故障转移转移的超时时间，若执行故障转移的哨兵超过该时间仍未进行主节点切换，则会由其他的哨兵来处理
sentinel failover-timeout redis-master 180000

```

编译目录下启动哨兵：`./src/redis-sentinel sentinel.conf`

#### 验证

经过以上配置在所有都启动的情况下，我们停止掉redis-master（192.168.56.91）

经过哨兵故障转移结果如下图所示

![哨兵机制下主从复制故障转移验证](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/sentinel_check.png)

### 2.Redis Cluster

#### 环境准备

六台主机：Redis81（192.168.56.81）、Redis82（192.168.56.82）、Redis83（192.168.56.83），Redis84（192.168.56.84）、Redis85（192.168.56.85、Redis86（192.168.56.86）

六个节点redis先进行基础安装（下载源码，解压，编译，基础配置等），基础安装参照[《学习必备——redis安装》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483935&idx=1&sn=0b4caf3c05e39271f2b1a551fb386929&chksm=e8d7ada8dfa024be82dc0aeab16790877a663632993e742c2e5b9c0c4d12b5b4a793b00ef3ab&token=440071545&lang=zh_CN#rd)

#### 配置操作

对6个节点的redis.conf进行配置如下：

```

#redis监听端口
port 6379
#redis在后台运行
daemonize yes
#设置redis运行为非保护模式
protected-mode no
#开启集群模式
cluster-enabled yes
#节点配置文件，每个节点都有一个，由redis自己维护，在创建新集群时需把原来旧的删除
cluster-config-file node-6379.conf 
#节点检测超时时间，超过改时间不能通信则认为节点宕机
cluster-node-timeout 5000
#开启aof持久化
appendonly yes 


```

启动六个节点的redis服务，

构建redis cluster

```

#-a 认证密码，没有设置密码可以不用该参数
#--cluster create 创建集群
#--cluster-replicas 集群副本节点数
redis-cli -a 123456 --cluster create --cluster-replicas 1 192.168.56.81:6379 192.168.56.82:6379 192.168.56.83:6379 192.168.56.84:6379 192.168.56.85:6379 192.168.56.86:6379

```

查看集群信息

```

#-a 认证密码，没有设密码，不用该参数
#--cluster check 指定集群中一个节点
redis-cli -a 123456 --cluster check 192.168.56.81:6379

```

查看结果如图

![redis cluster信息查看](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/cluster_info.png)

从图中我们可以看出：M代表Master节点，S代表slave节点，16384个槽平均分配在三个Master节点上，slave节点没有槽。

#### 验证

我们停掉集群中的一个节点，来验证

![Redis cluster高可用验证](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/cluster_check.png)

#### 其他配置

1.reshard

默认分片机制是均分，也就是多个master节点均分16834个槽，当默认均分策略不满足需求时，可以使用reshard调整分片，命令如下

```

#-a 认证密码，无密码不使用该参数
#--cluster reshard 进行分片槽调整操作，指定集群中任意节点
redis-cli -a 123456 --cluster reshard 192.168.56.81:6379

```

操作如下图所示

![Redis cluster下充分片](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Redis%E9%AB%98%E5%8F%AF%E7%94%A8/reshard.png)

2.节点管理

首先配置安装上面集群模式方式安装配置一个新的节点，然后使用如下命令将新节点添加到集群中

```

#添加节点
#newNodeIP:newNodePort 新节点ip和端口
#clusterNodeIp:clusterNodePort 原有集群中任意一个节点的ip和端口
redis-cli -a 123456 --cluster add-node newNodeIP:newNodePort clusterNodeIp:clusterNodePort

```

经过如上配置将节点加入集群，若将新增节点作为主节点则使用reshard给新加入节点分配槽即可，若将该节点作为从节点，则使用redis-cli连接新节点，使用`cluster replicate <MasterNodeId>` 将新节点指定某个主机的从节点

从集群中移除一个节点使用如下命令

```

#clusterNodeIp:clusterNodePort 集群中任意一个节点的ip和端口
#removeNodeId 要移除的节点id
redis-cli -a 123456 --cluster del-node clusterNodeIp:clusterNodePort removeNodeId

```

### 总结

到这里本篇内容结束，在本篇中主要介绍redis的两种高可用搭建方式，对其概念和理论进行介绍，同时也进行了演示两者redis高可用服务搭建方式并进行了验证。在下一篇中锤子将和大家一起学习在java中如何使用Redis

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ