> 操作系统：CentOS-7.8  
> Spark版本：2.4.4

本篇文章介绍Spark Standalone模式的安装，并配置高可用

### 一、演示环境准备

三台服务器，主机名和ip如下：

* spark11：192.168.56.11 Master Worker

* spark12：192.168.56.12 Master Worker

* spark14：192.168.56.13 Worker

环境内需要安装Hadoop和zookeeper，关于Hadoop和zookeeper的安装可以参考[《Hadoop高可用安装》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483968&idx=1&sn=0c9bbe1bdf51dad53ee78c9fd4ded4b3&chksm=e8d7adf7dfa024e1ab0bc73cc032f8a75cf9eed1fa8b05bfbdec388dd37b914b6f7d5cc00737&token=312074909&lang=zh_CN#rd) 或者 [《教你安装单机版Hadoop》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483865&idx=2&sn=111833a442ffdcbd19b653e2c068c69a&chksm=e8d7ae6edfa02778694e125cbd3b065b55e67c006f31db165823503c4a5c67c215c01062cae7&token=312074909&lang=zh_CN#rd)

### 二、下载Spark与配置（高可用）

#### 1.下载Spark

* 直接在Spark官网下载：[http://spark.apache.org](http://spark.apache.org)

* 在Apache软件下载中心：[http://archive.apache.org/dist/spark/](http://archive.apache.org/dist/spark/)

打开以上两个网址选择你要安装的版本

#### 2.配置spark-env.sh

配置内容如下：

```

export JAVA_HOME=/app/jdk/jdk1.8.0_231

#指定Spark Master的地址，非高可用的情况下直接指明Master地址
#export SPARK_MASTER_HOST=spark11
export SPARK_MASTER_PORT=7077

#高可用配置
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=spark11:2181,spark12:2181,spark13:2181 -Dspark.deploy.zookeeper.dir=/spark"

```

#### 3.修改配置文件Slave

```

spark11
spark12
spark13

```

#### 4.配置spark-default.conf

首先在hdfs上创建日志目录：`{HADOOP_HOME}/bin/hadoop fs -mkdir -p /spark_log`

修改spark-default.conf内容如下：

```

spark.eventLog.enabled true
spark.eventLog.dir hdfs://spark11:8020/spark_log
spark.eventLog.compress true

```

#### 5.配置History Server

在spark—env.sh添加一下内容：

```

#history server相关配置
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4000 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://spark11:8020/spark_log"

```

#### 6.启动

启动Spark：`{SPARK_HOME}/sbin/start-all.sh`
启动history server：`{SPARK_HOME}/sbin/start-history-server.sh`
在其他节点启动多个SparkMaster：`{SPARK_HOME}/sbin/start-master.sh`

Spark页面访问：http://spark11:8080
Spark historyServer页面访问：http://spark11:4000

