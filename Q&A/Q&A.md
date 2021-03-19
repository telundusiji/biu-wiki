### mariadb安装以及初始化

安装：`yum install mariadb mariadb-server -y`

初始化：`mysql_secure_installation`

### mysql修改密码

>mysql修改密码的几种方式:  
>1)mysqladmin -uroot -p ${旧密码} password ${新密码}  
>2)sql语句:   
>&emsp;**use mysql;**  
>&emsp;**select user,host,password from user;**  
>&emsp;**update user set password=password('${新密码}') where user='${用户名}' and host='${登录主机}';**  
>&emsp;**flush privileges;**  
>或者：  
&emsp;**set password for ${用户名}@${主机}=password('${新密码}');**

### Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again

修改 /etc/security/limits.conf 如下

```properties

* soft nofile 102400
* hard nofile 102400
root soft nproc 102400
root hard nproc 102400

```

查看当前主机的程序资源配置：`ulimit -a`

### Hadoop-Error: Could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster

首先修改 mapred-site.xml 在原有内容基础上添加如下内容

```xml

<property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>

```

然后重启yarn

如果在使用hive的过程中遇到该问题，需要重启hive的服务才可以生效

### nullCatalogMeansCurrent

从MySql8.0驱动开始，该项的默认值变为false。该配置为true是表示，当使用DataBaseMetaData查询元数据时，catalog项为空时，查询当前库的表的信息；当该配置为false时，当使用DataBaseMetaData查询元数据时，catalog为空时，则表示查询当前数据源的所有库的信息

### Docker容器添加host信息

* 直接修改容器/etc/hosts 重启容器失效

* docker run 命令使用--add-host

```shell

docker run --add-host=myhostname:10.180.8.1 --name test -it debian

```

* 使用docker-compose时在yml中加入

```yaml
extra_hosts:
	- "hostname:192.168.56.1"
```

### 使用 & 运行后台进程，过一段时间进程自动结束

在Centos中使用 & 运行的后台进程，在登陆用户退出登录后，该进程就会关闭，解决办法使用 nohup+&。例如:

```shell
nohup hive --service metastore &
```

### 查看SpringCloud与SpringBoot版本对应关系+

直接访问 [https://start.spring.io/actuator/info](https://start.spring.io/actuator/info)

