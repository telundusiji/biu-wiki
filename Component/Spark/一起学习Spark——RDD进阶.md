> 操作系统：CentOS-7.8   
> Spark版本：2.4.4

本篇内容我们将更加深入的了解RDD，在本篇中我们将学习RDD的分区、缓存和Checkpoint，通过本篇学习让大家对RDD有更深的了解，同时在工作中可以更好使用RDD的相关功能。对于Spark和RDD不了解的朋友可以参见前几篇

* [《一起学习Spark入门》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483994&idx=1&sn=be6e2285e346313c0a407dbbbcc6379b&chksm=e8d7adeddfa024fbbb075c256a890d2bf1e3141a6d97dffe14bb0f7c7dd8c8f6f83718a7ba5e&token=738234644&lang=zh_CN#rd)

* [《一起学习Spark——RDD入门》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483999&idx=1&sn=996f90b6c6cf40e7e6cf54d8589bdf6d&chksm=e8d7ade8dfa024fe37889627746e365948036d047204de04e81e7eb8179b8f53d4503aa27f33&token=738234644&lang=zh_CN#rd)

* [《一起学习Spark——Transformation算子》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247484005&idx=1&sn=94c9140ffae93a1e2f0c6565afc6066f&chksm=e8d7add2dfa024c41ff18bda185e4b34fad64530c6670c457f54dcca1f8506c6a11bd84a1523&token=738234644&lang=zh_CN#rd)

* [《一起学习Spark——Action算子》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247484013&idx=1&sn=3301d2936d0d3cfb2341b9ff029ded94&chksm=e8d7addadfa024ccb4f2942c279d554840ec14966ea160defb87c6222777427463e73cce4f99&token=738234644&lang=zh_CN#rd)

## 一、RDD的分区

### 1.什么是RDD分区

在RDD中分区是一个逻辑概念。理解RDD的分区时，你可以将RDD看成一个存储数据的数据集，那么这个数据集中存储着大量的数据，这些数据在RDD中又被分开在多个地方存放，那么每一个存放部分数据的位置也就可以认为是RDD的分区。仅从数据集的角度来理解，一个RDD中的所有数据就是由多个分区来保存，每个分区中仅包含一个RDD中的部分数据，一个RDD中的所有分区共同保存了整个数据集数据。

> 提示：在理解RDD分区时你可以认为RDD就是一个单纯的数据集合，但是千万不要忘记了RDD不是一个单纯的数据集，它更是一个分布式的编程模型

我们拿HDFS文件来类比一下，在HDFS上一个文件可以有多个块组成，每个块保存一个文件的部分数据，一个文件下的所有块就保存着整个文件的数据，那么类比RDD与分区的关系就与HDFS的文件和块的关系类似。

### 2.RDD分区的作用

RDD的分区设计主要是用来支持分布式并行处理数据的。我们在之前一篇文章提到过并行计算的一个要素：“解决的问题必须可以被分解为多个可以并发计算的问题”，那么RDD作为一个支持分布式并行运算的数据集，不同的分区可以被分发到不同的计算节点上并行执行，这也是体现了并行计算中问题可分解的要素。

RDD在使用分区来并行处理数据时, 是要做到尽量少的在不同的 Executor 之间使用网络交换数据, 所以当使用 RDD 计算时，会尽可能地把计算分配到在物理上靠近数据的位置。由于分区是逻辑概念，你可以理解为，当分区划分完成后，在计算调度时会根据不同分区所对应的数据的实际物理位置，会将相应的计算任务调度到离实际数据存储位置尽量近的计算节点


### 3.RDD的分区操作

#### 查看RDD的分区数据

```java

//查看分区数，两种方式
rdd.getNumPartitions

rdd.partitions.size

```

#### 指定RDD的分区

##### 在创建RDD的时候指定分区

```java

//创建rdd时指定分区，不指定时则spark会使用默认分区数(跟计算节点以及数据集的大小有关)
sparkContext.parallelize(data,6)

sparkContext.textFile("${filePath}", 6)

```

##### 在使用转换算子的过程中指定分区

```java

//在进行groupby操作时可以重新转换生成的新RDD的分区数，部分算子可以在使用的时候指定分区，这里只列举了一个

rdd.groupBy(function,5)

```

##### 使用rdd提供的分区函数指定分区

> coalesce和repartition两个修改分区数算子的作用讲解，我们在[《一起学习Spark——Transformation算子》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&tempkey=MTA3N18vZmlBSjZkTEROS1llYm5Wc096TUFKWlBTNEpYLUpyRkxkcVhhMS1MV3J0dmd1eTlnTXNURlZuLUtfYTZfOE03Z0JsV083Y2lSbmRqZmhCc3RaSVBEMXduRjBDMmxZRUludUxzRmxpT3RpeERFb1QzbVNWdnJPWkZVWlVGbmZsVFdlUF9QZVlVVThUaG5aLUxCRGxVQVJpckV5bV9WLUpjNi1VazlRfn4%3D&chksm=68d7addc5fa024ca674f3493842a631085b7178131976dee9e62919c8ec6cb976156b3b68200#rd) 里面有详细介绍过，所以这里仅简略演示一下

```java

/**
* 使用coalesce算子，进行重分区
* coalesce有两个参数，第一个是分区数，第二个是分区时是否允许shuffle
* 第二个参数默认是 false：不允许shuffle，则这种情况下，coalesce只能减少分区数，不能增大分区数
*/
rdd.coalesce(5,true)

//强制对rdd重分区，属于shuffle操作
data.repartition(5)

```

#### 分区函数（Partitioner）

分区函数用于RDD的shuffle操作时确定一条数据应该所属的分区。RDD的分区函数只作用于K-V类型数据，非K-V类型的数据在进行操作时，不需要将数据分发到不同分区，即使进行强制分区调整，也只需根据数据量即可保证每个分区的数据相对均衡，而且多行数据之间也没有关联操作。而对于K-V类型数据，则在进行shuffle操作时，需要将相同key的数据发往同一个分区，则此时确定某一key应该发往哪个分区就由分区函数计算

RDD在对K-V类型进行转换操作，可以指定生成新RDD的分区函数，默认不指定分区函数的情况下则使用的是HashPartitioner，即将数据的key进行hash转换后根据分区的个数将数据散列到多个分区中。

下面演示实现自定义实现一个分区函数

```java

/**
 * 处理key为时间戳类型的数据，将key按照月份格式化后分配到不同的分区
 */
class MonthPartition(np: Int) extends Partitioner {

  val format = new SimpleDateFormat("MM")

  override def numPartitions: Int = {
    np
  }

  override def getPartition(key: Any): Int = {
    val month = format.format(new Date(key.toString.toLong)).toInt
    month % np
  }
}

object MonthPartition {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("month partition test")
      .master("local[6]")
      .getOrCreate()

    spark.sparkContext.setLogLevel("ERROR")

    val format = new SimpleDateFormat("yyyyMMdd")

    val sourceData = spark.sparkContext.makeRDD(Seq(
      (format.parse("20200809").getTime, "8月"),
      (format.parse("20200709").getTime, "7月"),
      (format.parse("20200209").getTime, "2月"),
      (format.parse("20201009").getTime, "10月"),
      (format.parse("20200509").getTime, "5月")
    ), 3)

    //创建rdd后,数据的分区情况
    println("创建rdd后,数据的分区情况")
    sourceData
      .mapPartitionsWithIndex((index, partition) => {
        partition.foreach(item => {
          println(s"partitionIndex:${index},key:${item._1},value:${item._2}")
        })
        partition
      }).count()

    //按key分组后，并使用自定义的按月份分区函数后的数据分区情况
    println("按key分组后，并使用自定义的按月份分区函数后的数据分区情况")
    sourceData.groupByKey(new MonthPartition(12))
      .mapPartitionsWithIndex((index, partition) => {
        partition.foreach(item => {
          item._2.foreach(x => {
            println(s"partitionIndex:${index},key:${item._1},value:${x}")
          })
        })
        partition
      }).count()
  }
}


//最后运行的结果如下
/*
创建rdd后,数据的分区情况
partitionIndex:0,key:1596902400000,value:8月
partitionIndex:2,key:1602172800000,value:10月
partitionIndex:1,key:1594224000000,value:7月
partitionIndex:2,key:1588953600000,value:5月
partitionIndex:1,key:1581177600000,value:2月
按key分组后，并使用自定义的按月份分区函数后的数据分区情况
partitionIndex:2,key:1581177600000,value:2月
partitionIndex:5,key:1588953600000,value:5月
partitionIndex:8,key:1596902400000,value:8月
partitionIndex:7,key:1594224000000,value:7月
partitionIndex:10,key:1602172800000,value:10月
*/

```

## 二、RDD缓存

### 1.缓存的意义

在不同操作中可以在内存或者文件系统中持久化或者缓存数据集，这是Spark计算速度快的原因之一，RDD持久化后，每一个计算节点都将把计算分区结果保存在内存中，对RDD或其衍生RDD进行计算时可以减少重复计算，这是缓存的第一个作用“提速”，而同样RDD的缓存不仅限于内存，RDD同样可以缓存在文件系统中，缓存在文件系统中的RDD数据，可以在计算发生错误时，保证中间计算数据的不丢失，这就是缓存的另一个作用“容错”

#### 节省计算资源，加快计算速度

在为进行rdd缓存和持久化的情况下RDD的每次action操作都会将其依赖链上的所有rdd计算一遍，这样的计算方式会浪费计算资源，使得整个应用的计算速度变慢。我们举个例子如下：

```java

rdd1 = sc.makeRDD()

rdd2 = rdd1.map()

rdd3 = rdd2.flatMap()

rdd4 = rdd3.filter()

rdd5 = rdd3.map()

rdd4.count()

rdd5.count()

```

如上述示例，我们的rdd5和rdd4都是依赖rdd3，在rdd4进行count操作时，会计算rdd1->rdd2->rdd3->rdd4，在rdd5进行count操作时，会计算rdd1->rdd2->rdd3->rdd5，我们可以看出，两次action操作中都存在计算rdd1->rdd2-rdd3的过程，由于RDD是不可变的，所以在两次的计算过程中就存在重复计算，会造成计算资源浪费。

为了避免这种重复计算造成的计算资源浪费，我们可以将rdd3缓存起来，那么在rdd4进行count时，会把rdd1->rdd2->rdd3的计算进行一次，在rdd5再进行count时，之前重复的计算就不会再进行，而是直接计算rdd3->rdd5，这样就节省了计算资源，提高了效率

#### 容错

容错即可以保证中间计算数据不丢失，多用于提供网络服务的spark程序，有一个服务接口对外提供服务，每次请求该接口都需要使用某一rdd进行计算，则将该RDD进行多副本缓存不仅可以加快其计算效率，还可以在部分计算节点故障时，副本的数据可让服务继续在 RDD 上运行任务，而无需等待重新计算丢失的分区


### 2.缓存的使用

#### 使用cache方法进行缓存

cache方式是rdd的方法，在一个rdd中可以直接调用，该方法定义源码如下：

```java

def cache(): this.type = persist()

```

从源码中我们可以看出，cache就是persist在无参情况下的一个别名，在使用时就直接使用 rdd.cache() 即可将rdd缓存，使用cache方法缓存rdd时，rdd的数据仅缓存在内存中

#### 使用persist方法进行缓存

persist方法的使用也是直接使用RDD即可调用，该方法的定义源码如下：

```java

def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)

def persist(newLevel: StorageLevel): this.type = { ... }

```

#### 使用unpersist释放缓存

直接调用rdd的unpersist方法进行释放缓存

从源码可以看出persist方法有两个，其中一个是无参的，另一个是有参数的，无参的persist方法是直接调用的有参数的persist方法。有参数的persist方法的参数是缓存的级别，即在进行rdd缓存时用户可以指定缓存的级别。p就persist方法而言使用比较简单，就不再赘述，接下来我们来了解一下rdd的缓存级别

### 3.缓存的多种级别

在源码中默认定义了12种缓存级别，缓存级别的定义是通过 StorageLevel 这个类来设置的，关于缓存级别的部分源码如下：

```java


class StorageLevel private(
    //使用硬盘
    private var _useDisk: Boolean,
    //使用内存
    private var _useMemory: Boolean,
    /*使用堆外内存，堆外内存表示把内存对象分配在Java虚拟机的堆以外的内存，
    	堆外内存直接受操作系统管理（而不是虚拟机），可以减少JVM垃圾回收对应用的影响*/
    private var _useOffHeap: Boolean,
    /*使用反序列化（不序列化），反序列化就表示将字节恢复为对象的过程，
    	该项为true时，即代表数据不进行序列化，该项为false时，即代表数据序列化后缓存*/
    private var _deserialized: Boolean,
    //缓存副本数，默认为1
    private var _replication: Int = 1)


//不使用缓存
val NONE = new StorageLevel(false, false, false, false)
//仅使用硬盘
val DISK_ONLY = new StorageLevel(true, false, false, false)
//仅使用硬盘，且副本数是2
val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
//仅使用内存
val MEMORY_ONLY = new StorageLevel(false, true, false, true)
//仅使用内存，且副本数是2，
val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
//仅使用内存，且使用序列化
val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
//仅使用内存，且使用序列化，且副本数为2
val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
//同时使用硬盘和内存
val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
//同时使用硬盘和内存，且副本数为2
val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
//同时使用硬盘和内存，且使用序列化
val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
//同时使用硬盘和内存，且使用序列化，且副本数为2
val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
//使用硬盘和内存的时候，也是堆外内存
val OFF_HEAP = new StorageLevel(true, true, true, false, 1)

```

### 4.缓存使用场景选择

rdd的缓存会带来好处，但是带来好处的同时也有其缺点，比如缓存rdd时会消耗计算的内存空间，在对RDD缓存时会消耗一定的计算资源，消耗资源的多少取决于rdd的缓存级别，将RDD缓存在内存中，仅会占用内存空间，但是耗时较少，将rdd缓存在硬盘中则不占用内存空间，但是耗时较多。对于缓存级别的选择，核心问题是在内存使用率和CPU效率之间进行的权衡，所以在不同场景时，我们需要进行缓存级别选择。

如果您的 RDD 适合于默认存储级别（MEMORY\_ONLY）。这是 CPU 效率最高的选项，允许 RDD 上的操作尽可能快地运行.

在使用 MEMORY\_ONLY\_SER 时，你可以指定一个快速序列化的类库，以使缓存的数据对象更加节省空间，并仍然能够快速访问

如果你的数据价值并不高，那就不建议溢写到磁盘，因为读取和写入磁盘都将花费比内存更久时间的代价。

如果你需要快速故障恢复，则可以使用多副本的存储级别，在部分计算节点故障时，复制的数据可让您继续在 RDD 上运行任务，而无需等待重新计算一个丢失的分区.

## 三、Checkpoint

### 1.Checkpoint有什么作用

Checkpoint 是一个用检查点, 它是用来容错的，它会将RDD的数据存储在可靠的存储引擎中, 这个可靠的存储引擎可以是分布式存储系统 HDFS，也可以是本地文件系统，当错误发生的时候，可以进行迅速的恢复。通常情况下Checkpoint使用的是分布式文件系统。

在上面一部分我们学习了RDD的缓存部分，缓存是将RDD的数据缓存内存或者磁盘中，而Checkpoint也是将RDD的数据存储起来，那么我们就会迷惑，这两者有什么区别？

首先我们要明确RDD的缓存的主要目的是为了提高计算效率避免重复计算而设计的，在使用RDD缓存数据我们可以将数据缓存在内存或者硬盘中，并且这些数据是有每个节点的BlockManager来管理，但是对于分布式计算来说不论是内存和单个节点的硬盘它们都不是可靠的存储，所以当部分计算节点故障时，缓存的数据不保证一定完全可用，此时RDD丢失的分区就需要沿着RDD的依赖链重新计算，而Checkpoint与缓存不同的是，Checkpoint是将数据存储在可靠的分布式文件系统（例如：hdfs）中，在RDD进行Checkpoint成功后，其依赖链也会被斩断，即该RDD的数据生成不依赖其父RDD，它数据来源就变为了读取文件系统Checkpoint存储的数据，当部分节点故障时就算内存中的分区数据丢失，也不必重新计算Checkpoint点之前的RDD数据，而是直接从Checkpoint中读取数据到RDD中，从这里看出相对于缓存来说Checkpoint主要的目的是进行容错，将RDD数据存储在高容错的分布式文件系统中，以保证出现故障时可以快速直接的从Checkpoint恢复数据，这也是Checkpoint与缓存的最大的区别。

从功能上来讲，缓存和Checkpoint都具有避免重复计算和容错的作用，但是两者的重心是不同的，缓存更注重避免重复计算提高速度，它是将数据存储在各自的计算节点，以提高计算效率，但是在计算节点故障时，其缓存的数据就会丢失，还是需要进行重新计算依赖链中的RDD，虽然缓存也可以设置多副本，但是相对于HDFS来说，这仍然是不可靠的。而Checkpoint则更注重容错，使用Checkpoint的代价就是会花费比使用缓存更多的时间将数据写入HDFS中，但是在可靠性上，即使大部分计算节点缓存数据丢失，RDD也不需要重新计算整个依赖链上RDD，而只需从Checkpoint中将数据直接加载到对于RDD中，也避免了重复计算，相对于缓存来说Checkpoint需要读写hdfs，所以读写数据的速度不及直接读写内存和本地磁盘，但是这种方式比较可靠，而且在大数据量情况下重新计算整个依赖链RDD所花费的代价远高于直接从Checkpoint加载数据到RDD

### 2.如何使用Checkpoint

Checkpoint的使用也非常简单主要是两步

* 设置checkpoint的存储目录

```java

//设置一个存储的位置，通常情况下使用hdfs的存储
sparkContext.setCheckpointDir("hdfs:///...")

```

* 调用rdd.checkpoint方法存储数据

```java

//在rdd进行checkpoint之前可以先使用缓存，这样在checkpoint时即可以从内存中将数据写入hdfs中
rdd.cache
/*checkpoint这个算子类似一个转换算子，调用该方法时，不会立马进行数据存储，
而是等到该rdd或者该rdd的衍生rdd进行action操作时才会进行存储*/
rdd.checkpoint

```

## 总结

在本篇内容中，我们介绍了RDD的分区、缓存以及Checkpoint。通过这几个知识点的学习，我们会对RDD有更深的理解，同时我们在编程时也会更加注重如何去提升程序的效率，而并非仅仅实现功能即可。特别是在大数据的计算中，由于数据集庞大，所以在完成功能的情况下，提升计算效率是我们需要重点考虑的，希望本篇内容对大家有所帮助，我们后面会继续学习spark 相关内容

看完这篇文章，希望可以帮助大家对于RDD有一个初步的认识，可能现在大家对于RDD的原理还不是很懂，不过没关系，随着学习深入才会慢慢理解。

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
