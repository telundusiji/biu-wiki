> 操作系统：CentOS-7.8   
> Spark版本：2.4.4
> scala版本：2.11.12

本篇文章锤子和大家一起学习Spark RDD的常用Action算子，锤子会对每个算子含义和入参进行说明，并附上演示代码，帮助大家快速理解和使用这些常用算子(由于Spark的RDD算子还是比较多的，本篇文章主要列出的是一些常用的，后续如果学习更多了再继续补充)，完整示例代码的GitHub地址：[https://github.com/telundusiji/dream-hammer/tree/master/module-8](https://github.com/telundusiji/dream-hammer/tree/master/module-8)

关于Spark学习的其他知识可以参考

* [《一起学习Spark——Transformation算子》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247484005&idx=1&sn=94c9140ffae93a1e2f0c6565afc6066f&chksm=e8d7add2dfa024c41ff18bda185e4b34fad64530c6670c457f54dcca1f8506c6a11bd84a1523&token=482962526&lang=zh_CN#rd)

* [《一起学习Spark——RDD入门》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247484005&idx=2&sn=d193777d61f23270b4d5e455e21b78eb&chksm=e8d7add2dfa024c43b15e746198e1d993f0f862c6af6a98f94d5231669bfc75dcfb8f8042706&token=482962526&lang=zh_CN#rd)

* [《一起学习Spark入门》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247484005&idx=3&sn=a7c44163131ed836d836951710291a1b&chksm=e8d7add2dfa024c42fe3d10fa4eb429da2cf8f7a0afce1085745c49734d43b68efd41c363c62&token=482962526&lang=zh_CN#rd)

### reduce

说明

> 对整个结果集进行归约，最终生成一条数据，是整个数据集的汇总

方法

```java

def reduce(f: (T, T) => T): T

```

参数

* f -> 归约的处理函数，函数两个入参，第一个是归约的局部结果，第二个是当前的一条数据

其他

* 注意区分reduce和reduceByKey，reduce是一个action操作，不属于shuffle，reduceByKey是transformation操作中的shuffle操作

* reduce是在每个分区上先进行归约，然后再将各个分区的归约结果汇总成一条数据

示例

```java

def reduceTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 1, 1, 1, 1))
	
	sourceRdd.collect()
	
	val result = sourceRdd
	    .reduce((agg, curr) => agg * 2 + curr)
	println(result)
	sc.stop()
}

//运行结果
/*
31
*/

```

### collect

说明

> 以数组的形式获取结果集RDD中的所有数据

方法

```java

def collect(): Array[T]

```

其他

* 由于collect会从远程集群拉取数据到driver端，所以当数据量大时使用collect会将大量数据汇集到一个driver节点上，容易造成内存溢出

示例

```java

def collectTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 2, 3))
	sourceRdd.collect()
	val result: Array[Int] = sourceRdd.collect()
	result.foreach(println(_))
	sc.stop()
}

//运行结果
/*
1
2
3
*/

```

### foreach

说明

> 遍历结果集RDD中每一个元素

方法

```java

def foreach(f: T => Unit): Unit

```

示例

```java

def foreachTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 2, 3))
	sourceRdd.collect()
	sourceRdd.foreach(println(_))
	sc.stop()
}

//运行结果
/*
1
2
3
*/

```

### count

说明

> 求结果集的数量

示例

```java

def countTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 2, 3))
	val result = sourceRdd.count()
	println(result)
	sc.stop()
}

//运行结果
/*
3
*/

```

### countByKey

说明

> 按照key进行分组后，计算每个分组中的元素数量

示例

```java

def countByKeyTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(("a", "value"), ("b", "value"), ("b", "value"), ("c", "value"), ("a", "value")))
	val result: collection.Map[String, Long] = sourceRdd.countByKey()
	result.foreach(println(_))
	sc.stop()
}

//运行结果
/*
(a,2)
(b,2)
(c,1)
*/

```

### first

说明

> 获取RDD中的第一个元素

示例

```java

def firstTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 2, 3))
	val result = sourceRdd.first()
	println(result)
	sc.stop()
}

//运行结果
/*
1
*/

```

### take

说明

> 获取RDD中的前n个元素

其他

* 当你只需要获取第一个元素时，建议使用first，因为take中会从多个分区中收集数据，相对first来说效率比较低

示例

```java

def takeTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 3, 2, 1, 5))
	val result = sourceRdd.take(3)
	result.foreach(println(_))
	sc.stop()
}

//运行结果
/*
1
3
2
*/

```

### takeSample

说明

> 采样一个RDD中的数据，与转换算子中的sample操作类似，不同的是sample操作是action操作

方法

```java

def takeSample(
      withReplacement: Boolean,
      num: Int,
      seed: Long = Utils.random.nextLong): Array[T]

```

参数

* withReplacement：采样数据是否有放回，true代表又放回采样，false代表无放回采样

* num：采样个数

* seed：随机种子

示例

```java

def takeSampleTest(): Unit = {
	val sourceRdd = sc.parallelize(Seq(1, 3, 2, 1, 5))
	val result = sourceRdd.takeSample(false, 3, 100)
	result.foreach(println(_))
	sc.stop()
}

//运行结果
/*
2
3
1
*/

```

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
