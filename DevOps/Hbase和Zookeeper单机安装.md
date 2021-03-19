# HBase安装

> 操作系统:centos7.4  
> chd的HBase发行版:hbase-1.2.0-cdh5.15.1  
> chd的Zookeeper发行版:zookeeper-3.4.5-cdh5.15.1.tar.gz

由于 HBase 是使用 Zookeeper 来做注册配置管理中心的，所以HBase的使用依赖zookeeper的，默认HBase安装的时候会内置一个zookeeper，但我们一般不采用这种方式，一般服务的部署组件都是分开部署的，所以这个安装文档，我们是单独安装一个Zookeeper，然后在安装HBase。

### Zookeeper安装

#### Zookeeper环境变量配置

* zkEnv.sh配置：`${ZOOKEEPER_HOME}/bin/zkEnv.sh、${ZOOKEEPER_HOME}/libexec/zkEnv.sh`

> 这边有两个一样的配置，默认情况下是优先读取 libexec/zkEnv.sh里面的内容，如果libexec/zkEnv.sh文件不存在，才会去读取/bin/zkEnv.sh，所以如果两个文件都存在，我们都配置一下避免问题发生，两个文件添加配置同样如下

```shell

export JAVA_HOME=/app/jdk/jdk1.8.0_231/
export ZOOKEEPER_HOME=/app/zookeeper/zookeeper-3.4.5-cdh5.15.1

```

#### Zookeeper配置

* zoo.cfg配置：`${ZOOKEEPER_HOME}/conf/zoo.cfg`，若该文件不存在，则创建一个

```properties

#Client-Server通信心跳时间
tickTime=2000

# 集群中的follower服务器与leader服务器之间初始连接时能容忍的最多心跳数
initLimit=10
 
# 集群中的follower服务器与leader服务器之间请求和应答之间能容忍的最多心跳数（tickTime的数量）syncLimit=5

#数据文件目录
dataDir=/app/zookeeper/data

# 服务监听端口
clientPort=2181

```

#### Zookeeper启动

* 启动Zookeeper：`${ZOOKEEPER_HOME}/bin/zkServer.sh start`

* 查看状态启动状态：`${ZOOKEEPER_HOME}/bin/zkServer.sh start`

* 尝试连接(后面的-server参数可以不填，默认连接本机)：`${ZOOKEEPER_HOME}/bin/zkCli.sh -server [host:port]`

### HBase安装

#### HBase环境变量配置

* hbase-env.sh配置:`${HBASE_HOME}/conf/hbase-env.sh`

```shell
export JAVA_HOME=/bigdata/jdk/jdk1.8.0_231
export HBASE_HOME=/bigdata/hbase/hbase-1.2.0-cdh5.15.1
#true代表使用内置zookeeper，false代表使用外置zk
export HBASE_MANAGES_ZK=false
```

#### HBASE配置
* hbase-site.xml配置：`${HBASE_HOME}/conf/hbase-site.xml`

```xml
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://hdh100:9000/hbase</value>
</property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>hdh100:2181</value>
</property>
```

* regionservers配置：`${HBASE_HOME}/conf/regionservers`  

```
#配置自己的主机名
${主机名}
```

#### 启动HBase

* 启动HBase：`${HBASE_HOME}/bin/start-hbase.sh`  


> 文章欢迎转载，转载请注明出处，个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片
