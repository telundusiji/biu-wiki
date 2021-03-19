
> 操作系统：CentOS-7.8  
> nginx版本：1.18.0


## 一、概述

### 1、什么是Nginx

Nginx 是一个高性能的 WEB 和反向代理服务器，同时也提供了 IMAP/POP3/SMTP(邮件) 服务。Nginx采用C语言编写，它的作者是伊戈尔·赛索耶夫，第一个公开版本2004年发布，截至文章编写时nginx的最新稳定版是1.18.0。

Nginx因为它的稳定性、丰富的模块库、灵活的配置和低系统资源的消耗而闻名，Nginx常被用来做静态资源管理服务器和反向代理与负载均衡。

### 2、其他常用的web服务器

* Apache 

> Apache是世界上广泛使用的Web服务器软件，它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。19年之前Apache使用占有率居世界第一，可是随着Nginx的流行，现在Nginx已经占据了使用率第一的位置
 
* Tomcat

> Tomcat 服务器是一个免费的开放源代码的 Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，主要用于运行基于 Java 的 Web 应用软件，Tomcat 也具有处理 HTML 页面的功能，但是它处理静态 HTML 的能力不如 Apache 和 Nginx 服务器，所以 Tomcat 经常仅被用来提供API接口服务。 

* Jboss

> Jboss 是一个基于J2EE的开放源代码的应用服务器。 JBoss代码遵循LGPL许可，可以任何商业应用中免费使用

* WebLogic

> WebLogic是由Oracle出品，是一个用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器。WebLogic是一款商业软件，是面向企业的，需付费使用。


* IIS

> IIS服务器是微软旗下的WEB服务器，也是目前最流行的Web服务器产品之一。

* Kangle

> Kangle web服务器是一款跨平台、功能强大、安全稳定、易操作的高性能web服务器和反向代理服务器软件。

### 3、正向代理与反向代理

#### 代理

代理是网络信息的中转站，是个人网络和Internet服务商之间的中间机构，负责转发合法的网络信息，对转发进行控制和登记。

如下图是一个代理与非代理的比较，不存在代理时，用户直接与提供服务的服务器直接建立连接进行通信，存在代理时，用户和服务端之间需要经过代理服务器进行数据转发，并且代理也可以是多级的。

![非代理与代理](http://file.te-amo.site/images/thumbnail/%E9%9C%80%E8%A6%81%E5%AD%A6%E4%BC%9A%E7%9A%84Nginx%E7%9F%A5%E8%AF%86/proxy.png)

#### 正向代理

正向代理是一个位于客户端和服务端之间的服务器，客户端向代理服务器发送一个请求并指定目标服务端，然后代理服务器向指定的服务端转交请求并将获得的内容返回给客户端。

正向代理的过程可以将用户和代理服务器看为一体，从用户角度来看，服务端是可见的，用户清楚的知道服务端的信息，可以明确指定目标，而在服务端的角度来看，具体用户是不可见的，服务端可见的是代理服务器，至于代理服务器后面有多少具体用户，都是不可见的。如下图所示

![正向代理](http://file.te-amo.site/images/thumbnail/%E9%9C%80%E8%A6%81%E5%AD%A6%E4%BC%9A%E7%9A%84Nginx%E7%9F%A5%E8%AF%86/forward_proxy.png)


比如我们在公司内网环境，我们的电脑主机是无法访问外部服务器，如果此时我们需要访问外部服务器，这时我们的PC就会向代理服务器发送请求访问xxx服务器，代理服务器在收到请求后去访问指定服务器并将获得的数据再返回给发送请求的PC，这样就实现了访问。从例子,我们可以看出，我们的PC是明确知道要访问的目标，但是由于所有真实的请求都是代理服务器发出的，所以目标服务端只能看到代理服务器信息。

所以正向代理一般用在防火墙内网中提供防火墙外部网络的访问服务，一般公司中办公电脑访问外部网站都是通过正向代理来实现，这样可以做到对公司内部信息的监控和保护，内部信息向外泄露。

#### 反向代理

反向代理是一个位于客户端和服务端之间的服务器，反向代理服务器就相当于目标服务器，用户不需要知道目标服务器的地址，用户直接访问反向代理服务器就可以获得目标服务器提供的资源。

反向代理的过程是将代理服务器和服务端视为一体，在用户角度来看，具体服务端是不可见的，用户并不知道自己具体是从哪一个具体的服务器获取的数据，而站在服务端的角度来看，具体用户是可见的，服务端是知道是哪个用户发来的请求。如下图所示

![反向代理](http://file.te-amo.site/images/thumbnail/%E9%9C%80%E8%A6%81%E5%AD%A6%E4%BC%9A%E7%9A%84Nginx%E7%9F%A5%E8%AF%86/opposite_proxy.png)

比如我们的电脑要去访问www.baidu.com，当我们去访问时，我们的请求都是被分配到网站的反向代理服务器，反向代理服务器根据具体配置的规则，再将我们的请求转发给网站内部的具体服务器，然后再将数据返回给我们。

这样一个过程就将一个服务集群对用户进行了隐藏，用户只能看到暴露的代理服务器，而内部服务端内部具体的服务器是看不到的，也保证了内部服务器的安全性。反向代理一般被用来作为Web加速和负载均衡，以提高服务的可用性和访问效率

#### 一个具体栗子

我们在公司内网中的PC要访问www.baidu.com。图示如下

![代理栗子](http://file.te-amo.site/images/thumbnail/%E9%9C%80%E8%A6%81%E5%AD%A6%E4%BC%9A%E7%9A%84Nginx%E7%9F%A5%E8%AF%86/all_proxy.png)

上图过程可以描述为：

* a.用户PC向正向代理服务器发送请求，表明需要访问www.baidu.com

* b.正向代理服务器接收请求后，向www.baidu.com发送请求

* c.根据DNS解析出的IP指向了baidu的反向代理服务器

* d.baidu的反向代理服务器收到请求后，根据代理策略，从内部多台服务器中选择一台，向其发送请求

* e.baidu的反向代理服务器在获取到请求结果后，就将结果数据返回给发来请求的正向代理服务器

* f.正向代理服务器在收到数据后，将数据再返回给发来请求的PC

* g.最后PC收到数据后，这个请求就完成了

### 4、Nginx特性

#### 高并发

首先nginx整个服务采用了多进程master-worker的设计，nginx 在启动后，会有一个 master 进程和多个相互独立的 worker 进程（可配置）,master进程接收来自外界的连接，并向各worker进程发送信号，work进程用来处理具体连接。

其次nginx的IO模型是采用基于事件驱动的异步IO模型，且nginx使用的是最新的epoll和kqueue网络IO模型，
相比于select，epoll最大的好处在于它不会随着监听文件句柄数目的增长而降低效率，并且select在内核中也设置了监听句柄的上限为1024(可修改，需重新编译内核)，所以使用epoll也是nginx可以支持更多的并发原因
 
#### 内存消耗少

nginx有设计自己的内存池，使用内存池可以有减少向系统申请和释放内存的时间开销，也可以解决内存频繁分配产生的碎片，并且nginx的内存也有自己特殊的设计帮助其减少内存开销

#### 简单稳定

简单一般是指nginx的配置简单，其核心配置就一个配置文件，且配置内容没有非常繁琐复杂，对于一般的开发和使用者不会造成很大的困难。

稳定是指nginx性能比较稳定，可以长时间不间断运行，且支持热加载，修改配置等操作时不用重启服务

#### 低成本支持多系统

nginx是采用BSD开源协议，允许使用者修改代码或者发布自己的版本，所以使用在商业上也可以免费。nginx的代码是采用C语言编写，现在也已经移植到许多体系结构和操作系统，可以在不同的操作系统上安装使用。

## 二、nginx安装

参见文章[《学习必备——nginx安装》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483884&idx=1&sn=fddf903f62ed29f46077bf2c5b6b5513&chksm=e8d7ae5bdfa0274dbf6856c6b85f671d575fa4139cc8d7dfe2206d276ad06227c480c81ac2ec&token=548536718&lang=zh_CN#rd)

## 三、常用配置

### 1、常用命令

* 强制停止nginx：`./nginx -s stop`

* 正常关闭nginx，不再接受新连接，等待现有用户连接处理完成后退出nginx：`./nginx -s quit`

* 检测配置文件的正确性：`./nginx -t`

* 查看nginx当前版本：`./nginx -v`

* 查看nginx的详细信息：`./nginx -V`

* 指定nginx配置文件：`./nginx -c [filename]`

* 重新加载配置文件：`./nginx -s reload`

### 2、nginx.conf 配置说明
先看一下nginx.conf 的包含结构如下图所示：

![nginx.conf](http://file.te-amo.site/images/thumbnail/%E9%9C%80%E8%A6%81%E5%AD%A6%E4%BC%9A%E7%9A%84Nginx%E7%9F%A5%E8%AF%86/nginx_conf.png)

如下是一份安装完成后的默认配置,我将配置的说明也写在其中

```

#设置worker进程的所属用户，设置不同的用户由于用户权限不同所以对worker进程操作文件有影响，默认是nobody用户，查看nginx进程是用命令：ps -ef | grep nginx 就可以看到进程和所属用户
#user  nobody;
#设置worker进程的个数，默认设置为1个
worker_processes  1;

#错误日志配置，日志级别包括：debug、info、notice、warn、error、crit、alert、emerg
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#nginx的进程号
#pid        logs/nginx.pid;

#events模块中包含nginx中所有处理连接的设置
events {
    #选择nginx连接所使用的IO模型，Linux下建议使用epoll
    use epoll;
    #每个worker允许的最大连接数
    worker_connections  1024;
}

#http相关配置，内部可以嵌套多个server，配置代理，缓存，日志等绝大多数功能和第三方模块的配置
http {
    #mime.type是一个文件，里面定义了可使用的文件类型与文件拓展名的映射
    include       mime.types;
    
    #默认的文件类型
    default_type  application/octet-stream;
    
    #设置一个日志格式化的格式，名字叫‘main’，里面包含的参数含义如下：
    #$remote_addr	客户端ip
    #$remote_user	远程客户端用户名
    #$time_local		时间和时区
    #$request	请求的url以及method
    #$status	响应状态码
    #$body_bytes_send		响应客户端内容字节数
    #$http_referer	记录用户从哪个链接跳转过来的
    #$http_user_agent		用户所使用的代理，一般来时都是浏览器
    #$http_x_forwarded_for		通过代理服务器来记录客户端的ip
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
    #设置access_log的位置并制定使用main格式进行格式化
    #access_log  logs/access.log  main;

    #允许sendfile方式传输文件
    sendfile        on;
    #开源tcp_nopush时，数据包累积到一定大小时再发送，tcp_nopush必须和sendfile配合使用，是提高文件传输效率的
    #tcp_nopush     on;

    #连接超时时间，单位是秒，保证客户端多次请求的时候不会重复建立新的连接，节约资源损耗。设定为0时，客户端每次连接都是新建连接
    #keepalive_timeout  0;
    keepalive_timeout  65;
    
    #开启gzip压缩，进行数据传输时进行压缩
    #gzip  on;

    #一个虚拟的主机配置
    server {
        #监听的端口号
        listen       80;
        #监听的地址，虚拟主机名
        server_name  localhost;
        #字符集
        #charset koi8-r;
        
        #access日志的配置，当前只是配置当前server的日志
        #access_log  logs/host.access.log  main;
        
        #路由规则配置8
        location / {
            #当前路由的根目录
            root   html;
            #默认主页
            index  index.html index.htm;
        }
        
        #默认404错误页配置
        #error_page  404              /404.html;
        
        #50x错误页配置
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    


    #另一个虚拟主机配置
    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    
    #Https的虚拟主机配置
    # HTTPS server
    #
    #server {
    #    监听端口
    #    listen       443 ssl;
    #    监听地址
    #    server_name  localhost;
    #
    #    ssl安装连接相关配置
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

### 3.location路由匹配规则解析

#### 通用匹配

```

location / {
    root   html;
}

```

示例：http://host:port/a/b  
在“/"后面值的路由都会被匹配

#### 精确配置

```

location =/ {
    root   html;
}

```
示例：http://host:port/
只能配置 ‘/’ 后面如果添加其他路由是无法匹配的

#### 正则匹配

```

location ~* /ca {
    root   html;
}

location ~ /CA {
    root   html;
}

```

正则匹配分为区分大小写和不区分大小写两种。
‘*’ 代表不区分大小写，去掉‘*’则匹配时区分大小写

示例：

* “~* /case”可以匹配：http://host:port/ca 、 http://host:port/Ca 、 http://host:port/cA 和http://host:port/CA 

* “~ /CASE”只能匹配：http://host:port/CA

#### 精准前缀匹配

```

location ^~ /ca {
    root   html;
}

```

示例：http://host:port/ca/file
只能匹配以ca开头的下一级，例如“ca/file/a”就是无法匹配


#### 匹配优先级

在多个匹配规则同时存在时，精确匹配优先级最高，其次是精准前缀匹配，然后是区分大小写的正则匹配，再然后是不区分大小写的正则匹配，最后是通用匹配。

举个例子

```

精确匹配：location =/ca 
精确前缀匹配：location ^~ /ca 
区分大小写正则匹配：location ~ /ca 
不区分大小写正则匹配：location ~* /ca 
通用匹配：location /  

```

请求url：http://host:port/ca

* 5种规则全存在，则匹配精确匹配

* 2-5存在，则匹配精确前缀匹配

* 3-5存在，则匹配区分大小写的正则匹配

* 4-5存在，则匹配不区分大小写正则匹配

* 仅有5存在，则匹配通用匹配

### 4.跨域配置与防盗链

在server块的配置中进行配置跨域支持

```

#允许跨域请求的域，*代表所有
add_header 'Access-Control-Allow-Origin' *;
#允许带上cookie请求
add_header 'Access-Control-Allow-Credentials' 'true';
#允许请求的方法，比如 GET/POST/PUT/DELETE，*代表所有方法
add_header 'Access-Control-Allow-Methods' *;
#允许请求的header，*代表所有的header
add_header 'Access-Control-Allow-Headers' *;
 
```

防盗链配置如下

```

#对发送请求的域名验证
valid_referers *.domain.com; 
#验证不通过则返回404
if ($invalid_referer) {
    return 404;
} 

```


> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
