> 操作系统：CentOS-7.8  
> kubernets版本：1.20.4  
> docker版本：20.10.3

本篇是一个安装教程，包含了docker安装，kubernetes安装以及kube-flannel网络插件的安装，整个安装过程使用的为国内网络环境，在阿里云的镜像服务的加持下，最后得以安装成功，本篇文章仅作为学习kubernetes过程中的一个服务安装参考，如有纰漏，欢迎指正。

## 一、准备工作

### 1.服务器信息

准备三台虚拟机，修改其信息如下

> * 主机名: k8s111  ip:192.168.56.111  作为 k8s的master节点
> * 主机名: k8s112  ip:192.168.56.112 作为k8s的worker节点
> * 主机名: k8s113  ip:192.168.56.113 作为k8s的worker节点

修改三台服务器的 hosts文件  `vim /etc/hosts` 加入以下内容

```shell
192.168.56.111 k8s111
192.168.56.112 k8s112
192.168.56.113 k8s113
```

### 2.服务环境准备

服务准备工作包括以下四项：

* 关闭防火墙
* 禁用SeLinux
* 关闭Swap
* 修改k8s.conf

针对上述四项操作，本人准备了以下的小脚本，可以帮助你快速配置，当然你也可以一项一项的去操作，只要可以达到最终目的即可（ <strong style="color:#f44">注意：三台服务器均需执行这些操作</strong> ）。

操作脚本如下：

```shell
#!/bin/bash

#关闭防火墙，禁用防火墙开机自启动
systemctl stop firewalld
systemctl disable firewalld

# 临时禁用SeLinux，重启失效
setenforce 0
# 修改SeLinux配置，永久禁用
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

# 临时关闭Swap
swapoff -a
# 修改 /etc/fstab 删除或者注释掉swap的挂载，可永久关闭swap
sed -i '/swap/s/^/#/' /etc/fstab

#修改k8s.conf
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 3.卸载docker（可选）

若你使用的服务器之前有安装过docker的，建议卸载原有docker后重新安装以避免遇到不确定的错误。当然如果你确定之前安装的docker是正常的服务，也可以忽略此步骤。

若你需要卸载docker，则你需要卸载的服务包括：docker、docker-common、docker-selinux、docker-engine。卸载命令：`yum remove docker docker-common docker-selinux docker-engine`

### 二、安装服务（docker，kubeadm，kubectl，kubelet）

### 1.yum源准备

安装kubernetes相关服务需要连接谷歌的相关服务下载软件已经镜像包，由于本次教程使用的国内网络，所以我们选用的阿里云的yum源以及镜像服务，我们需要添加 docker-ce 和 kubernetes 两个yum源，添加操作如下，同样本人也将其写成脚本的形式，便于使用。

```shell
#!/bin/bash

# 安装部分依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加docker yum源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 添加kubernetes yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 对新添加的源的软件包信息cache一下
yum makecache
```

### 2.安装docker与kubernetes相关服务

由于我们上一步已经添加了yum源，所以安装这些服务，我们只需要使用yum命令即可安装，安装操作如下：

```shell
#!/bin/bash

#安装docker
yum install -y docker-ce-20.10.3 docker-ce-selinux-20.10.3
#启动docker并设置开机自启动
systemctl enable docker
systemctl start docker

#安装kubernetes相关服务
yum install -y kubelet kubeadm kubectl
# 设置kuberlet为开机自启动
systemctl enable kubelet
systemctl start kubelet

#输出docker和kubernetes的信息
docker version
kubectl version
```

<span style="color:#f60">需要注意的一点是在我们安装完以上服务后，docker是会启动的，但是kubelet服务是不会启动的，master节点的kubelet服务在kubeadm init 成功后才会启动，worker节点的kubelet是在将节点加入集群后才会正常启动，所以在你组装好集群前，不要担心kubelet没有启动的问题。</span>

### 3.对docker配置阿里云镜像加速

参考阿里云官方文档 [https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

## 三、服务启动与集群创建

### 1.使用kubeadm init进行初始化

<span style="color:#f80">该步操作仅在master节点执行，且如下的 `kubeadm init` 若多次执行，则在执行前需要先执行 `kubeadm reset` </span>

在master节点执行如下命令

```shell
# --apiserver-advertise-address指定apiServer的地址
# --image-repository指定镜像下载的仓库，我们这里指定的是阿里云的镜像仓库
# --pod-network-cidr指定pod的CIDR用于限制pod的ip范围，由于我们后面是有flannel网络，所以就直接配置10.244网段，
kubeadm init --apiserver-advertise-address=192.168.56.111   \
--image-repository registry.aliyuncs.com/google_containers  \
--pod-network-cidr=10.244.0.0/16
```

在指向完上述命令后，后面会有一段输出内容如下所示

```shell
To start using your cluster, you need to run the following as a regular user:

#这三条命令在master节点依次执行
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

# 这个地方是说我们需要安装一个网络插件，我们接下来就会安装flannel网络
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

#这个地方是其他worker节点加入该集群所要执行的命令，可以复制出来暂存一下，切记你要复制你自己主机上生成的
kubeadm join 192.168.56.111:6443 --token eud4cb.i6vf9rutybo9ve0u \
    --discovery-token-ca-cert-hash sha256:dc89e4bf471b552e19b0c553910285014d93752398bbac7b6de23e455204c4aa 

```



我们把上述要执行的三条命令放在下面，这个是用于配置kubectl，配置完成后我们就可以使用kubectl进行集群管理了

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 2.安装flannel网络插件

#### 2.1 下载kube-flannel.yml

flannel网络的官方github地址为 [https://github.com/flannel-io/flannel](https://github.com/flannel-io/flannel)

在官方文档中有对使用那个flannel的配置有说明，我们使用的是kubernetes 1.20.4，所以我们按照如下指示来安装flannel网络插件

> For Kubernetes v1.17+ `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

由于flannel插件使用的镜像需要从Google下载，所以我们使用docker上其他用户分享的镜像安装

首先下载 kube-flannel.yml文件到本地，下载地址：https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#### 2.2 拉取flannel镜像到本地

由于我们无法直接下载官方提供的镜像，所以我们去阿里云的镜像服务中使用其他用户转存的flannel镜像，在这里我就使用了阿里云用户公开镜像中的 k8sos/flannel 这个镜像，由于kube-flannel.yml中使用的镜像是 v0.13.1-rc2 版本，所以我们也从阿里云上下载了 k8sos/flannel:v0.13.1-rc2版本

阿里云镜像服务地址：[https://cr.console.aliyun.com/cn-hangzhou/instances/images](https://cr.console.aliyun.com/cn-hangzhou/instances/images) 选择镜像搜索，输入关键字“k8sos/flannel” 进行搜索，即可搜索到。

然后使用docker pull 将镜像拉取下来

```shell
# 在我写这篇文章时该镜像的公网地址如下，如果你在使用时无法通过该公网地址拉取镜像，请去阿里云获取最新镜像地址
docker pull registry.cn-hangzhou.aliyuncs.com/k8sos/flannel:v0.13.1-rc2
```

#### 2.3 修改kube-flannel.yml

镜像拉取本地后，修改kube-flannel.yml将里面使用的官方镜像的名字改为自己拉取的镜像名称

```shell
# 查看镜像拉取到本地后的名称
docker images | grep flannel
# 输出 
# registry.cn-hangzhou.aliyuncs.com/k8sos/flannel      v0.13.1-rc2   dee1cac4dd20   3 weeks ago     64.3MB
# 从上面输出可以看出该镜像在我本地服务器的名称为 registry.cn-hangzhou.aliyuncs.com/k8sos/flannel
```

修改 kube-flannel.yml 将其中的 "image: quay.io/coreos/flannel:v0.13.1-rc2" 修改为 "image: registry.cn-hangzhou.aliyuncs.com/k8sos/flannel:v0.13.1-rc2" 

#### 2.4 应用flannel网络

使用 `kubectl apply` 命令应用网络

```shell
# -f 后面指定kube-flannel.yml的文件路径
kubectl apply -f /k8s/kube-flannel.yml
```

在应用网络之后，我们就可以查看一下，flannel 的pod是否启动成功，在我的服务器上执行如下命令效果如下，代码flannel已经应用成功

```shell
[root@k8s111 k8s]# kubectl get pod -n kube-system | grep flannel
kube-flannel-ds-28mbw            1/1     Running   0          6m55s
```

### 3.添加worker节点到集群

直接在worker节点上执行我们在 kubeadm init 最后得到的那一段命令，格式如下：

```shell
kubeadm join <masterIp>:<masterPort> --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

将worker节点添加到集群成功后，我们就可以使用 kubectl 来查看节点了

```shell
# 查看所有节点
kubectl get nodes

# 得到如下效果，三个节点均运行成功
# NAME     STATUS   ROLES                  AGE   VERSION
# k8s111   Ready    control-plane,master   14m   v1.20.4
# k8s112   Ready    <none>                 24s   v1.20.4
# k8s113   Ready    <none>                 14s   v1.20.4

# 查看pods 集群搭建好，默认有一个kube-system命名空间，kubernetes自己的服务都部署在这个命名空间下
kubectl get pod -n kube-system
```

至此在国内网络环境下的一个kubernetes集群就搭建完成。

## 总结

整个过程所需要使用的所有镜像，可以在操作前先拉取下来，加快效率

```shell

docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.20.4
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.20.4
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.20.4
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.20.4
docker pull registry.cn-hangzhou.aliyuncs.com/k8sos/flannel:v0.13.1-rc2
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.13-0
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.13-0
docker pull registry.aliyuncs.com/google_containers/coredns:1.7.0
docker pull registry.aliyuncs.com/google_containers/pause:3.2

```

在最后我们再来描述一下整个过程

* 准备三台服务器，配置好主机名和IP，关闭防火墙，禁用SeLinux，关闭swap，配置k8s.conf，若服务器之前安装过docker可以将之前的docker卸载，避免安装过程中发生未知的问题
* 在三台服务器上配置docker和kubernetes的yum源，并安装docker和kubeadm、kubectl、kubectl
* 在master节点上进行kubeadm init初始化集群
* 在master节点上安装flannel网络插件
* 在worker节点上执行kubeadm join将其加入集群

经过以上过程，我们就安装了一个自己的kubernetes集群，虽然我已经尽可能的将教程写的易懂，但是作为读者你在安装的过程中可能会出现各种各样的问题，不过只要你去在搜索引擎上搜索，耐心的操作，总可以找到解决办法，祝你顺利！



