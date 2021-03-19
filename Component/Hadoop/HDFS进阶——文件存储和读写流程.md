我们在上一篇[《分布式文件系统HDFS——基础篇》](https://mp.weixin.qq.com/s?__biz=MzIzNjUwMTcyMA==&mid=2247483845&idx=1&sn=01a864831f0029d820bf7d3e71df72b4&chksm=e8d7ae72dfa027645bf8335bc429e70baf3cb719c0527e4c7675762d8592297a52e8030c1210&token=1877824118&lang=zh_CN#rd)中已经已经对HDFS的基本概念和操作进行了介绍，这一次我们将要更深入的了解一下HDFS。你已经知道了HDFS是个分布式文件系统，那么你知道上传到HDFS的文件存在哪里吗？文件是怎么上传上去的？我们读取文件的时候，HDFS又是怎么操作的呢？看完本篇或许你就有一个更清晰的认识了，让我们开始吧！

<font color=#dd2c00>**特别提示：本文所进行的演示都是在hadoop-2.6.0-cdh5.15.1的版本进行的**</font>

### 一、HDFS上数据的存储

我们知道一般的磁盘文件系统都是通过硬盘驱动程序直接和物理硬件磁盘打交道，那HDFS也和它们一样是直接操作物理磁盘吗？答案是否定的，HDFS和一般的磁盘文件系统不一样，HDFS的存储是基于本地磁盘文件系统的。

现在我们来做个小实验，让你更清楚的认识HDFS上的文件是怎么存的，实验步骤如下（单机HDFS、 Block Size：128M、 副本系数：1）：

#### <font color=#00bfa5>1.我们上传一个大小超过128M的文件到HDFS上</font>

```shell

#要上传的文件是JDK的压缩包，大小186M
[root@hdh100 bin]# ll -h /app/jdk/jdk-8u231-linux-x64.tar.gz 
-rw-r--r--. 1 root root 186M Jun  6 13:18 /app/jdk/jdk-8u231-linux-x64.tar.gz

#上传到HDFS上的根目录 ‘/’ 下 
[root@hdh100 bin]# ./hadoop fs -put /app/jdk/jdk-8u231-linux-x64.tar.gz /
20/06/07 15:17:57 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable


```

上传后效果如下：

![上传的jdk](http://file.te-amo.site/images/thumbnail/HDFS%E8%BF%9B%E9%98%B6%E2%80%94%E2%80%94%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E5%92%8C%E8%AF%BB%E5%86%99%E6%B5%81%E7%A8%8B/upload.png)

从图中看出，我们可以看出信息如下：

- a）185.16M 大小的JDK文件被分成了两个Block；
- b）两个Block的大小加起来是整个文件的大小；
- c）两个块都存储在主机hdh100上

#### <font color=#00bfa5>2.找出两个Block的存储位置</font>

> 如果你在安装HDFS的时候在hdfs-site.xml中配置了hadoop.tmp.dir那么你的数据就存储在你配置的目录下，如果你没有配置这个选项数据默认是存放在/tmp/hadoop-root下

```shell

#跳转到数据的存放目录
[root@hdh100 bin]# cd /tmp/hadoop-root/

#根据Block 0的BlockId查找数据块的位置
[root@hdh100 hadoop-root]# find ./ -name *1073741844*
./dfs/data/current/BP-866925568-192.168.56.100-1591426054281/current/finalized/subdir0/subdir0/blk_1073741844_1020.meta
./dfs/data/current/BP-866925568-192.168.56.100-1591426054281/current/finalized/subdir0/subdir0/blk_1073741844

#根据Block 1的BlockId查找数据块的位置
[root@hdh100 hadoop-root]# find ./ -name *1073741845*
./dfs/data/current/BP-866925568-192.168.56.100-1591426054281/current/finalized/subdir0/subdir0/blk_1073741845_1021.meta
./dfs/data/current/BP-866925568-192.168.56.100-1591426054281/current/finalized/subdir0/subdir0/blk_1073741845

#跳转到块所在的目录
[root@hdh100 hadoop-root]# cd ./dfs/data/current/BP-866925568-192.168.56.100-1591426054281/current/finalized/subdir0/subdir0/

#查看两个块的大小，以'.meta'结尾的文件是元数据
[root@hdh100 subdir0]# ll | grep 107374184[45]
-rw-r--r--. 1 root root 134217728 Jun  7 15:18 blk_1073741844
-rw-r--r--. 1 root root   1048583 Jun  7 15:18 blk_1073741844_1020.meta
-rw-r--r--. 1 root root  59933611 Jun  7 15:18 blk_1073741845
-rw-r--r--. 1 root root    468239 Jun  7 15:18 blk_1073741845_1021.meta


```

经过这个操作我们得到以下信息：

- a）HDFS的数据块存储在本地文件系统；
- b）两个数据块的大小和web看到的大小是一致的

那么HDFS是不是就是简单的把我们上传的文件给切分一下放置在这里了，它有做其他操作吗？我们继续下一步

#### <font color=#00bfa5>3.直接合并两个文件，验证我们想法</font>

> 合并后无非是两个结果：1.合并后的文件可以直接使用，则说明HDFS对上传的文件是直接切分，对数据块无特殊操作；2.合并后文件无法使用，则说明HDFS对拆分后文件进行了特殊处理

```

#将两个块合并
[root@hdh100 subdir0]# cat blk_1073741844 >> /test/jdk.tag.gz
[root@hdh100 subdir0]# cat blk_1073741845 >> /test/jdk.tag.gz

#跳转到test目录，test目录是之前创建的
[root@hdh100 subdir0]# cd /test/

#尝试解压文件，发现解压成功
[root@hdh100 test]# tar -xzvf jdk.tag.gz

```

经过上面操作，我们仅仅是将两个块进行了合并，然后尝试解压发现竟然解压成功了，这也证明了我们猜想HDFS 的文件拆分就是简单拆分，拆分的块合并后就直接是原文件。

<font color=#ff6d00>结论:</font>从上面的实验我们可以得出以下结论：a）HDFS的文件存储是基于本地文件系统的，也就是HDFS上的文件被切分后的块是使用本地文件系统进行存储；b）上传文件到HDFS后，HDFS对文件的拆分属于基本拆分，没有对拆分后的块进行特殊处理

### 二、HDFS写数据流程

经过上一步我们了解了HDFS上的文件存储，现在我们开始看看HDFS创建一个文件的流程是什么。

为了帮助大家理解，锤子画了一个图，咱们跟着图来看

![写流程](http://file.te-amo.site/images/thumbnail/HDFS%E8%BF%9B%E9%98%B6%E2%80%94%E2%80%94%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E5%92%8C%E8%AF%BB%E5%86%99%E6%B5%81%E7%A8%8B/write.png)


结合图，咱们来描述一下流程

* 1.客户端调用DistributedFileSystem的create方法

* 2.DistributedFileSystem远程RPC调用NameNode，请求在文件系统的命名空间中创建一个文件

* 3.NameNode进行相应检测，包括校验文件是否已经存在，权限检验等，校验通过则创建一条新文件记录，否则抛出未校验通过的相应异常，根据最后处理结果，对最请求端做出响应

* 4.DistributedFileSystem跟NameNode交互接收到成功信息后，就会根据NameNode给出的信息创建一个FSDataOutputStream。在FSDataOutputStream中持有一个final的DFSOutputStream引用，这个DFSOutputStream负责后续的NameNode和DataNode通信

* 5.输出流准备好后，客户端就开始向输出流写数据

* 6.此时DataStreamer就会向NameNode发出请求创建一个新的Block

* 7.NameNode接收请求后经过处理，创建一个新的BlockId，并且将该Block应该存储的位置返回给DataStream

> DataStreamer是DFSOutputStream的一个内部类，且DFSOutputStream中还持有一个DataStreamer实例的引用。DataStreamer类负责向管道中的数据节点发送数据包，它会向NameNode申请新的BlockId并检索Block的位置，并开始将数据包流传输到Datanodes的管道。

* 8.DateStreamer在接收到Block的位置后，会通知需要存储该Block副本的相应DataNode，让这一组DataNode形成一个管道。DataStreamer 就开始将数据包流传输到Datanodes的管道，每一个DataNode节点接收到数据包流后存储完成后就转发给下一个节点，下一个节点存放之后就继续往下传递，直到传递到最后一个节点为止

* 9.放进DataNodes管道的这些数据包每个都有一个相关的序列号，当一个块的所有数据包都被发送出去，并且每个包的ack（存储成功应答）都被接收到时，DataStreamer就关闭当前块了。（如果该文件有多个块，后面就是按照6，7，8，9步骤重复）

* 10.文件数据全部写入完毕后，客户端发出关闭流请求，待NameNode确认所有数据写入完成后，就把成功信息返回给客户端

### 三、HDFS读数据流程

同样读数据流程我们也画了一个图

![读流程](http://file.te-amo.site/images/thumbnail/HDFS%E8%BF%9B%E9%98%B6%E2%80%94%E2%80%94%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E5%92%8C%E8%AF%BB%E5%86%99%E6%B5%81%E7%A8%8B/read.png)

我们描述一下流程

* 1.客户端调用DistributedFileSystem的open方法

* 2.DistributedFileSystem远程RPC调用NameNode，请求读取指定文件

* 3.NameNode在接收请求后，查询该文件的元数据信息，包括文件Block起始块位置，每个块副本的DataNode地址，并将DataNode地址按照与当前请求客户端的距离远近进行排序，并将信息返回给请求端

* 4.DistributedFileSystem跟NameNode交互成功接收到该文件的元数据信息后，就会根据NameNode给出的信息创建一个FSDataInputStream。在FSDataInputStream中也持有一个DFSInputStream引用，DFSInputStream负责后续跟DataNode和NameNode通信

* 5.客户端对FSDataInputStream发出read请求

* 6.DFSInputStream连接第一个块的所有副本中与当前客户端最近的一个DataNode进行数据读取，当前块读取完成后继续下一个块，同样寻找最佳的DataNode读取，直到所有块读取完成

* 7.文件所有块读取完成后，客户端关闭流

### 四、一致模型

文件系统的一致性模型是描述文件读写的数据可见性的。HDFS的一致模型中主要有以下特点

* 新建文件，它在文件系统的命名空间中立即可见

```java

Path path = new Path("/dream-hammer.txt");
fileSystem.create(path);
fileSystem.exists(path) //结果为true

```

* 新建文件的写入内容不保证立即可见

```java

Path path = new Path("/dream-hammer.txt");
OutputStram out = fileSystem.create(p);
out.write("content");
out.flush();
fileSystem.getFileStatus(path).getLen() //可能为0，即使数据已经写入

```

* 当前写入的Block对其他Reader不可见

> 即当前客户端正在写入文件的某个Block，在写的过程中，其他客户端是看不到该Block的

hflush()方法：强行刷新缓存数据到DataNode，hflush刷新成功后，HDFS保证文件到目前为止对所有新的Reader而言都是可见的且写入数据都已经到达所有DataNode的写入管道。<font color=#dd2c00>但是hflush不保证DataNode已经将数据写到磁盘，仅确保数据在DataNode的内存中。</font>

hsync()方法：也是一种强行刷新缓存数据到DataNode，在hsync()刷新成功后，客户端所有的数据都发送到每个DataNode上，并且每个DataNode上的每个副本都完成了POSIX中fsync的调用，即操作系统已将数据存储到磁盘上

调用hflush()和hsync()都会增加HDFS的负载，但是如果不使用强制缓存数据刷新，就会有数据块丢失的风险，对于大部分应用来说因为客户端或者系统系统故障丢失数据是不可接受的，所以实际生产中我们要根据实际场景来适当调用，在数据安全和速度上进行平衡


> 文章欢迎转载，转载请注明出处，个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片
