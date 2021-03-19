
### 一、概述

#### 简介

Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎。它是基于Apache Lucene开发的，开发语言为Java，使用Apache 开源协议进行开源。Elasticsearch的特点是：分布式存储，近实时数据搜索和分析，稳定，可靠，快速，且安装使用方便。

> 同类型项目：  
> Apache Solr：一个开源的搜索服务，使用Java语言开发，主要基于HTTP和Apache Lucene实现的  
> Splunk：一个托管的日志文件管理工具，可收集、索引和利用所有应用程序、服务器和设备生成的快速移动型计算机数据

#### Apache Lucene

Apache Lucene 创建于1999年，2005年成为Apache的顶级开源项目。Lucene 的创始人是 Doug Cutting，他也是 Hadoop 的作者。

Lucene 的目的是为软件开发人员提供一个简单易用的工具包，以方便的在目标系统中实现全文检索的功能，或者是以此为基础建立起完整的全文检索引擎。有以下特点：

* 基于Java语言开发，属于搜索引擎类库

* 高性能、易拓展

* 局限性：

	* 1、只能基于Java语言；
	
	* 2、类库接口学习曲线陡峭；
	
	* 3、原生不支持水平拓展


#### Elasticsearch

Elasticsearch的核心功能：

* 海量数据的分布式存储和集群管理

* 高性能的近实时搜索

* 海量数据的近实时分析

Elasticsearch 有以下特点：

* 支持分布式，可水平拓展

* 可以被多语言调用，降低全文检索的学习曲线

* 集群规模可以从单个节点拓展至数百个节点

* 在服务和数据两个维度都支持高可用且水平易扩展

* 支持不同的节点类型

* 提供 RESTful API 和 Transport API（官方建议使用Rest API，从Elasticsearch 8.0 版本开始 Transport API将会被移除 ）

* 支持jdbc(java 数据库连接)和odbc(开放数据库连接)


#### Elasticsearch大版本更新新特性

<font color=#00bfa5>5.x 新特性</font>

* Lucene 6.x

* 默认评分机制从TF-IDF改为BM25

* 内部引擎级别移除了避免同一文档并发更新的竞争锁，性能提升15%-20%

* Instant aggregation,支持分片上聚合的缓存

* 新增 Painless 脚本引擎

* 新增 Completion Suggested 搜索提示功能

* 新增原生的Java rest客户端

* 新增Ingest Node，数据预处理节点

* 新增 Profile API 用于查询优化


<font color=#00bfa5>6.x 新特性</font>

* Lucene 7.x

* 新增跨集群复制（Cross Cluster Replication）

* 新增索引生命周期管理

* 新增Sql支持

* 主要版本之间的升级和迁移更为简化

* 全新的基于操作的数据复制框架，加快恢复数据

* 有效存储稀疏字段的新方法，降低了存储成本

* 在索引的过程中进行排序，可提高排序的查询性能

<font color=#00bfa5>7.x 新特性</font>

* Lucene 8.0

* 废除单个索引下多个Type的支持

* 从7.1开始Security免费使用

* ECK——基于 Kubernetes Operator 模式，扩展了 Kubernetes 的编排功能，支持在 Kubernetes 上设置和管理 Elasticsearch 和 Kibana

* 新的集群协调管理

* 完整的 High Level Rest API

* Script Score Query，使用脚本为返回的文档提供自定义分数

* 主分片（Primary Shard）默认数从5改为1

* 更快的Top K


### 二、Elastic家族其他成员

Elastic Stack生态圈

![Elastic Stack生态圈](http://file.te-amo.site/images/thumbnail/%E8%AE%A4%E8%AF%86Elasticsearch/es.png)

#### Logstash

Logstash 是一个开源的服务端数据处理管道，支持从不同的数据来源采集数据、转换数据，并将数据发送到不同的数据存储库中。Logstash 诞生于2009年，它的作者是 Jordan Sisel，最初 Logstash 是用来做日志采集和处理的，后来随着不断改进 Logstash 的功能得到了越来越多的拓展，在2013年 Elasticsearch 收购了 Logstash ，Logstash 也就成为了 Elastic 家族的一员


Logstash的特点：


* 实时解析和转换数据

	* 对事件字段执行常规转换，重命名，删除，替换和修改事件中的字段，例如：从Ip地址解析地理坐标、将PII（personally identifiable information 个人验证信息）数据匿名化，排除敏感字段
	
* 可拓展

	* 200多个插件（日志，数据库，Arcsigh，Netflow）
	
* 可靠性和安全性

	* Logstash 通过持久化队列来保证至少将运行中的事件送达一次
	
	* 数据传输和加密
	
#### kibana

Kibana是一个为Elasticsearch设计的开源的数据分析可视化平台，可用来搜索、查看存储在Elasticsearch中的数据并与之交互。

Kibana特点：

* 自由地选择如何呈现自己的数据

* 可以简单直观的构建可视化

* 配置简单、接口灵活，可以更便捷分享数据

* 与Elasticsearch无缝对接，可视化的与Elasticsearch REST API交互

#### Beats

Beats是开源的轻量级数据采集器，这些采集器作为安装在不同服务器上的代理，用于收集日志或指标，采用Go语言开发。

Beats可以使用多种方式采集数据，主要包括如下：

* Filebeat：用于收集和传送日志文件

* Packetbeat：捕获服务器之间的网络流量，可用于应用程序和性能监视。


* Metricbeat：收集并报告系统和平台的各种系统级度量

* Heartbeat：探测服务以检查它们是否可访问

* Auditbeat：用于审核Linux服务器上的用户和进程活动

* Winlogbeat：专门为收集Windows事件日志而设计

* Functionbeat：为监视云环境而设计，收集数据并将其发送到ELK堆栈


#### X-Pack

X-Pack 是一个 Elastic Stack 的扩展，将安全，警报，监视，报告和图形功能包含在一个易于安装的软件包中。使用X-pack通过Kibana可以实时查看集群的健康和性能，以及分析过去的集群、索引和节点度量，还可以监视Kibana本身性能。6.3之前 X-Pack 以插件方式安装，在 X-Pack 开源后，提供了两个版本：白金版和黄金版，部分 X-Pack 功能免费使用，从6.8和7.1开始 Security 功能免费使用


三、Elasticsearch发展


* 2004年Shay Banon基于Lucene开发Compass

* 2010年Shay Banon重写Compass，取名字为ElasticSearch

* 2010年2月第一次发布0.4版本

* 2012年Elasticsearch商业公司成立

* 2014年1月发布1.0版本

* 2014年10月发布2.0版本

* 2015年3月，Elasticsearch 收购 Elastic Cloud，开始提供 Cloud 服务。

* 2015年3月，收购PacketBeat

* 2016年9月，收购PreAlert-Machine Learning异常检测

* 2016年10月，Elasticsearch发布5.0版本

* 2017年6月，收购Opbeat进军APM

* 2017年10月，Elasticsearch 发布6.0版本

* 2017年11月，收购Saas厂商Swiftype，提供网站和App搜索

* 2018年X-pack开源

* 2019年4月Elasticsearch发布7.0版本


> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ