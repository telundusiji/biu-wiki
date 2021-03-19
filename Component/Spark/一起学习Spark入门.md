> 操作系统：CentOS-7.8   
> Spark版本：2.4.4

本篇文章是一个Spark入门文章，在文章中首先会对Spark进行简单概述，帮助大家先认识Spark，然后会介绍Spark安装部署上的基础知识，随后我们再演示几个简单案例帮助大家入门Spark，整篇文章所介绍的都是入门知识，更加适合没有接触过Spark刚开始学习时参考，希望对大家有帮助，[Spark官网：http://spark.apache.org](http://spark.apache.org)，下面开始正文

## 一、Spark概述

### 什么是Spark

Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎。

Spark是一个快速的，多用途的集群计算系统，相对于Hadoop来说Spark只是一个计算框架，它不包含Hadoop的分布式文件系统HDFS和完备的调度系统YARN，它与Hadoop的MapReduce对比在计算性能和效率上有大幅度提升，而且Spark为数据计算提供了丰富的api封装，可以减少编程的复杂度提高编程效率。Hadoop的MapReduce在计算过程中是将中间结果存储在文件系统中，而Spark在计算的过程中是将中间结果存储在内存中，可以将数据在不写入硬盘的情况下在内存中进行计算，这也是Spark运行速度效率远超Hadoop MapReduce的主要原因。

### Spark有什么特点

Spark是一个分布式计算引擎，它支持多种语言的API，其计算节点可以拓展到成千上万个，它能够在内存中进行数据集缓存实现交互式数据分析，同时它也提供了shell命令行交互窗口，为探索式数据分析节省时间。其具体特点，我们从以下四个方面简单列出

#### 快速

* Spark在内存时的计算速度大概是Hadoop的100倍

* Spark基于硬盘的计算速度大概是Hadoop的10倍

* Spark的DAG执行引擎，其数据缓存在内存中可以进行迭代处理

#### 易用

* Spark支持Java、Scala、Python、R、SQL等多种语言API

* Spark支持多种高级运算API使得用户可以轻松开发、构建和运行计算程序

* Spark可以使用基于Scala、Python、R、SQL的shell交互式查询

#### 通用

* Spark提供了一个完整的技术栈，包括SQL执行，Dataset命令式API，机器学习库MLlib，图计算框架GraphX，流计算SparkStreaming

* 用户可以在同一个应用中同时使用多种Spark的技术

#### 兼容

* Spark可以运行在Hadoop Yarn，Apache Mesos，Kubernets，Spark Standalone等集群之中

* Spark可以访问HBase、HDFS、Hive、Cassandra等多种数据源

### Spark主要组成

Spark提供了批处理，结构化查询，流计算，图计算，机器学习等组件，这些组件都依托于通用的计算引擎RDD，它们之间的关系结构如图所示

![spark](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Spark%E5%85%A5%E9%97%A8/Spark.png)


#### Spark Core和弹性分布式数据集(RDD)

* Spark Core是整个Spark的基础，提供了分布式任务调度和基本的I/O功能

* Spark的基础的程序抽象是弹性分布式数据集RDD，它是一个可以并行操作，支持容错的数据集合

* RDD可以通过引用外部存储系统的数据集创建创建，也可以通过现有的RDD转换得到

* RDD抽象提供了Java、Scala、Python等语言的API

* RDD简化了编程的复杂性，操作RDD类似通过Scala或者Java8的Streaming操作本地数据集合

#### Spark SQL

* Spark SQL在Spark Core的基础上提出了DataSet和DataFrame的数据抽象化概念

* Spark SQL提供了在DataSet和DataFrame之上执行SQL的能力

* Spark SQL提供了DSL，可以通过Scala、Java、Python等语言操作DataSet和DataFrame

* 支持使用JDBC和ODBC操作SQL语言

#### Spark Streaming

* Spark Streaming充分利用Spark Core的快速开发能力来运行流式分析

* Spark Streaming截取小批量的数据并可以对之运行RDD Transformation

* Spark Streaming提供了在同一个程序中同时使用流分析和批量处理分析的能力

#### GraphX

* GraphX是分布式图计算框架，提供了一组可以表达图计算的API，GraphX还对这种抽象化提供了优化运行

### 二、Spark集群

### Spark的运行方式

先来了解几个名词：

* 集群：一组协同工作的计算机，分工合作对外表现的向一台计算机一样，所有运行的任务有软件来控制和调度

* 集群管理工具：调度任务到集群的软件

* 常见的集群管理工具有：Hadoop YARN，Apache Mesos，Kubernetes

Spark 的运行模式大方向分为两种：

* 单机：使用线程模拟并行运行的程序

* 集群，使用集群管理器来和不同类型的集群交互，将任务运行在集群中

Spark Standalone模式下几个名词概念

* Master：负责总控，调度，管理和协调Worker，保留资源状况等

* Worker：用于启动Executor执行Tasks，定期向Master汇报

* Driver：运行在Client或者Worker中，负责创建基础环境和任务分配

Spark在集群模式下的支持的管理方式分两个方向：

* Spark自带集群管理器:Spark Standalone

> 在Standalone集群中，分为两个角色Master和Worker，在Standalone集群启动的时候会创建固定数量的Worker。  
> Driver的启动分为两种模式：Client和Cluster，在Client模式下，Driver运行在Client端，在Client启动的时候被启动，在Cluster模式下，Driver运行在某个Worker中，随着应用的提交而启动

* 第三方集群管理器：Hadoop YARN，Apache Mesos，Kubernetes

> 在Yarn集群模式下运行Spark程序，首先会和RM交互，开启ApplicationMaster，其中运行了Driver，Driver创建基础环境后，会由RM提供对应的容器，运行Executor，Executor会再向Driver注册自己，并申请Task执行


### Spark Standalone模式部署

在使用第三方集群管理器时Spark不需要进行部署操作，所有本篇中只介绍Standalone模式部署

部署教程参考：[《一起学习Spark集群搭建》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483987&idx=1&sn=fb8d2cd247f55413e1f4f6e24712ad2b&chksm=e8d7ade4dfa024f215925572e62b9b43f623d409604bea2097735eec4aa0f208108c7612bbd9&token=312074909&lang=zh_CN#rd)


## 三、入门操作

### 自带Spark示例运行

安装完成Spark后，我们运行一下Spark自带的一个演示程序：蒙特卡洛算法求圆周率

```

#进入Spark安装目录
cd {SPARK_HOME}

./bin/spark-submit \
#指定运行的主类
--class org.apache.spark.examples.SparkPi \
#指定master地址
--master spark://spark13:7077,spark12:7077,spark11:7077 \ 
#指定executor的内存
--executor-memory 1G \
#指定executor使用的核心数
--total-executor-cores 2 \
#指定jar包的位置
/app/spark/spark-2.4.4-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.4.jar \
#计算Pi所需要的的一个入参
100

```

### spark-shell使用

Spark-Shell 是Spark提供的一个基于Scala语言的交互式解释器，Spark-Shell可以直接在Shell中编写代码执行，这是一种快速使用Spark的方式，由于可以采用交互式命令行进行编程，所以适用快速数据探索和分析，下面演示一个Spark-Shell的使用示例

启动spark-shell：`{SPARK_HOME}/bin/spark-shell --master spark://spark13:7077`

```

#读取hdfs上文件
val rdd1 = sc.textFile("hdfs://spark11:8020/data.txt")
#将每行按照空格分隔出多个单词
val rdd2 = rdd1.flatMap(line=>line.split(" "))
#将每个单词给词频为1
val rdd3 = rdd2.map(word=>(word,1))
#按照Key进行聚合操作
val rdd4 = rdd3.reduceByKey((curr,agg)=>curr+agg)
#从rdd4中取出数据
rdd4.collect

```



### spark独立应用编写


引入pom依赖

```

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
</dependency>

<build>
	<plugins>
	    <plugin>
	        <groupId>net.alchim31.maven</groupId>
	        <artifactId>scala-maven-plugin</artifactId>
	        <version>3.2.2</version>
	        <executions>
	            <execution>
	                <id>compile-scala</id>
	                <phase>generate-sources</phase>
	                <goals>
	                    <goal>add-source</goal>
	                    <goal>compile</goal>
	                </goals>
	            </execution>
	        </executions>
	        <configuration>
	            <scalaVersion>2.11.12</scalaVersion>
	            <args>
	                <arg>-target:jvm-1.8</arg>
	            </args>
	            <jvmArgs>
	                <jvmArg>-Xss8196k</jvmArg>
	            </jvmArgs>
	        </configuration>
	    </plugin>
	</plugins>
</build>

```

示例代码读取hdfs上文件内容进行WordCount

```java

//local模式直接在本地即可运行
object SparkTest {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf()
      //设置spark应用名称
      .setAppName("SparkTest")
      //设置Spark运行方式为local方式，在集群模式运行时，将这一行去掉
      .setMaster("local[*]")
    val sparkContext = new SparkContext(sparkConf)

    //读取hdfs上文件
    val sourceFile = sparkContext.textFile("hdfs://spark11:8020/data.txt")
    //对读取的内容进行按空格分隔后在统计词频
    val resultRdd = sourceFile
      .flatMap(line => line.split(" "))
      .map(word => (word, 1))
      .reduceByKey((curr, agg) => curr + agg)
    //从rdd中获取结果并打印
    val result = resultRdd.collect()
    result.foreach(r=>println(r))
    sparkContext.stop()
  }

}

//在集群运行时，上面代码去掉   .setMaster("local[*]")这一行，然后打成jar包
//spark-submit进行提交到集群运行
./bin/spark-submit --class site.teamo.learning.spark.SparkTest --master spark://spark11:7077,spark12:7077,spark13:7077 /app/spark/spark-test-1.0-SNAPSHOT.jar

```

### 总结

经过以上学习，帮助大家对Spark有了基本认识，也学习了Spark的部署安装，最后我们有演示了几个入门案例，帮助大家入门Spark后续，我们会从基础开始学习和使用spark，然后再到理解器原理等知识，欢迎关注后续的文章

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
