
> 操作系统：CentOS-7.8  
> keepalived版本：2.0.20  
> nginx版本：1.18.0

本篇文件主要是介绍keepalived和LVS的基本概念和基本操作，nginx相关知识锤子在之前的文章写过，需要了解的可以参考
[《需要学会的Nginx知识》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483904&idx=2&sn=3fe1f9c3ff855182e250d582cffd2fab&chksm=e8d7adb7dfa024a183cb42aafb120fd69c95f260e5eaa2d3e3409258fd61b0dca924727344a3&token=691604302&lang=zh_CN#rd) ，[《需要学会的Nginx知识——负载均衡和缓存》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483904&idx=1&sn=c7d29d934e5298a63cd30d58b05b6b70&chksm=e8d7adb7dfa024a149c782e4e5008417a0fdbc7315d05b0efbe270be29d83e104ab28ca89baf&token=691604302&lang=zh_CN#rd)

## 一、keepalived

keepalived是在Linux系统下的一个轻量级的高可用解决方案，是使用C语言编写的，它主要目标是为Linux系统和基于Linux的基础架构提供简单而可靠的负载均衡和高可用。在 Keepalived 中实现了一组检查器，可以根据服务集群中服务器的健康状态，自动的进行动态维护管理。

### 1.VRRP

VRRP（Virtual Router Redundancy Protocol）虚拟路由器冗余协议。VRRP协议是一种容错的主备模式的协议，保证当主机的下一跳路由出现故障时，由另一台路由器来代替出现故障的路由器进行工作，通过VRRP可以在网络发生故障时透明的进行设备切换而不影响主机之间的数据通信


### 2.高可用

Keepalived软件主要是通过VRRP协议的设计思路实现高可用功能，在安装keepalived的服务器主机上会在配置文件中设置一个虚拟IP，当该keepalived节点为主节点且正常运行时，设置的虚拟Ip就会在该节点生效且绑定在该主机的网卡上，而其他备用主机设置的虚拟IP就不会生效。当备用keepalived节点检测到主keepalived节点出现故障时，会进行抢占提供服务，抢占成功的keepalived节点就会将配置的虚拟IP绑定在自己的网卡上，这样对外部用户来说虚拟IP提供的服务是一直可用的，这也就是keepalived基于VRRP实现的高可用。

keepalive的自动化体现在他的故障检测和排除机制，Keepalived可以检测服务器的状态，如果服务器群的一台服务器宕机或工作出现故障，Keepalived将检测到，并将有故障的服务器从服务集群中剔除，同时使用配置的其他备用的服务器节点代替该服务器的工作，当故障服务器被修复后可以正常工作时Keepalived会自动的将该服务器加入到服务器群中。在整个过程中，故障检测、故障服务器剔除以及修复后的服务器重新上线这些操作都是由keepalived自动完成，运维人员只需要关注故障服务器的修复。

### 3.keepalived安装

keepalive的安装参考[学习必备——Keepalived安装](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483909&idx=1&sn=c78a8157d0262a2daecb8175eecfc6a5&chksm=e8d7adb2dfa024a4e4be74c33c54a21ead117f78ea094db0d92dcb4ce42e70e33682bc6b78a6&token=691604302&lang=zh_CN#rd)

### 4.使用keepalived搭建一个高可用nginx服务

#### 准备工作

服务搭建示意图如下

![nginx.conf](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/single_master_cool_backup.png)

从图中我们可以看出，有两台服务器Server1（192.168.56.101）和Server2（192.168.56.102）。每台服务器上都运行一个nginx实例和一个keepalived实例，其中Server1的keepalived实例是Master节点，Server2的keepalived实例是备用节点，两个keepalived实例配置的虚拟IP为192.168.56.102

#### 配置

nginx配置不进行特殊设置，保证默认即可。在nginx的html页面里面加上一些不同文章便于区分，启动nginx服务，保证可以正常访问即可，如下图所示

![nginx.conf](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/nginx_page.png)

接下来对keepalived进行配置，keepalived的配置文件位置是在安装的时候指定，锤子的配置文件在/etc/keepalived/keepalived.conf，相关配置如下

keepalive-master配置

```

! Configuration File for keepalived

global_defs {
   #路由id，全局唯一，表示当前keepalived节点的唯一性
   router_id keep_101
}

vrrp_instance VI_1 {
    #设置当前实例状态为MASTER。MASTER代表是主实例，BACKUP代表是备用实例
    state MASTER
    #当前实例绑定的网卡
    interface enp0s8
    #当前实例的虚拟路由id，一组主备的实例的路由id是相同的
    virtual_router_id 51
    #当前实例的优先级
    priority 100
    #主备之间同步检查时间间隔
    advert_int 1
    #一组主备实例的认证密码，方式非法节点进入路由组
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    #设置当前实例的虚拟IP
    virtual_ipaddress {
        192.168.56.100
    }
}


```

keepalive-backup配置（只标注于keepalived-master不同的点）

```

! Configuration File for keepalived

global_defs {
   #由于是全局唯一Id，所有需要与master保持不同
   router_id keep_102
}

vrrp_instance VI_1 {
    #备用实例状态应设置为BACKUP
    state BACKUP
    interface enp0s8
    virtual_router_id 51
    #设置备用实例的优先级低于主实例，这样可保证在主实例故障修复后可以再次将主节点抢占回来
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.56.100
    }
}


```

#### 启动验证

启动两台服务器的keepalived，通过虚拟ip访问nginx服务，然后kill掉Server1上的keepalived验证是否保证虚拟ip正常访问，验证图示如下

![nginx.conf](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/single_master_cool_backup_page.png)

图中我们可以看出，当两个keepalived实例都正常运行时，虚拟IP是绑定在Server1的网卡上对外提供服务，当我们kill掉Server1上的keepalived实例后，虚拟IP自动绑定到了Server2来提供服务，这样就保证了服务的高可用，对于外部用户来说通过虚拟IP来访问服务并不会感知到服务发生了故障，整个过程对外部用户来说是透明的。

## 二、LVS

LVS（Linux Virtual Server）Linux虚拟服务器，是一个虚拟的服务器集群系统。LVS在Linux内核中实现了基于IP的内容请求分发的负载均衡调度解决方案，属于四层负载均衡。LVS调度器具有很好的吞吐率，将请求均衡地转移到不同的服务器上执行，且调度器自动屏蔽掉服务器的故障，从而将一组服务器构成一个高性能的、高可用的虚拟服务器。在使用LVS负载均衡时，用户的请求都会被LVS调度器转发到真实服务器，真实服务器在处理完请求后将数据发送给用户时会有多种方式（三种）可以选择，整个服务器集群的结构对客户是透明的

### 1.工作模式

#### NAT

NAT（Network Address Translation）网络地址转换，通过修改数据报头，使得内网中的IP可以和外部网络进行通信。LVS负载调度器使用两块不同的网卡配置不同的IP地址，网卡一设置为公网IP负责与外部通信，网卡二设置内网IP负责与内网服务通信。

外部用户通过访问LVS调度器的公网IP发送服务请求，LVS调度器接受请求后，将请求数据进行转换，通过内网IP的网卡设备，根据调度策略将数据转发给内部服务，内部服务处理完成将响应数据再返回给LVS调度器，LVS调度器再将数据转换通过公网IP的网卡设备将响应结果返回给请求用户。

以上描述的就是一个基于NAT工作模式的LVS调度，这种模式的瓶颈在于LVS调度器，因为所有的请求数据和响应数据都需要经过LVS来进行转换处理，当大流量到来时，LVS调度器就成了一个短板，限制整个集群服务性能的上限。

#### TUN

TUN模式与NAT的不同在于，TUN模式下LVS调度器只负责接受请求，而真实服务器进行响应请给用户。LVS调度器与真实服务器建立IP隧道，IP隧道它可以将原始数据包封装并将新的源地址及端口、目标地址及端口添加新的包头中，将封装后的数据通过隧道转发给后端的真实服务器，真实服务器在收到请求数据包后直接给外部用户响应数据，这种模式下要求真实服务器具有直接外部用户通信的能力。

外部用户访问LVS调度器发送服务请求，LVS调度器接收请求后，将请求数据转换，根据调度策略将数据转发给服务集群真实服务器，真实服务器在处理完成后，就直接与外部请求用户通信，直接将响应结果返回给请求用户。

以上描述就是一个基于TUN工作模式的LVS调度，这种模式下LVS调度器就只负责请求的负载均衡转发，而处理数据的响应则全部由真实服务器来直接和用户通信了。在实际环境中，请求的数据量往往是小于响应的数据量，所以仅仅将请求数据让LVS来转发，好处就是LVS调度器的压力减少很多，可以承载更大的流量，同时真实服务器的性能也能得到充分利用，缺点就是真实服务器需要与外部网络用户直接通信，在安全上会存在一定风险。

#### DR

DR模式是在TUN模式的基础上又进行了改造，在DR模式下LVS调度器与真实服务器共享一个虚拟IP，且调度器的虚拟IP对外暴露，而真实服务器的虚拟IP地址将配置在Non-ARP的网络设备上，这种网络设备不会向外广播自己的MAC及对应的IP地址，这样即保证了真实服务器可以接受虚拟IP的网络请求也让真实服务器所绑定的虚拟IP对外部网络部是不可见的。

外部用户通过访问虚拟IP将请求数据包发送到调度器，调度器根据调度策略确定转发的真实服务器后，在不修改数据报文的情况下，将数据帧的MAC地址修改为选出的真实服务器的MAC地址，通过交换机将该数据帧发给真实服务器，之后真实服务器在处理完后进行数据响应时，会将虚拟IP封装在数据包中，再经过路由将数据返回给外部用户，在这整个过程中，真实服务器对外部用户不可见，外部用户只能看到虚拟IP的地址

在DR模式下因为真实服务器给外部用户回应的数据包设置的源IP是虚拟IP地址，又因为真实服务器的虚拟IP不对外暴露，这样外部用户在通过虚拟IP访问时，就访问到了调度器的虚拟IP地址，就实现了整个集群对外部用户透明。

### 2.负载均衡算法

#### 轮询(RR)

轮询（Round Robin），循环的方式将请求调度到真实服务器，所有的请求会被平均分配给每个真实服务器。

#### 加权轮询(WRR)

加权轮询（Weight Round Robin），轮询调度的一种补充，每台真实服务器配置一个权重，轮询过程中，权重越高的服务器，被分配的请求越多。

#### 最小连接(LC)

最小连接（Least Connections），把新的连接请求分配到当前活跃连接数最小的服务器。

#### 加权最小连接(WLC)

加权最少连接（Weight Least Connections），在LC调度算法上的补充，调度新连接时尽可能使真实服务器的已建立连接数和其权重成比。

#### 基于局部的最小连接(LBLC)

基于局部的最少连接（Locality-Based Least Connections），先根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则使用'最少连接'的原则选出一个可用的服务器。

#### 带复制的基于局部性的最少连接(LBLCR)

带复制的基于局部性的最少连接（Locality-Based Least Connections with Replication），按'最小连接'原则从该服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器；若服务器超载，则按'最小连接'原则从整个集群中选出一台服务器，将该服务器加入到这个服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，则将最忙的服务器从服务器组中删除。

#### 目标地址散列(DH)

目标地址HASH（Destination Hashing），根据请求的目标IP地址进散列计算从散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空。

#### 源地址散列调度(SH)

源地址HASH（Source Hashing），根据请求的源IP地址进行散列计算从散列表找出对应的服务器，若该服务器是可用的且并未超载，将请求发送到该服务器，否则返回空。

#### 最短的期望的延迟(SED)

最短的期望的延迟（Shortest Expected Delay），基于加权最小连接的一个调度算法，会根据权重，当前连接数，进行期望计算，将连接分配给计算结果最小的服务器

#### 最少队列(NQ)

最少队列（Never Queue），永不使用队列。

### 3.使用LVS对集群进行DR模式的负载均衡

#### ipvsadm安装

从Linux内核的2.4版本开始在内核中就已经集成了LVS功能。ipvsadm是LVS的管理工具，使用yum install -y ipvsadm 即可直接安装。

可以尝试使用一下命令 ipvsadm -Ln，若命令可以使用则代表已经安装

#### 准备工作

环境说明：

DR模式下虚拟IP：192.168.56.130

三台服务器主机包括：一台LVS调度主机和两台真实服务主机

LVS调度主机真实IP：192.168.56.121

两台真实服务器的真实IP：192.168.56.101和192.168.56.102

两台真实服务器分别按照了nginx服务，仅做基本基本配置便于区分

![lvs_serverinfo.conf](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/lvs_serverinfo.png)

* LVS调度器配置虚拟IP

如果你是虚拟机建议在操作前关闭NetworkManager，由于虚拟的网卡都是虚拟的以免自动的网络管理导致出现问题，使得实验失败

进入网卡配置目录：cd /etc/sysconfig/network-scripts

根据自己的网卡情况选择虚拟ip要绑定的网卡，锤子的网卡名字是enp0s8，所以我会将虚拟ip绑定在此网卡上。

复制一份配置：cp ifcfg-enp0s8 ifcfg-enp0s8:1

vim ifcfg-enp0s8:1 配置如下：

```

#网卡名称需要更高
DEVICE=enp0s8:1
ONBOOT=yes
#IP地址修改为虚拟IP地址
IPADDR=192.168.56.130
NETMASK=255.255.255.0
BOOTPROTO=static

```

重启网络服务：systemctl restart network.service

查虚拟IP是否生效：ifconfig enp0s8:1 ，如可以查到ip信息则配置成功

* 为真实服务器配置虚拟IP

我们演示的DR模式的LVS负载均衡，所以真实服务器的虚拟IP将和LVS调度器的虚拟IP一样，且真实服务器的虚拟IP不能对外暴露，所以我们会将虚拟IP绑定在lo回环接口上。由于两台真实服务器的虚拟ip配置方式一样，如下配置就只演示一次。

进入网卡配置目录：cd /etc/sysconfig/network-scripts

复制一份配置：cp ifcfg-lo ifcfg-lo:1

vim ifcfg-lo:1 配置如下：

```

#主要修改DEVICE、IPADDR、NETMASK这三项如下，其他配置保留不变即可
DEVICE=lo:1
IPADDR=192.168.56.130
NETMASK=255.255.255.255

```

重启网络服务：systemctl restart network.service

查虚拟IP是否生效：ifconfig lo:1 ，如可以查到ip信息则配置成功

* 为真实服务器配置ARP

同样两台真实服务器配置过程相同，仅演示一次。

配置ARP响应级别和通告行为，关于这方面知识，在这里不细讲，有兴趣的朋友可以自己查阅资料。vim /etc/sysctl.conf ，加入如下内容：

```

# configration for lvs
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.lo.arp_ignore = 1

net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce = 2

```

刷新配置：sysctl -p

增加路由（单次生效）：route add -host 192.168.56.130 dev lo:1

可以将该路由配置命令（route add -host 192.168.56.130 dev lo:1）追加在rc.local中，防止重启失效

查看路由表：route -n ，看到如下配置，代表路由配置成功

```

192.168.56.130  0.0.0.0         255.255.255.255 UH    0      0        0 lo

```

#### 配置负载集群规则

* 添加LVS节点信息

```

ipvsadm  -A -t 192.168.56.130:80 -s wrr -p 5

```
说明如下：

-A 添加LVS节点

-t 代表TCP协议，后面是负载的虚拟ip和端口

-s 负责均衡算法，wrr代表加权轮询

-p 连接持久化的世界

* 添加真实服务器信息

```

ipvsadm -a -t 192.168.56.130:80 -r 192.168.56.101:80 -g
ipvsadm -a -t 192.168.56.130:80 -r 192.168.56.102:80 -g

```

说明如下：

-a 添加真实服务器

-t tcp协议，负载虚拟IP和端口

-r 真实服务器IP和端口

-g 工作模式DR

* 保存配置

ipvsadm -S

#### 验证

ipvsadm -Ln 查看负载集群信息

```

IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.56.130:80 wrr persistent 5
  -> 192.168.56.101:80            Route   1      0          0         
  -> 192.168.56.102:80            Route   1      0          0 

```

ipvsadm -Ln --stats 查看集群连接信息

```

IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.56.130:80                   7       64        0    13622        0
  -> 192.168.56.101:80                   2        8        0     1117        0
  -> 192.168.56.102:80                   5       56        0    12505        0

```

浏览器通过虚拟ip访问服务器效果如下：

![lvs_loadbalance.conf](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/lvs_loadbalance.png)

这样使用lvs的一个负载均衡简单集群搭建完成。


### 总结

在本篇文章中，主要对keepalived和lvs进行了基础理论介绍和基本操作的演示，掌握这些基础使用后，我们才能把他们结合起来使用。在下一篇中锤子将会演示一个结合keepalived+lvs+nginx的一个高可用负载均衡集群，欢迎持续关注。

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
