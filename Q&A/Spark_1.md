# Spark_1

### 1. 简单说说 Spark 支持的4种集群管理器

Spark目前支持4中集群管理器如下：

* Standalone

> 这个是Spark自带的一种集群管理器，资源管理器是 Master 节点，调度策略相对单一，只支持先进先出模式，固定任务资源

* Hadoop Yarn

> Yarn是Hadoop提供的资源管理器，Yarn 支持动态资源的管理，还可以调度其他实现了 Yarn 调度接口的集群计算，非常适用于多个集群同时部署的场景，是目前最流行的一种资源管理系统，这个在我们工作中是常有的一种

* Apache Mesos

> Mesos 是专门用于分布式系统资源管理的开源系统，可以对集群中的资源做弹性管理。这个仅仅是知道这个组件，在工作中没有使用过

* Kubernetes

> 这个是从 Spark 2.3.0开始才引入的集群管理器，Docker 作为基本的 Runtime 方式，主要运用了容器化技术，在我之前的工作中没有使用过，自己学习过程中仅仅尝试过Kubernetes，没有在Kubernetes中跑过Spark

### 2. 为什么要用Yarn来运行Spark程序

首先Yarn 是一个支持**动态资源配置**的集群资源管理器。

而Standalone 模式只支持简单的固定资源分配策略，每个任务固定数量的 core，各 Job 按顺序依次分配在资源，资源不够的时候就排队。这种模式比较适合单用户的情况，多用户的情境下，会有可能有些用户的任务得不到资源。

相对于Standalone模式，Yarn的调度策略更加灵活，支持多队列，多用户，还支持多种不同应用，Yarn 作为通用的资源调度平台，除了 Spark 提供调度服务之外，还可以为其他系统提供调度，像 Hadoop MapReduce, Hive,Flink  等其他实现Yarn调度接口的应用都可以被Yarn调度

当然将Spark程序运行在Apache Mesos也是可以，不过之前的工作中并没有选择这种方式，我个人认为应该是Hadoop的Hdfs我们本来就要使用，那么部署Hdfs的同时直接安装一套Yarn会比单独在安装Mesos更加方便

至于使用Kubernetes运行Spark程序，这个由于目前Kubernetes对于Hadoop的容器化，还没有太好的解决方案，而且对Spark的版本还有限制，在真正使用中可能会发生预想不到的问题，所以目前没有使用

### 3. Spark提交作业流程是怎么样的

Spark提交作业的大致过程如下

* 使用Spark-submit提交代码，然后会执行new SparkContext()，然后再SparkContext中构造DAGScheduler和TaskScheduler
* TaskScheduler构造完成后，连接资源调度器的Master，向Master注册Application
* 资源调度器Master收到Application请求后，会使用相应的资源调度算法，在其管理的集群的Worker节点上为该Application启动多个Executor
* Executor启动成功后，再向TaskScheduler注册告知自己启动成功，在所有的Executor都注册到Driver端的TaskScheduler上之后，SparkContext初始化就结束了，接下来开始执行计算代码
* 在代码执行过程中，每遇到一个Action操作，就会创建一个Job提交给DAGScheduler，DagScheduler会将Job划分成多个Stage
* Stage划分后，会为每个Stage会创建一个TaskSet，TaskSet里面就是每个Stage的所需要执行的所有Task，TaskScheduler会把每个TaskSet里面的Task，都提交到Executor上去执行
* 每个Executor 本质就是一个多线程模型的执行器，Executor上的线程池每接收一个Task，就会使用TaskRunner封装，然后将其交由线程池中的线程来进行执行(TaskRunner 将我们编写的代码，拷贝，反序列化，执行 Task，每个 Task 执行 RDD 里的一个 partition）

### 4.Worker 和 Executor 的区别

Worker 是指每个工作节点，启动的一个进程，负责管理本节点，jps 可以看到 Worker 进程在运行，对应的概念是 Master 节点。 Executor 每个 Spark 程序在每个节点上启动的一个进程，专属于一个 Spark 程序，与 Spark 程序有相同的生命周期，负责 Spark 在节点上启动的 Task，管理内存和磁盘。如果一个节点上有多个 Spark 程序，那么相应就会启动多个执行器。所以说一个 Worker 节点可以有多个 Executor 进程。

### 5.Spark Local 和 Standalone 有什么区别

我们常有的Spark的运行模式有Local(主要用于本地调试)，然后就是Standalone，Yarn(Yarn 又分为Yarn-Cluster和Yarn-Client)，除了这几种常用的还有Mesos和Kubernetes。其中Local模式和Standalone模式的主要区别就是：

* Local 模式是一种单机模式，在编程和使用Spark时，若在命令语句中不加任何配置，这个时候则默认就是 Local 模式，在本地运行。这也是部署、设置最简单的一种模式，所有的 Spark 进程都运行在一台机器或一个虚拟机上面。
* 而Standalone模式就不是单机模式了（当然你也可以使用单台机器启动Standalone模式）。Standalone 是 Spark 自身实现的资源调度框架。Standalone的应用场景多在：我们只使用 Spark 进行大数据计算，不使用其他的计算框架时，这个就采用 Standalone 模式就够了，也不需要进行与其他资源管理框架对接，相对简单一些，尤其是单用户的情况下。

所以Local模式和Standalone的主要区别就是Local模式是一种单机运行模式常用于调试，Standalone模式是一种Spark 自带的集群资源管理的运行模式，这种集群资源管理调度只可以运行Spark程序，不依赖其他第三方集群资源调度管理。

#### 拓展回答

Standalone 的集群资源管理服务是一种Master—Worker架构。Master节点负责接收提交的Job以及Job调度，在Master模式下我们的Spark的应用的Driver 既可以运行在 Master 节点上中，也可以运行在本地 Client 端。当用 spark-shell 交互式工具提交 Spark 的 Job 时，Driver 在 Master 节点上运行；当我们在本地JVM中使用 `new SparkConf.setManager(“spark://master:7077”)` 方式运行 Spark 任务时，Driver 是运行在本地 Client 端上的。

提交命令类似于，在master参数中指定Spark Master节点的主机名和端口号:

```bash
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://hostname:7077 \
  /examples/jars/spark-examples_2.11-2.2.0.jar \
  100
```

### 6.RDD, DAG, Stage, Task 和 Job 如何理解

这几个概念，不严谨的来讲除了DAG外的几个，可以按照从大到小的关系排列，依次为RDD最大，然后Job，Stage，最后Task最小，而DAG主要是表示依赖关系，所以没有大小。接下来是每个概念的详细信息：

* **RDD**  全称是弹性分布式数据集，也可以说是 Spark 的灵魂。一个 RDD 代表一个可以被分区的只读数据集。RDD 内部可以有许多分区(partitions)，每个分区又拥有大量的记录(records)。
* **Job** Spark 的 Job 来源于用户对RDD执行的 action 操作（这是 Spark 中实际意义的 Job），对一个RDD执行多个action操作，也就会产生多个Job。当然我们在操作RDD时也会将一个 RDD 转换成另一个 RDD，这种操作是Transformation也就是转换操作，转换操作不会产生Job
* **DAG**  全称是有向无环图，Spark 中**使用 DAG 对 RDD 的关系进行建模**，所以DAG就是描述了 RDD 的依赖关系，从DAG中我们可以得到RDD的变换信息，当我们执行一个action操作时就会生成一个Job，而一个Job的执行通常都是一个或多个RDD经过各种转换最终得到的，所以在这个job中就需要描述RDD的依赖关系，那么执行这个Job时就会产生一个DAG，里面描述了产生这个Job操作结果的前面的RDD的依赖关系
* **Stage**  翻译过来就是阶段的意思，在每个Job的DAG 中又进行 Stage 的划分，划分的依据是依赖是否是 shuffle 的，每个 Stage 又可以划分成若干 Task。接下来的事情就是 Driver 发送 Task 到 Executor，Executor 线程池去执行这些 task，完成之后将结果返回给 Driver。
* **Task**  就是最后使用executor中线程去执行的每个最小任务了，在一个 Stage 内，最终action操作的 RDD 有多少个 partition，就会产生多少个 task。

### 7. 宽依赖和窄依赖如何理解

对于宽窄依赖可以理解为：

* 窄依赖指的是父RDD(Parent RDD)的每个Partition 最多被子 RDD 的一个 Partition 使用，**子RDD分区通常对应常数个父RDD分区**

* 宽依赖是指父RDD的每个分区都可能被多个子RDD分区所使用，**子RDD分区通常对应所有的父RDD分区**

宽依赖往往对应着shuffle操作，需要在运行过程中将父RDD的同一个分区传入到不同的子RDD分区中，中间可能涉及多个节点之间的数据传输；而窄依赖的每个父RDD的分区只会传入到一个子RDD分区中，通常可以在一个节点内完成转换。

当RDD分区丢失时（某个节点故障），spark会对数据进行重算，这个时候：

* 对于窄依赖，由于父RDD的一个分区只对应一个子RDD分区，这样只需要重算和子RDD丢失分区对应的父RDD分区即可，所以这个重算对数据的利用率是100%的
* 对于宽依赖，由于子RDD的一个分区依赖父RDD的多个分区，所以需要重算父RDD的多个分区，且重算的父RDD分区对应多个子RDD分区，这样实际上父RDD 中只有一部分的数据是被用于恢复这个丢失的子RDD分区的，另一部分对应子RDD的其它未丢失分区，这就造成了多余的计算；更一般的，宽依赖中子RDD分区通常来自多个父RDD分区，极端情况下，所有的父RDD分区都要进行重新计算

窄依赖常用算子有：map, filter, union, join(父RDD是hash-partitioned ), mapPartitions, mapValues
宽依赖常用算子有：groupByKey, join(父RDD不是hash-partitioned ), partitionBy,repartition(分区数变多)

### 8. Repartition 有什么作用

一般上来说有多少个 Partition，就有多少个 Task，Repartition 的理解其实很简单，就是把原来 RDD 的分区重新安排。当使用Repartition减少分区时，可以避免小文件，相应的task数量也会减少，若partition的数量太少，远小于spark程序的并行度时，会降级计算效率，导致任务只能被分配到部分线程，不能发挥并行计算的优势。当使用Repartition扩大分区时，可能会产生较多小文件，同时task数量也会变多，在扩大分区时会产生shuffler，在多节点场景下，对网络带宽会有较大消耗，同样若扩大的分区数若远大于spark程序的并行度，也不能起到提高效率的作用，返回会因为task过多频繁调度以及网络读取降低整个计算效率

### 9. Spark 为什么快？Spark Sql 一定比Hive快吗

 Spark 快主要是一下原因

1. Spark 师基于内存的计算，消除了冗余的 HDFS 读写: Hadoop 的MapReduce每次 shuffle 操作后，必须写到磁盘，而 Spark 在 shuffle 后不一定落盘，可以 persist 到内存中，以便迭代时使用。如果操作复杂，很多的 shufle 操作，那么 Hadoop 的读写 IO 时间会大大增加，也是 Hive 更慢的主要原因了。
2. Spark消除了冗余的 MapReduce 阶段: Hadoop 的 shuffle 操作一定连着完整的 MapReduce 操作，冗余繁琐。而 Spark 基于 RDD 提供了丰富的算子操作，且 reduce 操作产生 shuffle 数据，可以缓存在内存中。
3. JVM 的优化: Hadoop 每次 MapReduce 操作，启动一个 Task 便会启动一次 JVM，基于进程的操作。而 Spark 每次 MapReduce 操作是基于线程的，只在启动 Executor 时是启动 JVM，内存的 Task 操作是在**线程复用**的。每次启动 JVM 的时间消耗可能就需要几秒甚至十几秒，那么当 Task 多了，这个时间 Hadoop 不知道比 Spark 慢了多少。

不过Spark SQL 比 Hive 快并不是绝对的，也是有一定的条件的，而且不是 Spark SQL 的引擎比 Hive 的引擎快，相反，Hive 的 HQL 引擎还比 Spark SQL 的引擎更快。所以主要还是Spark 快，所以会表现出Spark SQL比Hive快

**结论** Spark 快不是绝对的，但是绝大多数，Spark 都比 Hadoop 计算要快。这主要得益于其对 mapreduce 操作的优化以及对 JVM 使用的优化。

