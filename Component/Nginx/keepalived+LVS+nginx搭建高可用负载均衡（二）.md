
> 操作系统：CentOS-7.8  
> keepalived版本：2.0.20  
> nginx版本：1.18.0

本篇是[《keepalived+LVS+nginx搭建高可用负载均衡》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483920&idx=1&sn=fa10a7f847883d93174584fd91164c49&chksm=e8d7ada7dfa024b1f28947549c33ac1d8d602aa32a1756d5ee0b1c135ab1c5bd78b0c4e95e7b&token=400958876&lang=zh_CN#rd)的第二篇，上一篇主要是介绍了这些组件和基础知识，以及基础演示，本篇将把这三个组件进行结合做一个演示，对这三个组件不熟悉的朋友请先看上一篇文章[《keepalived+LVS+nginx搭建高可用负载均衡（一）》传送门](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483920&idx=1&sn=fa10a7f847883d93174584fd91164c49&chksm=e8d7ada7dfa024b1f28947549c33ac1d8d602aa32a1756d5ee0b1c135ab1c5bd78b0c4e95e7b&token=400958876&lang=zh_CN#rd)

## 一、准备工作

### 1.环境说明

该演示总共需要4台服务机器（可使用虚拟机），分别取名字：LVS1，LVS2，Server1、Server2。每台机器说明如下：

* LVS1：LVS复杂均衡调度器，安装keepalived，配置LVS调度策略。真实IP：192.168.56.121，keepalived主实例虚拟IP：192.168.56.131，keepalived备用实例虚拟IP：192.168.56.132

* LVS2：LVS复杂均衡调度器，安装keepalived，配置LVS调度策略，真实IP：192.168.56.122，keepalived主实例虚拟IP：192.168.56.132，keepalived备用实例虚拟IP：192.168.56.131

* Server1：服务器集群服务器，安装nginx，keepalived。真实IP：192.168.56.101，keepalived主实例虚拟IP：192.168.56.121，keepalived备用实例虚拟IP：192.168.56.122，回环虚拟IP（对外隐藏）：192.168.56.131、192.168.56.132

* Server2：服务器集群服务器，安装nginx，keepalived。真实IP：192.168.56.102，keepalived主实例虚拟IP：192.168.56.122，keepalived备用实例虚拟IP：192.168.56.121，回环虚拟IP（对外隐藏）：192.168.56.131、192.168.56.132

注意：两台LVS1服务器构成双主热备作为负载均衡，两台Server作为真实服务器提供nginx的web服务采用双主热备。（更深层次可以将nginx集群继续作为负载均衡集群，搭建多级负载均衡，在这里就不进行这种拓展了）


### 2.服务主机关系图

如下图即为本次演示的一个主机关系图，从图中可以清晰看出四台服务器的关系。

![lvs_keepalive_nginx](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/lvs_keepalive_nginx.png)


## 二、配置与操作

### 1.准备工作

#### nginx活跃检测脚本

由于nginx和keepalived是两个独立的服务，默认情况下keepalived仅仅是检测集群中节点的keepalived服务是否存活来判断主机是否正常服务。而在keepalived服务正常，而nginx服务down掉了，这种情况下，keepalived服务是不会判定当前服务器宕机，也就不会进行虚拟IP绑定转移，所以此时使用虚拟IP访问服务会访问到nginx服务不可用的服务器，为了解决这种情况，我们需要配置keepalived使其可以坚持nginx状态来决定是否进行虚拟IP绑定转移。


nginx服务活跃检测脚本如下，建议将该脚本放在keepalived的配置目录便于配置使用。

```shell

#!/bin/bash

#判断当前服务器是否有nginx的进程
if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
	   #若没有则尝试启动nginx
        /usr/local/nginx/sbin/nginx
        sleep 5
        #继续判断当前服务器是否有nginx进程
        if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
        		#若仍然没有nginx进程，则停止当前节点的keepalived
                systemctl stop keepalived
        fi
fi

```

脚本写好先放着，后面再配置使用

#### 配置真实服务器的回环虚拟IP


由于我们演示lvs是使用DR的工作模式，所以在server1和server2上需要将虚拟IP：192.168.56.131和192.168.56.132绑定在两台主机的lo回环接口上，具体理论请参加上一篇文章[《keepalived+LVS+nginx搭建高可用负载均衡（一）》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483920&idx=1&sn=fa10a7f847883d93174584fd91164c49&chksm=e8d7ada7dfa024b1f28947549c33ac1d8d602aa32a1756d5ee0b1c135ab1c5bd78b0c4e95e7b&token=400958876&lang=zh_CN#rd)


server1和server分别在 /etc/sysconfig/network-scripts目录下，将ifcfg-lo复制两份

```
cd /etc/sysconfig/network-scripts

cp ifcfg-lo ifcfg-lo:1

cp ifcfg-lo ifcfg-lo:2

```

然后编辑网络配置文件

```

#ifcfg-lo:1配置如下
DEVICE=lo:1
IPADDR=192.168.56.131
NETMASK=255.255.255.255
NETWORK=127.0.0.0
# If you're having problems with gated making 127.0.0.0/8 a martian,
# you can change this to something else (255.255.255.255, for example)
BROADCAST=127.255.255.255
ONBOOT=yes
NAME=loopback

#ifcfg-lo:2配置如下
DEVICE=lo:2
IPADDR=192.168.56.132
NETMASK=255.255.255.255
NETWORK=127.0.0.0
# If you're having problems with gated making 127.0.0.0/8 a martian,
# you can change this to something else (255.255.255.255, for example)
BROADCAST=127.255.255.255
ONBOOT=yes
NAME=loopback

```

修改完成后，重启网络服务：`systemctl restart network.service`

随后使用：ifconfig lo:1 和 ifconfig lo:2 两个命令即可查看配置的虚拟IP

之后Server1和Server2增加路由（单次生效）：route add -host 192.168.56.131 dev lo:1 和 route add -host 192.168.56.132 dev lo:2

### 2.nginx+keepalive双主热备配置

首先配置Server1的keepalived，我的配置文件在/etc/keepalived/keepalived.conf，该配置文件我仅说明新增的一些属性含义，没有说明的属性含义，请参加上一篇文章[《keepalived+LVS+nginx搭建高可用负载均衡（一）》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483920&idx=1&sn=fa10a7f847883d93174584fd91164c49&chksm=e8d7ada7dfa024b1f28947549c33ac1d8d602aa32a1756d5ee0b1c135ab1c5bd78b0c4e95e7b&token=400958876&lang=zh_CN#rd)。Server1的配置如下

```

! Configuration File for keepalived

global_defs {
   router_id keep_101
}

#nginx活跃检测脚本配置
vrrp_script check_nginx_alive{
    #脚本所在位置
    script "/etc/keepalived/check_nginx_auto_pull.sh"
    #脚本执行间隔
    interval 3
    #脚本执行成功后权重增加1，执行失败权重不变，当配置为负数时，脚本执行失败权重减少1，执行成功权重不变
    #主从权重增加或减少改变时，当主节点权重<备用节点权重时，会发生主节点失败，备用节点活跃
    weight 1
}

#使用虚拟Ip：192.168.56.111的主实例
vrrp_instance VI_1 {
    state MASTER
    interface enp0s8
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        #两个实例使用了不同的密码
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.56.111
    }
    #引用上方配置的脚本，检测nginx是否活跃
    track_script{
        check_nginx_alive
    }
}

#使用虚拟Ip：192.168.56.112的备用实例
vrrp_instance VI_2 {
    state BACKUP
    interface enp0s8
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        #两个实例使用了不同的密码
        auth_pass 2222
    }
    virtual_ipaddress {
        192.168.56.112
    }
    #引用上方配置的脚本，检测nginx是否活跃
    track_script{
        check_nginx_alive
    }
}


```

Server2的keepalived配置与Server1的keepalived配置类似，不同点就是主备实例调换，Server2的配置如下

```

! Configuration File for keepalived

global_defs {
   router_id keep_102
}


vrrp_script check_nginx_alive{
    script "/etc/keepalived/check_nginx_auto_pull.sh"
    interval 3
    weight 1
}

#使用虚拟Ip：192.168.56.111的备用实例
vrrp_instance VI_1 {
    state BACKUP
    interface enp0s8
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.56.111
    }
    track_script{
        check_nginx_alive
    }
}

#使用虚拟Ip：192.168.56.112的主实例
vrrp_instance VI_2 {
    state MASTER
    interface enp0s8
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 2222
    }
    virtual_ipaddress {
        192.168.56.112
    }
    track_script{
        check_nginx_alive
    }
}

```

配置完成后启动keepalived和nginx，这样配置完成后就保证了nginx的高可用，同时nginx可以作为负载均衡继续给更多的集群进行负载均衡。

### 3.LVS+keepalived双主热备配置

首先配置LVS1的keepalived，keepalived的配置中集成了对lvs的管理，所以在使用LVS+keepalived的时候，不用再使用上一篇文章讲的用命令配置了，我们直接将lvs的配置写在keepalived的配置文件中即可。

我仅说明新增的一些属性含义，没有说明的属性含义，请参加上一篇文章[《keepalived+LVS+nginx搭建高可用负载均衡（一）》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483920&idx=1&sn=fa10a7f847883d93174584fd91164c49&chksm=e8d7ada7dfa024b1f28947549c33ac1d8d602aa32a1756d5ee0b1c135ab1c5bd78b0c4e95e7b&token=400958876&lang=zh_CN#rd)

LVS1服务器的keepalived配置如下：

```

! Configuration File for keepalived

global_defs {
   router_id keep_101
}


vrrp_instance VI_1 {
    state MASTER
    interface enp0s8
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 3333
    }
    virtual_ipaddress {
        192.168.56.131
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface enp0s8
    virtual_router_id 52
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 4444
    }
    virtual_ipaddress {
        192.168.56.132
    }
}

#lvs虚拟服务器，虚拟IP 端口号
virtual_server 192.168.56.131 80 {
    
    delay_loop 6
    #负载均衡策略，wrr代表加权轮询
    lb_algo wrr
    #lvs工作模式，DR模式
    lb_kind DR
    #连接持久化，超时时间，单位：秒
    persistence_timeout 5
    #连接类型，TCP
    protocol TCP

    #真实服务器信息，真实服务器IP 端口
    real_server 192.168.56.111 80 {
    	   #当前服务器的权重
        weight 1
        #TCP连接的检查配置
        TCP_CHECK {
        		#检查端口为 80
                connect_port 80
                #检查连接超时时间，单位：秒
                connect_timeout 2
                #超时后重试次数
                nb_get_retry 2
                #被认为宕机后，间隔多久继续重试，单位：秒
                delay_before_retry 3
        }
    }

    real_server 192.168.56.112 80 {
        weight 1
        TCP_CHECK {
                connect_port 80
                connect_timeout 2
                nb_get_retry 2
                delay_before_retry 3

        }
    }

}

virtual_server 192.168.56.132 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 5
    protocol TCP

    real_server 192.168.56.111 80 {
        weight 1
        TCP_CHECK {
                connect_port 80
                connect_timeout 2
                nb_get_retry 2
                delay_before_retry 3
        }
    }

    real_server 192.168.56.112 80 {
        weight 1
        TCP_CHECK {
                connect_port 80
                connect_timeout 2
                nb_get_retry 2
                delay_before_retry 3

        }
    }

}

```

LVS2的keepalived的配置与LVS1的不同之处仅在于keepalived的实例配置不同，lvs虚拟服务配置相同，所以我在下面仅列出不同的配置，相同的配置就不在写出来，LVS2服务器的keepalived的不同的配置如下：

```

! Configuration File for keepalived

global_defs {
   router_id keep_101
}

#主要是连个keepalived实例的主备配置与lvs1中配置相反

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s8
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 3333
    }
    virtual_ipaddress {
        192.168.56.131
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface enp0s8
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 4444
    }
    virtual_ipaddress {
        192.168.56.132
    }
}

#下面是lvs虚拟服务器配置，这里就不在贴代码了

```

经过以上配置后，启动lvs1和lvs2两台服务器的keepalived


### 4.验证

通过浏览器访问效果如下

![浏览器访问nginx](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/lvs_keepalive_nginx_page.png)

查看两条lvs上的虚拟服务状态

![ipvsadm -Ln查看信息](http://file.te-amo.site/images/thumbnail/keepalived+LVS+nginx%E6%90%AD%E5%BB%BA%E9%AB%98%E5%8F%AF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/lvs_keepalive_nginx_ipvsadm.png)

### 总结

到此一个实验keepalived+nginx+lvs搭建的高可用负载服务就完成了，在本演示案例中是将nginx作为web服务来使用的，其实在生成中可以将nginx作为层负载均衡是用，这样这就是一个基于lvs四层负载+nginx七层负载的高可用负载均衡集群，更多的搭配架构还需在实际使用时多探索，本演示案例仅仅是提供了一个简单思路，便于在学习的时候，可以快速理解


> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
