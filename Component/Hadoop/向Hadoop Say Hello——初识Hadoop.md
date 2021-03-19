### 一、概述

> Hadoop这个名字的由来是Hadoopde之父Doug Cutting的孩子给一个棕黄色大象样子的玩具起的名字  
> Hadoop官网地址[http://hadoop.apache.org/](http://hadoop.apache.org/)

#### 什么是Hadoop？
> 官网原话：The Apache™ Hadoop® project develops open-source software for reliable, scalable, distributed computing.    
> 翻译过来：Apache的Hadoop项目是一个可靠的，可拓展的分布式计算开源软件

Hadoop 的功能是利用服务器集群，根据用户自定义业务逻辑对海量数据进行分布式处理。它包括四个核心部分：Hadoop Common、Hadoop Distributed File System（HDFS）、Hadoop YARN、Hadoop MapReduce。  

> * Hadoop Commmon：支持其他Hadoop模块的通用功能  
> * HDFS：分布式文件系统，可提供对应用程序数据的高吞吐量访问  
> * Hadoop YARN：作业调度和集群资源管理的框架  
> * Hadoop MapReduce：基于YARN的并行处理大型数据集的框架  

狭义Hadoop是指集分布式文件系统（HDFS）和分布式计算（MapReduce）以及集群资源调度管理框架（YARN）的一个软件平台。  
广义的Hapdoop指的是Hadoop生态系统，在Hadoop生态中Hadoop是重要和基础的一个部分，生态中包含了很多子系统，每一个子系统只能解决某一个特定的问题域。  

Hadoop核心组件HDFS和YARN都是采用主从架构
> 在一个集群中，会有部分节点充当主服务器的角色，其他服务器是从服务器的角色，这种架构模式就叫*主从结构*

* HDFS 的主节点是NameNode，从节点是DataNode
* YARN 的主节点是ResourceManager，从节点是NodeManager

### 二、核心组件——HDFS

> * 源自于Google在2003年10月的GFS论文  
> * HDFS是GFS的一个开源实现版本

HDFS是一个分布式的文件系统，其设计的核心思想：**分散均匀存储 + 备份冗余存储**。HDFS会把一个大文件按照blocksize（块大小）的要求将其拆分成多个block(块)，并以多副本的方式存储在HDFS集群中的多台服务器的本地硬盘上，通过统一的命名空间来定位文件、由很多服务器联合起来实现其功能。在一个HDFS集群中包含两个重要的部分：*NameNode*、*DateNode*

* **NameNode（NN）**是HDFS集群的主节点，一个HDFS集群中只有一个NN，负责维护目录结构和文件分块信息，同时还负责接收客户端的请求进行处理

* **SecondaryNameNode（SNN）**是NN的一个冷备份，可以看做是一个CheckPoint，它定时到NN去获取edit logs，合并 fsimage和fsedits，并推送给NameNode，在NameNode故障时，SNN不能立即替代NN工作，但是它作为NN的一个冷备份是防止数据完全丢失，可以协助NN恢复

> 对namenode的操作都放在edits中,相当于一个文件操作的记录  
> fsimage是namenode中关于元数据的镜像，一般称为检查点

* **DataNode（DN）**是HDFS集群的从节点，在一个集群中可以有多个DataNode，它是负责各个文件block的存储管理，执行数据块的读写操作。

#### 特性

* 文件在物理上是分块并以多副本的方式存储

* HDFS文件系统给客户端提供统一的抽象目录树，数据切分、多副本、容错等操作对用户是透明的

* 适用于一次写入多次读出的场景，不支持文件修改

* 适合存储大文件，不适合存储小文件（每个小文件都是一个block块，存储相同大小的文件，小文件会使用更多的块，增大NN的负担）

* 分散均匀存储和备份冗余存储的设计思想，保证了存储在HDFS上的数据高可用

#### 劣势

* 数据访问延迟高

* 小文件存储不友好

* 单用户写入不支持修改

### 三、核心组件——MapReduce

> 源自于Google在2004年12月发表的MapReduce论文  
> MapReduce是Google MapReduce的一个开源实现版本

MapReduce是一个分布式计算编程框架，核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上。

一个MapReduce作业主要分为两部分Map（映射）和Reduce（归约）。首先把输入数据集切分成若干独立数据块，然后将数据块分给多个Map任务并行处理，将map并行处理的结果输入给reduce任务进行处理。  

MapReduce作业的输入和输出都会被存储在文件系统中，一般情况下运行MapReduce框架和运行HDFS文件系统的节点通常是在一起的。这种配置允许框架在那些已经存好数据的节点上高效地调度任务，这可以使整个集群的网络带宽被非常高效地利用。*当然MapReduce作业读取的数据文件并不一定要求是在HDFS上，这是由用户指定的，默认是HDFS*。

#### MapReduce作业的过程描述

* 输入文件的存储位置

* InputFormat接口可以设置文件分割的逻辑，对文件进行分割，将分割后的文件输送给Mapper

* Map读取Input的key-value，根据用户自定义逻辑进行计算，对Key和Value重新映射，产生新的key-value

* 经过Map后所产生的新的key-value会按key进行分区，并写入一个环形内存缓冲区中

* 当环形缓冲区存满后，会生成临时文件，将数据写在磁盘上（这个过程叫溢写，本文只是概述不再详细讲解，在后续文章会细讲）

* Map中所有数据处理完毕后，会将所有临时文件进行合并生成只生成一个数据文件

* 进入shuffle过程，将map的结果按照key进行分组，将相同的key的数据放在一起

* 在reduce函数中数据以 key-value 列表输入，根据用户自定义逻辑计算，产生新的key-value数据，将其作为输出结果存储在HDFS上

### 四、核心组件——YARN

YARN是一个分布式的资源管理和作业调度框架，负责将自己管理的系统资源分配给在集群中运行的应用程序，并调度在不同集群节点上执行的任务。YARN的设计是主从结构，包含两个主要服务：*ResourceManager、NodeManager*，还有两个重要概念：*ApplicationMaster、Container*

* **ResourceManager（RM）**是YARN的主节点服务，是在系统中的所有应用程序之间仲裁资源的最终权限  

* **NodeManager（NM）**是YARN服务的从节点，是每个节点上的资源和任务管理器，负责 Containers的启动停止以及运行状态监控，监视其当前节点资源使用情况（CPU，内存，磁盘，网络）并将其报告给 RM

* **ApplicationMaster（AM）**是每个用户程序的管理者，每个运行在YARN上的应用程序都有一个AM，负责与RM协商进行资源获取，负责监控、管理Application在各个节点上的具体运行的内部任务

* **Container**封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是使用Container作为单位表示。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源

### 五、其他

#### 优势

* 可靠性

> 数据存储：数据块多副本，NameNode主备设计  
> 数据计算：失败任务会重新调度计算

* 可拓展性

> 可横向的线性拓展集群节点  
> 集群中节点的个数可以数以千计 

* 存储在廉价机器上，成本低

* 有一个相对成熟的生态圈

#### 常用发行版

* Apache版本

> 完全开源  
> 不同版本、不同框架的整合比较麻烦

* CDH发行版

> cloudera manager可视化安装，组件管理方便
> cloudera manager不开源，且组件与Apache社区版稍有改动

* HortonWorks（HDP）

> 原装Hadoop、纯开源、可以基于页面框架自己定制改造
> 企业级安全服务不开源

.

> 文章欢迎转载，转载请注明出处，个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

