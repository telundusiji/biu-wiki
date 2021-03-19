

## 一、RDD的分区

### 1.什么是RDD分区

在RDD中分区是一个逻辑概念。理解RDD的分区时，你可以将RDD看成一个存储数据的数据集，那么这个数据集中存储着大量的数据，这些数据在RDD中又被分开在多个地方存放，那么每一个存放部分数据的位置也就可以认为是RDD的分区。仅从数据集的角度来理解，一个RDD中的所有数据就是由多个分区来保存，每个分区中仅包含一个RDD中的部分数据，一个RDD中的所有分区共同保存了整个数据集数据。

> 提示：在理解RDD分区时你可以认为RDD就是一个单纯的数据集合，但是千万不要忘记了RDD不是一个单纯的数据集，它更是一个分布式的编程模型

我们拿HDFS文件来类比一下，在HDFS上一个文件可以有多个块组成，每个块保存一个文件的部分数据，一个文件下的所有块就保存着整个文件的数据，那么类比RDD与分区的关系就与HDFS的文件和块的关系类似。

### 2.RDD分区的作用

RDD的分区设计主要是用来支持分布式并行处理数据的。我们在之前一篇文章提到过并行计算的一个要素：“解决的问题必须可以被分解为多个可以并发计算的问题”，那么RDD作为一个支持分布式并行运算的数据集，不同的分区可以被分发到不同的计算节点上并行执行，这也是体现了并行计算中问题可分解的要素。

RDD在使用分区来并行处理数据时, 是要做到尽量少的在不同的 Executor 之间使用网络交换数据, 所以当使用 RDD 计算时，会尽可能地把计算分配到在物理上靠近数据的位置。由于分区是逻辑概念，你可以理解为，当分区划分完成后，在计算调度时会根据不同分区所对应的数据的实际物理位置，会将相应的计算任务调度到离实际数据存储位置尽量近的计算节点


### 3.什么是RDD的Shuffle

Shuffle是一种让数据重新分布使得某些数据被放在同一分区里的一种机制，通俗的讲就是对数据进行重组，只有 Key-Value 型的 RDD 才会有 Shuffle 操作

### 4.Shuffle的原理是什么


### 5.RDD的分区操作

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

### 5.RDD的分区和shuffle有什么联系

## 二、RDD缓存

### 1.缓存的意义

### 2.缓存的使用

### 3.缓存的多种级别

## 三、Checkpoint

### 1.Checkpoint有什么作用

### 2.如何使用Checkpoint

