> 操作系统：CentOS-7.8   
> Spark版本：2.4.4
> scala版本：2.11.12

本篇文章锤子和大家一起学习Spark RDD的常用Transformation算子，在文章中把转换算子分为了六大类：转换操作、过滤操作、集合操作、排序操作、聚合操作、分区操作，锤子会对每个算子含义和入参进行说明，并附上演示代码，帮助大家快速理解和使用这些常用算子(由于Spark的RDD算子还是比较多的，本篇文章主要列出的是一些常用的，后续如果学习更多了再继续补充)，完整示例代码的GitHub地址：[https://github.com/telundusiji/dream-hammer/tree/master/module-8](https://github.com/telundusiji/dream-hammer/tree/master/module-8)

## 转换操作

### map

说明

> 把RDD中的数据一对一的转换成另一种形式

方法

```java

def map[U: ClassTag](f: T => U): RDD[U]

```

参数

* f -> 原RDD转向新RDD的过程，传入函数的参数是原RDD数据，函数返回值是经过转换后的新RDD数据

示例

```java

//示例中是将String 进行分割成 String[]  
//所以转换后的RDD的每一条数据就是一个数组，在最后我们打印了数组的长度
def mapTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      "Hadoop HBase FLink",
      "Hello world",
      "你好 Spark"
    ))
    sourceRdd
      .map(item => item.split(" "))
      .collect()
      .foreach(item => println(item.length))

    sc.stop()
}

//运行结果为
/*
3
2
2
*/

````

### flatMap

说明

> flatMap和map类似，但是flatMap是把RDD中的数据一对多的转换成另一种形式

方法

```java

def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]

```

参数

* f -> 原RDD数据，返回值经过转换后的新RDD数据，传入函数的入参是原RDD数据，传入函数的出参是一个集合，集合被展平后的数据才是新的RDD数据

示例

```java

//和map示例的初始数据相同，使用flatMap后每一条数据被分割后的字符串数字，会被展平，产生多条数据
def flatMapTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      "Hadoop HBase FLink",
      "Hello world",
      "你好 Spark"
    ))
    sourceRdd
      .flatMap(item => item.split(" "))
      .collect()
      .foreach(item => println(item))

    sc.stop()
}
  
//运行结果
/*
Hadoop
HBase
FLink
Hello
world
你好
Spark
*/

```

### mapPartitions

说明

> 针对整个分区的数据进行转换。

与map区别

* map针对每一条数据进行处理，粒度是一条数据 item=>item；mapPartitions每次一个分区的数据，粒度是分区，iter=>iter

方法

```java

  def mapPartitions[U: ClassTag](
      f: Iterator[T] => Iterator[U],
      preservesPartitioning: Boolean = false): RDD[U]

```

参数

* f -> 原RDD数据，返回值经过转换后的新RDD数据，传入函数的入参是原RDD的分区的迭代器(集合)，传入函数的出参是转换后的数据的迭代器（集合）

* preservesPartitioning：是否保留父RDD的分区信息，默认false

示例

```java

/**
* mapPartitions对一整个分区数据进行转换
* 与map区别
* 1.map的func参数是单条数据，mapPartitions的func的参数是一个分区的数据，及一个集合
* 2.map的func返回值是单条数据，mapPartitions的func的返回值是一个集合
*/
//每次处理一个分区数据，每个分区是一个迭代器，使用迭代器就可以对每条数据转换，最后再返回迭代器
def mapPartitionsTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4), 3)
	sourceRdd
	    .mapPartitions(partition => {
	    partition.map(item => item * 10)
	    })
	    .collect()
	    .foreach(item => println(item))
	
	sc.stop()
}
//运行结果
/*
10
20
30
40
*/

```

### mapPartitionsWithIndex

说明

> 针对整个分区的数据进行转换，并且再传入分区数据时会附带分区的索引编号

方法

```java

def mapPartitionsWithIndex[U: ClassTag](
      f: (Int, Iterator[T]) => Iterator[U],
      preservesPartitioning: Boolean = false): RDD[U]

```

参数

* f -> 原RDD数据，返回值经过转换后的新RDD数据，传入函数的入参是原RDD的分区的索引以及分区数据迭代器(集合)，传入函数的出参是转换后的数据的迭代器（集合）

* preservesPartitioning：是否保留父RDD的分区信息，默认false

示例

```java

/**
* mapPartitionsWithIndex和mapPartitions的区别就是再mapPartitionsWithIndex中func的入参多个分区编号
*/
//与mapPartitions演示案例不同的就是，再转换每条数据时，将其与分区索引编号相乘
def mapPartitionsWithIndexTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4), 3)
    sourceRdd
      .mapPartitionsWithIndex((index, partition) => {
        partition.map(item => index * item)
      })
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
0
2
6
8
*/

```

### mapValues

作用

> 只能使用在RDD内数据为Key-Value类型的RDD，使用函数对数据进行转换，与map不同的是，mapValues只转换Key-Value类型数据的value

方法

```java

def mapValues[U](f: V => U): RDD[(K, U)]

```

参数

* f -> 原RDD转向新RDD的过程，传入函数的参数是原RDD每条k-v数据的value，函数返回值是经过转换后的新RDD数据的每条k-v数据的value

演示

```java

/**
* mapValues与map相似，不同的是map作用于整条数据，mapValues作用于每条Key-Value类型数据的value
*/
//对源rdd的key-value类型数据的value进行转换
def mapValuesTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(("a", 1), ("b", 2), ("c", 3), ("d", 4)), 3)
    sourceRdd
      .mapValues(itemValue => itemValue * 10)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(a,10)
(b,20)
(c,30)
(d,40)
*/

```

### join

说明

> 将两个RDD按照相同的Key进行连接

方法

```java

def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]

```

参数

* other：进行join操作的另一个RDD

示例

```java

//将rdd1与rdd2按照key进行join
def joinTest(): Unit = {
    val rdd1 = sc.parallelize(Seq(("a", 1), ("a", 2), ("b", 3), ("b", 4)))
    val rdd2 = sc.parallelize(Seq(("a", 5), ("a", 6)))
    rdd1
      .join(rdd2)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(a,(1,5))
(a,(1,6))
(a,(2,5))
(a,(2,6))
*/

```

## 过滤操作

### filter

说明

> filter算子是过滤掉RDD中不需要的内容

方法

```java

def filter(f: T => Boolean): RDD[T]

```

参数

* f -> 过滤RDD中的数据，传入函数的入参是原RDD中的一条数据，传入函数返回值是一个Boolean类型的值，ture代表该条数据保留，false代表该条数据去除

示例

```java

/**
* filter过滤掉数据集中不符合要求的元素
* filter的func的入参是RDD的一个元素，返回值是boolean类型值，true代表保留该数据，false代表去除该数据
*/
//过滤数据，把值大于2的数据保留
def filterTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4, 5), 3)
    sourceRdd
      .filter(item => item > 2)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
3
4
5
*/

```

### sample

说明

> sample是从一个数据集中抽样出一部分，常被用来在尽可能减少数据规律损失的情况下，减少数据集以保证运行速度

方法

```java

 def sample(
      withReplacement: Boolean,
      fraction: Double,
      seed: Long = Utils.random.nextLong): RDD[T]

```

参数

* withReplacement：抽样出来的数据是否有放回，ture代表有放回，false代表不放回

* fraction：抽样比列

* seed：随机数种子

示例

```java

//对源数据进行两种情况抽样演示，
//第一种无放回抽样，则抽样结果不会出现重复
//第二种有放回抽象，则抽样结果可能出现重复
def sampleTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4, 5), 3)
    //无放回
    System.out.println("抽样无放回演示")
    sourceRdd
      .sample(false, 0.8, 100)
      .collect()
      .foreach(item => println(item))
    //有放回
    System.out.println("抽样有放回演示")
    sourceRdd
      .sample(true, 0.8, 100)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
抽样无放回演示
1
3
4
5
抽样有放回演示
1
1
2
5
*/

```

## 集合操作

### intersection

说明

> 对两个rdd中元素进行交集操作

方法

```java

//交集总共有三个重载方法，如下

def intersection(other: RDD[T]): RDD[T]

def intersection(
    other: RDD[T],
    partitioner: Partitioner)(implicit ord: Ordering[T] = null): RDD[T]
    
def intersection(other: RDD[T], numPartitions: Int): RDD[T]

```

参数

* other：进行交集的另一个RDD

* partitioner：交集操作后生成RDD的分区器（分区方式）

* numPartitions：交集操作后生成RDD的分区数

演示

```java

def intersectionTest(): Unit = {
    val rdd1 = sc.parallelize(Seq(1, 2, 3, 4, 5))
    val rdd2 = sc.parallelize(Seq(4, 5, 6, 7, 8))
    rdd1
      .intersection(rdd2)
      .collect()
      .foreach(item => println(item))

    val rdd3 = sc.parallelize(Seq((1, "a"), (2, "b"), (3, "c")))
    val rdd4 = sc.parallelize(Seq((2, "b"), (3, "d"), (4, "d")))

    rdd3
      .intersection(rdd4)
      .collect()
      .foreach(println(_))

    sc.stop()
}

//运行结果
/*
4
5
(2,b)
*/

```

### union

说明

> 对两个RDD中的元素进行并集操作

方法

```java

def union(other: RDD[T]): RDD[T]

```

参数

* other：进行交集的另一个RDD

演示

```java

def unionTest(): Unit = {
    val rdd1 = sc.parallelize(Seq(1, 2, 3))
    val rdd2 = sc.parallelize(Seq(2, 3, 4))
    rdd1
      .union(rdd2)
      .collect()
      .foreach(item => println(item))
    
    sc.stop()
}

//运行结果
/*
1
2
3
2
3
4
*/

```

### subtract

说明

> 调用该操作的RDD与入参中传入RDD的差集

方法

```java

//差集的三个重载方法

def subtract(other: RDD[T]): RDD[T]

def subtract(other: RDD[T], numPartitions: Int): RDD[T]

def subtract(
    other: RDD[T],
    p: Partitioner)(implicit ord: Ordering[T] = null): RDD[T]

```

参数

* other：对其进行差集操作的另一个RDD

* numPartitions：差集操作后生成新的RDD的分区数

* partitioner：差集操作后生成新的RDD的分区方式

演示

```java

//因为差集是xx对xx的差，所以是分方向，即A对B的差集和B对A的差集结果是不一样的
def subtractTest(): Unit = {
    val rdd1 = sc.parallelize(Seq(1, 2, 3, 4, 5))
    val rdd2 = sc.parallelize(Seq(4, 5, 6, 7, 8))
    System.out.println("rdd1.subtract(rdd2)")
    rdd1
      .subtract(rdd2)
      .collect()
      .foreach(item => println(item))

    System.out.println("rdd1.subtract(rdd2)")
    rdd2
      .subtract(rdd1)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
rdd1.subtract(rdd2)
1
2
3
rdd2.subtract(rdd1)
6
7
8
*/

```

## 排序操作

### sortByKey

说明

> 对RDD内元素为Key-Value类型数据时，按照key排序

方法

```java

def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length): RDD[(K, V)]

```

参数

* ascending：是否升序，默认是true代表升序，设置为false时代表降序

* numPartitions：排序后RDD的分区数

演示

```java

def sortByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq((2, "a"), (1, "b"), (3, "c")))

    sourceRdd
      .sortByKey()
      .collect()
      .foreach(println(_))

    sc.stop()
}

//运行结果
/*
(1,b)
(2,a)
(3,c)
*/

```

### sortBy

说明

>根据指定的元素的属性进行排序

方法

```java

def sortBy[K](
      f: (T) => K,
      ascending: Boolean = true,
      numPartitions: Int = this.partitions.length)
      (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]

```

参数

* f -> 该函数返回要排序的字段

* ascending：是否升序，默认是true代表升序，设置为false时代表降序

* numPartitions：排序后RDD的分区数

演示

```java

def sortByTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq((2, "a"), (1, "b"), (3, "c")))

    sourceRdd
      .sortBy(item => item._2,false)
      .collect()
      .foreach(println(_))

    sc.stop()
}

//运行结果
/*
(3,c)
(1,b)
(2,a)
*/

```

## 聚合操作

### reduceByKey

说明

> 先按照Key进行分组生成一个Tuple，然后针对每个分组再进执行reduce算子

方法

```

def reduceByKey(func: (V, V) => V): RDD[(K, V)]

```

参数

* func -> 执行数据处理的函数，传入的函数有两个入参，第一个入参是局部聚合值，第二个入参是当前值，函数有一个输出为汇总结果

示例

```java

//一个年级和名称的集合，按照key分组后，进行汇总，
//即“张三、王五、赵六”属于同一个key，“李四”属于另一个Key，相同Key所对应的数据会汇总在一起
def reduceByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      ("一年级","张三"),
      ("二年级","李四"),
      ("一年级","王五"),
      ("一年级","赵六")
    ))
    sourceRdd
      .reduceByKey((agg, curr) => curr + "-" + agg)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(二年级,李四)
(一年级,张三-王五-赵六)
*/


```

### groupByKey

说明

> 对于Key-Value数据类型RDD的元素，按照Key进行分组，不对分组后的结果聚合

方法

```java

//groupByKey的三个重载方法

def groupByKey(): RDD[(K, Iterable[V])]

def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]

def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]

```

参数

* numPartitions：操作结果RDD的分区数

* partitioner：操作结果RDD的分区器

注意

* groupByKey是一个shuffled操作

* groupByKey需要列举key对应的所有数据，所以无法在Map端进行Combine操作，所以groupByKey性能没有reduceByKey性能好

示例

```java

def groupByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      ("一年级", "张三"),
      ("二年级", "李四"),
      ("一年级", "王五"),
      ("一年级", "赵六")
    ))
    sourceRdd
      .groupByKey()
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(二年级,CompactBuffer(李四))
(一年级,CompactBuffer(张三, 王五, 赵六))
*/

```

### combineByKey

说明

> 对数据集按照key来聚合，用户可以自定义聚合的初步转换，以及在每个分区进行初步聚合和在所有分区上最终聚合结果

方法

```java

def combineByKey[C](
    createCombiner: V => C,
    mergeValue: (C, V) => C,
    mergeCombiners: (C, C) => C,
    partitioner: Partitioner,
    mapSideCombine: Boolean = true,
    serializer: Serializer = null): RDD[(K, C)]

```

参数

* createCombiner：对value的值进行初步转换，函数入参为Key-value数据的value

* mergeValue：在每个分区上把上一步转换的结果进行聚合，函数入参为createCombiner的出参以及下一条数据的value

* mergeCombiners：在所有分区上把每个分区的结果再进行聚合

* partitioner：分区器

* mapSideCombine：是否在map端combine

* serializer：序列化器

其他

* groupByKey和reduceByKey的底层都是combineByKey

示例

```java

def combineByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      ("一年级", "张三"),
      ("二年级", "李四"),
      ("一年级", "王五"),
      ("一年级", "赵六")
    ))
    sourceRdd
      .combineByKey(
        createCombiner = curr => (curr, 1),
        mergeValue = (agg: (String, Int), nextValue) => (agg._1 + nextValue, agg._2 + 1),
        mergeCombiners = (agg: (String, Int), curr: (String, Int)) => (curr._1 + "|" + agg._1, curr._2 + agg._2))
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(二年级,(李四,1))
(一年级,(赵六|王五|张三,3))
*/

```

### foldByKey

说明

> 和reduceByKey类似，都是按照Key进行分组聚合，不同点是foldByKey在聚合前会对每条数据加上初始值

方法

```java

//三个重载方法

def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]

def foldByKey(
    zeroValue: V,
    partitioner: Partitioner)(func: (V, V) => V): RDD[(K, V)]

def foldByKey(zeroValue: V, numPartitions: Int)(func: (V, V) => V): RDD[(K, V)]

```

参数

* zeroValue：初始值

* func -> 和reduceByKey一致，执行数据处理的函数，传入的函数有两个入参，第一个入参是局部聚合值，第二个入参是当前值，函数有一个输出为汇总结果

* numPartitions：操作结果RDD的分区数

* partitioner：操作结果RDD的分区器


示例

```java

//在进行聚合操作前会对每一条数据都加上一个初识值
def foldByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      ("一年级", "张三"),
      ("二年级", "李四"),
      ("一年级", "王五"),
      ("一年级", "赵六")
    ))
    sourceRdd
      .foldByKey("|")((agg, curr) => curr + agg)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(二年级,李四|)
(一年级,赵六|王五|张三|)
*/

```

### aggregateByKey

说明

> 聚合所有相同的Key的value，并且聚合前可以使用初识值对每条数据进行操作

方法

```java

//同样有三个重载，另外两个跟其他算子类似多了一个分区方法入参，所以这里仅列出主要的一个

def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U,
    combOp: (U, U) => U): RDD[(K, U)]

```

参数

* zeroValue：指定初始值

* seqOp -> 转换每一个值的函数

* combOp -> 将转换的值聚合的函数

示例

```java

//先对每一条值的左右两边加上“|”，然后再聚合
def aggregateByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(
      ("一年级", "张三"),
      ("二年级", "李四"),
      ("一年级", "王五"),
      ("一年级", "赵六")
    ))
    sourceRdd
      .aggregateByKey("|")((zeroValue, item) => zeroValue + item + zeroValue, (agg, curr) => curr + agg)
      .collect()
      .foreach(item => println(item))

    sc.stop()
}

//运行结果
/*
(二年级,|李四|)
(一年级,|赵六||王五||张三|)
*/

```

## 分区操作

### repartitions

说明

> 对RDD重分区，可以把分区数调大，也可以把分区数调小

方法

```java

def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]

```

参数

* numPartitions：指定分区数

示例

```java

//调整分区并指定分区数
def repartitionByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7), 3)
    System.out.println("调大分区数:" + sourceRdd.repartition(5).getNumPartitions)
    System.out.println("调小分区数" + sourceRdd.repartition(2).getNumPartitions)
    sc.stop()
}

//运行结果
/*
调大分区数:5
调小分区数2
*/

```

### coalesce

说明

> 对分区数进行调整，默认只能调小，不能调大，当在参数中指定允许shuffle时，才可以调大

方法

```java

def coalesce(numPartitions: Int, shuffle: Boolean = false,
               partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
              (implicit ord: Ordering[T] = null)
      : RDD[T]

```

参数

* numPartitions：重分区的分区数

* shuffle：是否允许shuffle，默认false，即在重分区不允许shuffle，则不能扩大分区数

* partitionCoalescer：分区合并器，可选项，默认不指定

示例

```java

//在不允许shuffle的情况下，分区数无法扩大
def coalesceByKeyTest(): Unit = {
    val sourceRdd = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7), 3)
    System.out.println("调大分区数:" + sourceRdd.coalesce(5).getNumPartitions)
    System.out.println("调大分区数:" + sourceRdd.coalesce(5, true).getNumPartitions)
    System.out.println("调小分区数" + sourceRdd.coalesce(2).getNumPartitions)
    sc.stop()
}

//运行结果
/*
调大分区数:3
调大分区数:5
调小分区数2
*/

```

###总结

本篇文章主要是对RDD的常见转换算子进行了说明并附带了示例，帮助大家学习， 由于RDD的算子比较多，所以本篇文章没有全面覆盖，后续学习过程中锤子再进行补充

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
