
> 操作系统：CentOS-7.8  
> redis版本：6.0.5  

### 一、下载源码

访问redis官网，下载redis包，[https://redis.io/](https://redis.io/)

### 二、安装依赖

* 安装gcc的c++编译环境：yum install -y gcc-c++

由于本次安装使用的 redis 是 6.0.5 版本所以需要升级gcc编译环境到5.3以上版本

升级gcc步骤如下：

```

#安装最新的centos软件集
yum -y install centos-release-scl

#在这个软件集里面包含的gcc大版本有7，8，9三个，我们直接安装9
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

#将配置写入环境变量
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
source /etc/profile

```

### 三、编译安装

* 解压redis包：tar -xzvf redis-6.0.5.tar.gz

* 进入解压后的目录，进行编译，命令：make

* 进行安装，命令：make install

### 四、redis配置与启动

在当前目录下，编辑 redis.conf 进行基本配置

配置如下

```

#redis进程是否在后台运行，默认是no，将其修改为yes允许后台运行
daemonize yes

#配置redis的工作目录，根据自己情况选择路径
dir /app/redis/workspace

#修改绑定的ip，默认是127.0.0.1仅允许本地用户访问，绑定0.0.0.0运行所有用户访问
bind 0.0.0.0

#设置密码，也可以不设置，建议设置密码
requirepass 123456

```

编辑启动脚本，在编译完的目录下面有一个utils的目录，下面有一个 redis\_init\_script 的脚本可以作为启动和关闭redis的管理脚本

vim redis\_init\_script 修改里面指定的配置文件路径

```

#将CONF这一项配置指向你的redis.conf配置文件
CONF="/app/redis/redis-6.0.5/redis.conf"

```

经过如上配置就可以启动redis了，使用刚刚配置的 redis\_init\_script 脚本，命令是：`redis_init_script start`

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ

