
> 操作系统：CentOS-7.8  
> nginx版本：1.18.0  
> Tomcat版本：9.0.36

在上一篇文章中我们对Nginx进行了简单介绍，也学习了nginx的基本配置和操作，今天这次我们将学习Nginx在生产中常被使用的功能负载均衡和缓存加速。上一篇文章[《需要学会的Nginx知识》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483899&idx=1&sn=75c3becc882018e2cdabf0227aaa7729&chksm=e8d7ae4cdfa0275ae830f5b596e2b05ad2e88dc3f69b8855fbc035fe6080643d2ebe0ef352e5&token=1523054489&lang=zh_CN#rd)

## 一、概念

### 1.OSI七层网络模型

OSI七层网络模型是国际标准化组织（ISO）制定的一个用于计算机或通信系统间互联的标准体系

#### 应用层。网络服务与最终用户的一个接口，与用户交互

应用层是与用户最近的一层，应用层为用户提供了交互接口，以此来为用户提供交互服务。在这一层常用的协议有HTTP，HTTPS，FTP，POP3等。

#### 表示层。数据的表示、安全、压缩

表示层提供数据格式编码以及加解密功能，确保从请求端发出的数据到响应端能被识别。

#### 会话层。创建、管理和销毁会话

会话存在于从请求发出到接受响应这个过程之间，会话层充当这一过程的管理者，包括会话创建、维持到销毁

#### 传输层。定义传输数据的协议端口号，以及流控和差错校验等功能

传输层创建以及管理端到端的连接，提供数据传输服务。在传输层常见的协议是TCP、UDP协议

#### 网络层。进行逻辑地址寻址，实现不同网络之间的路径选择

网络层也被称为IP层，在网络层中通过IP寻址来建立两个节点之间的连接，选择合适的路由和交换节点，正确无误地按照地址将数据传送给目的端的运输层。

#### 数据链路层。建立逻辑连接、进行硬件地址寻址、差错校验，提供介质访问和链路管理

数据链路层会提供计算机MAC地址，通信的时候会携带，为了确保数据的正确投递，会对MAC地址进行校验，以保证请求响应的可靠性

#### 物理层。传输介质，物理媒介

物理层是端到端通信过程的物理介质，实际最终信号的传输都是通过物理层实现的。常用设备有（各种物理设备）集线器、中继器、调制解调器、网线、双绞线、同轴电缆等，这些都是物理层的传输介质。

### 2.负载均衡

负载均衡是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，在网络应用中就是根据请求的信息不同，来决定怎么样转发流量。

负载均衡建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展网络设备和服务器的带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。

在OSI七层网络模型的基础上，负载均衡也有不同的分类，主要有以下几种

* 二层负载：基于MAC地址，通过一个虚拟MAC地址接受请求，然后再分配到真实的MAC地址。

* 三层负载：基于IP地址，通过一个虚拟IP地址，然后再分配到真实的IP。

* 四层负载：基于IP和端口，通过虚机的IP+端口接收请求，然后再分配到真实的服务器，LVS是典型的四层负载均衡应用

* 七层负载：基于应用信息，通过虚机主机名或者URL接收请求，再根据一些规则分配到真实的服务器，nginx就是典型的七层负载均衡应用

## 二、配置

### 1、负载均衡策略

#### 轮询

轮询策略下，所有上游服务器被访问到的概率是一致的，且是按照一定的顺序依次被请求到。如下示例配置，hdh100:9001、hdh100:9002和hdh100:9003三台tomcat服务器依次被请求，每刷新一次就切换一次。

```

#上游tomcat集群
upstream www.learn.com{
        server hdh100:9001;
        server hdh100:9002;
        server hdh100:9003;
}

#反向代理服务器
server {
        listen 80;
        server_name www.learn.com;

        location / {
                proxy_pass http://www.learn.com;
        }
}

```

#### 加权轮询

加权轮询策略下，是按照每个上游服务器分配的权重不同处理的连接数也不同，权重越大，被访问到的次数就越多。如下示例配置，hdh100:9001、hdh100:9002和hdh100:9003的权重分别是1，3，5，权重越大，被分配的流量就越多。

```

#上游tomcat集群，后面weight就是配置权重，在设置weight时，默认权重是1
upstream www.learn.com{
        server hdh100:9001 weight=1;
        server hdh100:9002 weight=3;
        server hdh100:9003 weight=5;
}

#反向代理服务器
server {
        listen 80;
        server_name www.learn.com;

        location / {
                proxy_pass http://www.learn.com;
        }
}

```

#### ip_hash

ip_hash是根据用户的ip进行hash散列将其请求分配到指定上游服务器。在用户ip没有发生变化的情况下，且上游服务器无变更时，用户的多次请求都会被转发到同一台上游服务器。

且在使用ip_hash负责均衡策略时，如果需要临时删除其中一台服务器，则应使用down参数对其进行标记，以便保留客户端IP地址的当前哈希值。

```

upstream www.learn.com{
	   ip_hash;
	   
        server hdh100:9001 weight=1;
        server hdh100:9002 weight=3;
        server hdh100:9003 weight=5;
}

```

#### url\_hash

url\_hash策略是根据url进行hash散列将其请求分配到指定的上游服务器。同一个url在上游服务器没有发生变更的情况下是请求到同一台服务器。

```

upstream www.learn.com{
	   hash $request_uri;
	   
        server hdh100:9001 weight=1;
        server hdh100:9002 weight=3;
        server hdh100:9003 weight=5;
}

```

#### last\_conn

last_conn将流量分发到当前连接数最少的服务器上的一个策略

```

upstream www.learn.com{
	   least_conn;
	   
        server hdh100:9001 weight=1;
        server hdh100:9002 weight=3;
        server hdh100:9003 weight=5;
}


```

### 2、优化配置

#### max\_conns

限制上游服务器的最大连接数，用于保护避免过载，可以起到限流的作用，在nginx1.11.5之前，该配置只能作用于商业版本的nginx，默认值是0表示没有限制，配置如下：

```

#上游服务器配置，每个服务器都可以设置max_conns
upstream www.learn.com{
        server hdh100:9001 max_conns=200;
        server hdh100:9002 max_conns=200;
        server hdh100:9003 max_conns=200;
}

```

#### slow\_start

缓慢开始，当我们给上游服务器设置权重时，在配置了slow_start的情况下，该服务的权重是从0在配置的时间内慢慢增长到所配置的权重，该配置目前是在商业版本的nginx中生效

* slow_start不适用hash和random load balancing

* 上游服务器只有一台的情况下，该参数也无效

如下配置则表示hdh100:9003这个服务，在启动后的60s内权重是从0慢慢提升到5。

```

#上游服务器配置，给hdh100:9003配置了权重5和slow_start 60s，则在集群启动后该服务器的权重是在60s的时间内慢慢从0变为5
upstream www.learn.com{
        server hdh100:9001 weight=1;
        server hdh100:9002 weight=3;
        server hdh100:9003 weight=5 slow_start=60s;
}


```

#### down和backup

down是将标记的服务器移除当前的服务器集群，被标记的服务器不会被访问到。

backup表示的是备用的意思，即在其他服务器可以提供服务的时候被backup标记的服务器不会被访问到，当其他服务器都挂掉后，backup标记的服务开始提供服务。

```

#down配置示例，如下配置，则hdh100:9001这台服务器是不能提供服务的，不会被分配流量
upstream www.learn.com{
        server hdh100:9001 down;
        server hdh100:9002;
        server hdh100:9003;
}

#backup配置示例，如下配置，在hdh100:9002和hdh100:9003正常服务的时候，hdh100:9001不会提供服务，仅当另外两台服务都挂掉后，hdh100:9001开始提供服务
upstream www.learn.com{
        server hdh100:9001 backup;
        server hdh100:9002;
        server hdh100:9003;
}


```
#### max\_fails和fail\_timeout

max_fails是一个验证服务器是否能提供正常服务的校验，需结合fail_timeout使用，表示在一定时间段内请求当前服务失败的次数超过该值时，则认为该服务宕机失败掉，就将该服务剔除上游服务器集群，不再给其分配访问流量。

fail_timeout表示失败服务的重试时间，即当该服务器被认为宕机失败时，间隔指定时间后再来重新尝试请求，如果仍请求失败则继续间隔指定时间再尝试请求，一直重复这种行为

```

#如下是示例配置，表示在30s的时间内请求hdh100:9001的失败次数超过10次时，则认为hdh100:9001服务宕机，则随后的30s内该服务器不再被分配请求，30s后会重新尝试请求该服务器，若仍然请求失败，则重复上述行为
upstream www.learn.com{
        server hdh100:9001 max_fails=10 fail_timeout=30s;
        server hdh100:9002;
        server hdh100:9003;
}

```
#### keepalive

设置nginx和上游服务器直接保持的长连接数量，目的是为了减少创建连接和关闭连接的损耗

```

#如下示例，设置保持的长连接数量为32
upstream www.learn.com{
        server hdh100:9001 max_fails=10 fail_timeout=30s;
        server hdh100:9002;
        server hdh100:9003;
        
        keepalive 32
}

#设置http版本为1.1，默认是http1.0版本是短连接
#设置Connection header为空
server {
        listen 80;
        server_name www.learn.com;

        location / {
                proxy_pass http://www.learn.com;
                proxy_http_version 1.1;
                proxy_set_header Connection "";
        }
}


```

### 3、缓存配置

#### nginx缓存

浏览器中的缓存加快单个用户浏览器的访问的速度，缓存是存在浏览器本地。

```

location / {
            root   html;
            index  index.html;
            
            #设置缓存过期时间为10s
            # expires 10s;
            #设置具体到期时间，如下示例代表缓存过期时间为晚上10点30分时
		 # expires @22h30m;
		 #设置过期时间为现在的前一个小时，则代表不使用缓存
		 # expires -1h;
		 #设置不使用缓存
		 # expires epoch;
		 # nginx关闭expires，则缓存机制是根据浏览器而定的
		 # expires off;
		 #设置缓存过期时间最大
		 # expires max;
        }
        
```

#### nginx反向代理缓存

反向代理缓存是将，要代理的上游服务器的数据缓存在nginx端，目的是为了提升访问nginx端的用户的访问速度

```

#proxy_cache_path 设置存储存储目录
#keys_zone 设置名称以及共享内存大小
#max_size 设置缓存大小
#inactive设置缓存过期时间，超过该时间则被清理
#use_temp_path设置是否使用临时目录，默认是使用，在这里设置关闭
proxy_cache_path /home/local/cache  keys_zone=learncache:5m max_size=2g inactive=60s use_temp_path=off;

server {
        listen 80;
        server_name www.learn.com;

	   #开启缓存
	   proxy_cache learncache;
	   #对200和304状态的请求设置缓存过期时间
	   proxy_cache_valid 200 304 1h;

        location / {
                proxy_pass http://www.learn.com;
        }
}


```


> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
