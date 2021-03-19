> 操作系统：CentOS-7.8  
> keepalived版本：2.0.20  

### 一、下载源码

访问keepalived官网，下载keepalived包，[https://www.keepalived.org/download.html](https://www.keepalived.org/download.html)

### 二、安装依赖

* 安装gcc的c++编译环境：yum install gcc-c++

* 安装解析正则表达式的库：yum install -y pcre pcre-devel

* 安装数据压缩函式库：yum install -y zlib zlib-devel

* 安装用于安全通信的库：yum install -y openssl openssl-devel

* 安装Linux系统基于Netlink协议通信的API接口库：yum install libnl libnl-devel -y

### 三、编译安装

* 解压keepalived包：tar -xzvf keepalived-2.0.20.tar.gz

* 进入解压后的目录，进行构建：./configure 选择需要配置的选项

然后执行构建脚本命令如下：

```shell

#--prefix 参数是指定安装位置
#--sysconf 指定配置文件位置
./configure \
	--prefix=/usr/local/keepalived \
	--sysconf=/etc
	
```

* 进行编译，命令：make

* 进行安装，命令：make install

### 四、将keepalive注册为系统服务

在keepalived的解压目录编译后会生成一个keepalived目录，进入该目录如下，执行如下命令

* cd keepalived/etc/

* cp init.d/keepalived /etc/init.d/

* cp sysconfig/keepalived /etc/sysconfig/

* systemctl daemon-reload

经过如上配置即可使用systemctl 来管理keepalived

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ

