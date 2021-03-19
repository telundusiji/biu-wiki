# hadoop单机安装


>操作系统:centos7.4  
cdh的hadoop发行版:hadoop-2.6-chd5.15.1 

### ssh免密登录配置

* ssh安装(已安装跳过):yum install -y openssh-server openssh-clients
* 生成密钥:ssh-keygen -t rsa
* 拷贝密钥:ssh-copy-id -i ~/.ssh/id_rsa.pub [ip/hosts]

> 注意事项:
> 1) .ssh目录的权限必须是700  
> 2) .ssh/authorized_keys文件权限必须是600

### hadoop环境变量配置
* hadoop-env.sh配置:${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
```shell
export JAVA_HOME=/bigdata/jdk/jdk1.8.0_231
export HADOOP_HOME=/bigdata/hadoop/hadoop-2.6.0-cdh5.15.1
export HADOOP_CONF_DIR=/bigdata/hadoop/hadoop-2.6.0-cdh5.15.1/etc/hadoop
```

### HDFS启动
* core-site.xml配置:${HADOOP_HOME}/etc/hadoop/core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://主机名:9000</value>
    </property>
</configuration>
```
* hdfs-site.xml配置:${HADOOOP_HOME}/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

* slave文件配置:${HADOOP_HOME}/etc/hadoop/slaves
```
#根据自己的主机名填写
${主机名}
```

* HDFS文件系统格式化:${HADOOP_HOME}/bin/hdfs namenode -format

* 启动HDFS:${HADOOP_HOME}/sbin/start-dfs.sh

### YARN启动

* mapred-site.xml配置:${HADOOP_HOME}/etc/hadoop/mapred-site.xml
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

* yarn-site.xml配置：${HADOOP_HOME}/etc/hadoop/yarn-site.xml
```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
* 启动yarn：${HADOOP_HOME}/sbin/start-yarn.sh