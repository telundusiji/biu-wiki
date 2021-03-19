# Hive安装

> 操作系统:centos7.4  
> CDH的Hive发行版:hive-1.1.0-cdh5.15.1

### 环境配置

* hive-env.sh配置:${HIVE_HOME}/conf/hive-env.sh

```shell
export JAVA_HOME=/bigdata/jdk/jdk1.8.0_231
export HADOOP_HOME=/bigdata/hadoop/hadoop-2.6.0-cdh5.15.1
export HADOOP_CONF_DIR==/bigdata/hadoop/hadoop-2.6.0-cdh5.15.1/etc/hadoop
export HIVE_HOME=/bigdata/hive/hive-1.1.0-cdh5.15.1
export HIVE_CONF_DIR=/bigdata/hive/hive-1.1.0-cdh5.15.1/conf
```

* hive-default.xml配置:${HIVE_HOME}/conf/hive-default.xml
```xml
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://hdh100:3306/hive?createDatabaseIfNotExist=true</value>
  <description>JDBC connect string for a JDBC metastore</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
  <description>username to use against metastore database</description>
</property>

<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>****</value>
  <description>password to use against metastore database</description>
</property>
```

* hive-log4j.properties配置:${HIVE_HOME}/conf/hive-log4j.properties

```
hive.log.dir=/bigdata/hive/hive-1.1.0-cdh5.15.1/logs
```

* 复制mysql的jdbc驱动到${HIVE_HOME}/lib

* 初始化mysql元数据库
```shell
${HIVE_HOME}/bin/schematool -dbType mysql -initSchema
```

>提示:mysql安装完成后，需要注意修改密码和开启对应host的原创访问权限，避免初始化元数据库失败  
>mysql修改密码的几种方式:  
>1)mysqladmin -uroot -p ${旧密码} password ${新密码}  
>2)sql语句:   
>&emsp;**use mysql;**  
>&emsp;**select user,host,password from user;**  
>&emsp;**update user set password=password('${新密码}') where user='${用户名}' and host='${登录主机}';**  
>&emsp;**flush privileges;**  
>或者：  
&emsp;**set password for ${用户名}@${主机}=password('${新密码}');**

### 使用hive

* hive命令行
```shell
${HIVE_HOME}/bin/hive
```

* hive启用metasore服务

```xml
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://hostname:10000</value>
</property>
```

```shell
#启用服务
${HIVE_HOME}/bin/hive --service metastore &
```

> 提示:再没有安装第三方数据库做元数据存储时，可以使用hive自带的临时元数据库*${HIVE_HOME}/bin/hive --service metastore

* hive启动hiveserver2

```shell
${HIVE_HOME}/bin/hive --service hiveserver2 &
```

* 使用Beeline连接hiveserver2

```shell
${HIVE_HOME}/bin/beeline
beeline> !connect jdbc:hive2://biu188:10000
Connecting to jdbc:hive2://biu188:10000
Enter username for jdbc:hive2://biu188:10000: root
Enter password for jdbc:hive2://biu188:10000:  #有密码则输入密码
```

> 若使用Beeline连接hiveserver2中报错User: root is not allowed to impersonate root ，则在hadoop的core-site.xml中加入下面配置，并重启hdfs
>
> ```xml
> 	<property>
> 		<name>hadoop.proxyuser.root.hosts</name>
> 		<value>*</value>
> 	</property>
> 	<property>
> 		<name>hadoop.proxyuser.root.groups</name>
> 		<value>*</value>
> 	</property>
> ```