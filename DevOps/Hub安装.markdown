# Hue安装

> 操作系统:centos7.4  
> chd的Hue发行版:hue-3.9.0-cdh5.15.1

### 依赖安装
* python和python-devel安装，一般centos系统自带有python，只需安装python-devel
```shell
yum search python-devel
yum install -y python-devel.x86_64
```
* 其他依赖安装(官网给出)
```shell
yum -y install ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi gcc gcc-c++ krb5-devel libtidy libxml2-devel libxslt-devel openldap-devel python-devel sqlite-devel openssl-devel mysql-devel gmp-devel
```

> 提示：如果环境没有安装maven，需要安装maven，编译过程会用到，命令：yum install -y maven

### 编译Hue

* 解压安装包，进入解压目录，执行编译命令
```
make apps
```

>提示：编译失败再此编译时，可以使用*shell make clean*和*make clean distclean*进行清理

### 准备和配置工作

* 配置hue.ini文件:${HUE_HOME}/desktop/conf/hue.ini
```
http_host=hdh100
http_port=8888
```

* 创建hue用户(**重要**)
```shell
useradd hue
chown -R hue:hue ${HUE_HOME}
```

### 启动HUE

* 使用supervisor脚本启动
```shell
${HUE_HOME}/build/env/bin/supervisor
```

* 访问web页面
```
http://hdh100:8888
```

>提示:停止hue命令:*ps -ef | grep hue | grep -v grep | awk '{print $2}' | xargs kill -9*




