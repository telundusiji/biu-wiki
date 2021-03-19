> 操作系统：CentOS-7.8   
> Spark版本：2.4.4

本篇文章是对RDD的简单介绍，希望通过阅读本文你可以对RDD有一个初步认识和了解，帮助你在Spark的后续学习中更加轻松，如果你不知道什么是Spark可以先阅读[《一起学习Spark入门》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483994&idx=1&sn=be6e2285e346313c0a407dbbbcc6379b&chksm=e8d7adeddfa024fbbb075c256a890d2bf1e3141a6d97dffe14bb0f7c7dd8c8f6f83718a7ba5e&token=385574373&lang=zh_CN#rd)

### 1.RDD是什么？

RDD，全称 Resilient Distributed Datasets，弹性分布式数据集。RDD 是一个容错的，并行的分布式数据结构，可以让用户显式的将数据存储到磁盘和内存中，并对控制数据分区。

RDD不仅仅是一个数据集，它还提供了丰富的操作API可以用来操作数据集中的数据，对常见的数据运算有很好的支持。而且RDD混合了 Iterative Algorithms（迭代算法）、Relational Queries（关系查询）、MapReduce、Stream Processing（流式处理）四种编程模型，使得Spark的应用场景更加广泛。

RDD作为数据结构，它是一个只读的分区记录集合，一个RDD可以包含多个分区，每个分区就是一个Dataset片段。

多个RDD之间也可以相互依赖，当一个RDD的每个分区最多只能被一个子RDD的一个分区使用时，则称为窄依赖，若RDD的分区可以被多个子RDD的分区依赖时，则称为宽依赖，不同的操作依据其特性会产生不同的依赖

### 2.RDD为什么会出现

在RDD出现前，主流的计算方式是Hadoop MapReduce，由于MapReduce没有基于内存的数据共享方式，使得在进行迭代计算时，多个MapReduce任务之间只能通过磁盘进行数据共享，由于磁盘的性能远不如内存，所以MapReduce效率就显得比较底下

而在Spark出现后推出了分布式弹性数据集RDD解决了Hadoop MapReduce迭代计算效率底下的问题。在Spark中整个数据迭代计算的过程可以完全通过内存共享数据，而不需要将中间结果放在可靠的分布式文件系统中，这种方式在保证容错的前提下，提供了更灵活、更快速的任务执行。

### 3.RDD的特点

#### RDD不仅仅是数据集，也是编程模型

RDD既是一种数据结构，同时也提供了多种上层算子API。RDD的算子大致分为两类Transformation和Action算子，RDD是采用惰性求值的运算方式，在执行RDD的时候，在执行到Transformation操作时，并不会立刻进行运算，直到遇见Action操作时，触发真正的运算

* RDD允许用户显式的指定数据的存放位置(内存或者磁盘)  

* RDD是分布式的，用户可以控制RDD的分区

* RDD提供了丰富的操作

* RDD是混合型编程模型，支持迭代计算，关系查询，MapReduce，流计算

#### RDD是可分区的

先了解一下并行计算的四个要点

* 要解决的问题必须可以被分解为多个可以并发计算的问题

* 每个部分可以在不同的处理器上被同时执行

* 需要一个共享内存机制

* 需要一个总体上的协作机制来调度


RDD作为一个分布式数据集，也是一个分布式计算框架，并行计算也是它的一个主要功能，所以一定需要能够进行分区计算的，只有进行分区计算才能对任务进行拆解，充分利用集群的并行计算能力。

同时RDD不是一个必须被具体化的数据集，即RDD中可以没有数据，只要有足够的信息能够描述一个RDD是通过哪些运算计算得来的即可，这也是一种有效的容错方式

#### RDD是只读的

RDD是只读不可变的，它不允许任何形式的修改，每次的RDD的转换操作都是生成新的RDD。将RDD设计为只读的，可以降低问题的复杂度，同时对于容错、惰性求值以及移动计算也给予了更加好的支持

#### RDD的容错性

作为一个计算系统容错是需要考虑的一个重要方面，在RDD中的容错方式主要有两种方式

* 保存RDD之间的依赖关系，以及计算函数，当出现错误时重新计算。RDD之间的依赖关系，根据操作不同，可以分为宽依赖窄依赖两种

* RDD支持将其中的数据存放在外部存储之中，当出现错误时直接读取Checkpoint

### 4.RDD的弹性、分布式和数据集三个概念如何理解

#### 弹性

* RDD支持高效的容错

* RDD中的数据既可以缓存在内存中，又可以缓存在磁盘中，也可以缓存在外部存储中

#### 分布式

* RDD支持分区

* RDD支持分布式并行计算

#### 数据集

* RDD可以缓存起来，相当于存储具体的数据

* RDD可以不直接保留数据，而只保留数据的描述信息和创建该RDD的依赖信息，由这些信息也可以描述一个完整RDD

### 5.RDD的五个属性

* Partition List 分区列表，记录RDD的分区信息。分区数目可以在创建RDD的时候指定，也可以在一些转换操作中指定

* Compute Function 计算函数，RDD的转换是依赖于计算函数

* RDD Dependencies RDD的依赖关系，使用依赖关系可以进行RDD的容错和计算

* Partitioner 分区器，进行shuffle操作时，通过该函数来决定数据分到那个分区

* Preferred Location 优先位置，为了实现数据本地性操作，从而移动计算而不移动数据，需要记录每个RDD的分区应该放在什么位置

### 6.RDD算子分类

RDD的算子分为两大类：Transformation算子和Action算子

* Transformation算子，也叫转换算子，经过Transformation算子的操作，会在一个已经存在的RDD的基础上创建一个新的RDD，新的RDD中的数据是将旧的RDD通过转换算子所指定的函数对数据转换后得来的

* Action算子，也叫作行动算子，action操作会执行各个分区的计算任务，将结果返回到Driver端

算子的特点

* Transformation算子是惰性算子，在调用是不会立即进行运算获取结果，而只会记录在数据集上要进行的对于操作，当调用action算子时，才会真正进行计算，将多种转换操作通过调度分发到集群执行

* 默认情况下每一个action执行时，其所关联的Transformation操作都会被重新计算（多次重复执行相同RDD的相同action算子会浪费计算资源），不过RDD提供了persist方法可以将RDD持久化到磁盘或者内存中，加快多次访问相同RDD的速度，避免重复计算浪费计算资源


### 7.认识SparkContext

SparkContext是Spark Core的入口组件，是Spark程序的入口。可以把Spark程序看作服务端和交互端，那么服务端就是可以运行Spark程序的集群，而Driver端就是交互端，在Driver中SparkContext就是一个主要组件，也是Driver在运行时首要创建的组件。

SparkContext提供的API，涵盖了连接集群，创建RDD，数据累加器，广播变量等多种操作，也是用户与Spark程序交互的一个重要入口

SparkContext的创建

```java

/**实例化一个SparkConf，可以添加多种设置
* 包括的设置：spark程序名称
* master地址（local[*]代表本地,中括号中的 * 代表线程数）
* SparkHome的目录路径
* 应用要加载的jar包
* 以上是简略常用配置，SparkConf可以配置的配置项有很多，可以参考官网说明
*/
val sparkConf = new SparkConf()
	.setAppName("Spark Test")
	.setMaster("local[*]")
	.setSparkHome("{SPARK_HOME}")
	.setJars(Seq("jars"))
//SparkContext的创建可以直接使用new的方式，入参为SparkConf
val sc = new SparkContext(sparkConf)

```

### 8.RDD的创建方式

* 通过本地集合创建

```java

/**sc代表SparkContext
* 通过parallelize创建rdd，有两个入参
* 第一个参数是一个Sql集合，第二个参数是一个int类型，代表rdd的分区数
*/
val rdd1 = sc.parallelize(Seq("a1","a2"),2)
//通过makeRDD创建与parallelize入参含义一样
val rdd2 = sc.makeRDD(Seq(Seq("a1","a2"),2)

```

* 通过读取外部数据创建

```java

/**通过textFile读取外部文件，包含两个参数
* 第一个参数是文件路径：支持本地文件，hdfs文件，云文件系统
* 第二个参数是最小分区数，即读取的文件到RDD后分区数不小于指定分区数
*/
val rdd1 = sc.textFile("{path}",3)

```

* 通过其他RDD转换衍生而来

```java

//源RDD
val rdd1 = sc.parallelize(Seq("a1","a2"),2)
//由其他RDD转换衍生一个新的RDD
val rdd2 = rdd1.map(item=>item+"a")

```



### 总结

看完这篇文章，希望可以帮助大家对于RDD有一个初步的认识，可能现在大家对于RDD的原理还不是很懂，不过没关系，随着学习深入才会慢慢理解。

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
