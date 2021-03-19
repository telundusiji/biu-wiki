> Elasticsearch 版本为 7.1.0 ，本文的讲解都是基于该版本  
> 文章中Elasticsearch将使用简称ES代替

### 一、基本概念

#### 文档——Document

ES是面向文档的搜索，文档是ES所有可搜索数据的最小单元。在ES中文档会被序列化成json格式进行保存，每个文档都会有一个Unique ID，这个ID可以有用户在创建文档的时候指定，在用户未指定时则由ES自己生成。 

在ES中一个文档所包含的元数据如下：

* \_index：文档所属索引名称

* \_type：文档所属类型名

* \_id：文档唯一ID

* \_version：文档的版本信息

* \_seq\_no：Shard级别严格递增的顺序号，保证后写入文档的\_seq\_no大于先写入文档的\_seq\_no

* \_primary\_term：主分片发生重分配时递增1，主要用来恢复数据时处理当多个文档的_seq_no一样时的冲突

* \_score：相关性评分，在进行文档搜索时，根据该结果与搜索关键词的相关性进行评分

* \_source：文档的原始JSON数据

ES中一个文档的栗子如下：

```json

{
  "_index" : "user",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "username" : "dream-hammer",
    "message" : "爱做梦的锤子13",
    "test" : "测试"
  }
}

```

> 上面举栗子的文档是直接使用文档id获取的，所有没有相关性评分。  
> 从ES 7.0 版本开始一个索引只能设置一个Type，ES官方说明是：最开始设计和使用类型的初衷是在与 Lucene 不兼容的单个索引中提供多租户，但是在实际使用中，事实证明，使用类型带来的问题比解决的问题还多，所以从7.0开始弃用了接受类型的 API，引入了新的无类型 API，并移除了对 _default_ 映射的支持，从8.0开始将移除接受类型的 API。具体的说明可以参照官方说明[《告别类型。迎接无类型》(点击打开)](https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0)


#### 索引——Index

索引这个词可以有两个理解名词和动词。当作名词使用时，就是指存在的实体，体现的是一种逻辑空间概念。当动词使用时通常代表一种动作，也可以理解为“建立索引”这个动作的简略说法。

在ES中索引（名词）是一类文档的集合，是文档的容器，通常索引是由两部分构成：Mapping和Setting。Mapping
定义该索引包含的文档的数据结构的信息；Setting定义了该索引的数据分布信息

#### 节点——Node

在ES服务中，一个ES实例，本质上是一个Java进程，每个ES实例可以承担不同的工作内容，ES的实例我们可以称之为节点，当一个节点承担某项工作内容时，就可以称这个节点为xxx节点。ES中一个节点可以承担多种功能，每个节点都有一个名字和一个UID，节点名称可以通过配置文件指定，或者在启动ES实例时使用命令参数的方式：`-E node.name='节点名'`来指定，节点的UID是保存在实例的data目录下。

在一个ES的集群中包含着多个ES的节点，往往每个节点所扮演的角色也不尽相同，ES的节点类型主要包含以下几类：

* Master-eligible Node

> 每个节点启动后，默认是一个 Master-eligible 节点，Master-eligible的节点可参加选主流程，成为Master节点，通过配置项 node.master:falase 可以禁用节点的Master-eligible职责，禁止后当前节点就不会参加选主流程

* Master Node

> ES集群中虽然每个节点都保存了集群状态，但是只有Master节点才有修改集群状态的权限，集群状态包括：集群中节点信息、所有索引和其相关的Mapping和Setting信息、分片的路由信息。在集群启动时，第一个启动的Master-eligible节点会将自己选举为主节点。


* Data Node

> 保存数据的节点，负责保存分片数据，对数据扩展有重要作用

* Coordinating Node

> 负责接受Client请求，将请求分发到合适的节点获取响应后，将结果最终汇集在一起，每个节点默认都有Coordinating节点的职责

* Machine Learning Node

> 负责运行机器学习的Job，用来做异常检测

* Ingest Node

> 数据预处理的节点，支持Pipeline管道设置，可以使用Ingest对数据进行过滤、转换等操作

每个ES节点可以承担多个职责，具体配置如下：

* Master-eligible节点配置：node.master，默认值是true

* Data节点配置：node.data，默认值是true

* Ingest节点配置：node.ingest 默认值是true

* Machine Learning节点配置：node.ml 在enable X-pack的前提下默认是true

* Coordinating节点配置：无需配置每个节点都是Coordinating节点

#### 分片——Shard

由于单台机器的存储能力是有限的，所以为了解决数据水平扩展问题ES使用了分片的设计。在这个设计中定义了两种分片类型：主分片（Primary Shard）和副本分片（Replica Shard），主要功能如下

* 主分片 Primary Shard

> 主分片用于解决数据水平拓展问题，在ES中可以将一个索引中的数据切分为多个分片，分布在多台服务器上存储,这样单个索引数据的拓展就不会受到单机存储容量的限制。  
> 同时让搜索和分析等操作分布到多台服务器上去执行，吞吐量和性能也得到提升。  
> 每个主分片都是一个lucene实例，是一个最小工作单元，它承载部分数据，具有建立索引和处理请求的能力。主分片数在创建索引的时候就需要指定，后续不可再修改，在ES 7.0版本之前一个索引的默认主分片是5，从ES 7.0 开始索引的默认主分片数量改为了1

* 副本分片 Replica Shard

> 副本分片用于保证数据服务的高可用。一个索引的多个分片分布在不同的机器上存储，当一个服务器宕机后，就会造成该索引分片数据丢失，因此ES也设计了分片的副本机制。  
> 一个分片可以创建多个副本，副本分片的数量也可以动态调整，副本分片可以在主分片故障时提供备用服务，保证数据安全，同时设置合理个数的副本分片还可以提升搜索的吞吐量和性能。

* 分片设定的问题

主分片数设置过小

> * 后续无法通过增加节点实现水平拓展  
> * 单个分片数据量太大，数据重分配慢

主分片数设置过大

> * 影响搜索的准确性  
> * 单个节点上分片过多，浪费资源和性能

### 二、文档基本操作


#### Create

* 1.POST {index_name}/_doc {data}

> index_name：指定索引名称  
> data：要存储的数据

创建文档时自动生成文档id，若指定的索引不存在，则创建索引

示例：

```json

POST user/_doc
{
  "username" : "dream-hammer",
  "message" : "爱做梦的锤子"
}

```

* 2.PUT {index_name}/_doc/{id}?op_type=create {data}

> index_name：指定索引名称  
> id：指定文档id  
> data：要存储的数据

创建新文档使用指定的文档id，若id已存在，则报错，若指定的索引不存在，则创建索引

示例：

```json

PUT user/_doc/1?op_type=create
{
  "username" : "dream-hammer",
  "message" : "爱做梦的锤子"
}

```

* 3.PUT {index_name}/_create/{id} {data}

> index_name：指定索引名称  
> id：指定文档id  
> data：要存储的数据

创建新文档使用指定的文档id，若id已存在，则报错，若指定的索引不存在，则创建索引

```json

PUT user/_create/1
{
  "username" : "dream-hammer",
  "message" : "爱做梦的锤子"
}

```

#### Read

* 1.GET {index_name}/_doc/{id}

> index_name：指定索引名称  
> id：指定文档id  

获取指定索引下的指定id的文档

示例：

```json

GET user/_doc/1

```


#### Update

* 1.PUT {index_name}/_doc/{id} {data}

> index_name：指定索引名称  
> id：指定文档id  
> data：要更新的数据

先删除指定id的文档数据，再将当前数据写入，指定id文档不存在时，则插入当前数据，<font color=#00bfa5>与创建文档的第二种方式对比，当有op_type=create时，就是创建文档</font>

示例：

```json

PUT user/_create/1
{
  "username-new" : "dream-hammer"
}

```

* 2.POST {index_name}/_update/{id} {data}

> index_name：指定索引名称  
> id：指定文档id  
> data：要更新的数据

将更新数据与指定id的文档原始数据进行合并更新，若指定id文档不存在，则报错

示例：

```json

POST user/_update/1
{
  "doc":{
    "message" : "爱做梦的锤子update",
    "test":"测试"
  }
}

```

#### Delete

* 1.DELETE {index_name}/_doc/{id}

> index_name：指定索引名称  
> id：指定文档id  

删除指定id的文档

示例：

```json

DELETE user/_doc/3

```

#### 批量操作 

* _buik

请求格式如下：

```json

POST _bulk
{operation:{"_index":"{index_name}","_id":"10"}}
{ data}
{operation:{"_index":"{index_name}","_id":"10"}}
{ data}
... ...

```

> operation：操作类型  
> index_name：指定索引名称  
> id：指定文档id 
> data：操作数据，当操作没有不需要数据时，可以不写

一次请求可以指定多个索引进行多种操作，每个操作都有自己的返回码，各个操作之间的成功与否不相互影响

示例：

```json

POST _bulk
{ "index" : { "_index" : "user", "_id" : "1" } }
{ "username" : "爱做梦的锤子1" }
{ "delete" : { "_index" : "user", "_id" : "1" } }
{ "create" : { "_index" : "user", "_id" : "2" } }
{ "username" : "爱做梦的锤子2" }
{ "update" : {"_index" : "user"，"_id" : "1"} }
{ "doc" : {"username" : "爱做梦的锤子update"} }

```

* _mget

请求格式如下：

```json

#方式一

GET /_mget
{
    "docs" : [
        {
            "_index" : {index_name},
            "_id" : {id}
        },
        {
            "_index" : {index_name},
            "_id" : {id}
        },
        ... ...
    ]
}

#方式二

GET {index_name}/_mget
{
    "docs" : [
        {
            "_id" : {id}
        },
        {
            "_id" : {id}
        },
        ... ...
    ]
}


```

> index_name：指定索引名称  
> id：指定文档id

方式一：一次请求get到指定的多个索引的多个id的文档

方式二：一次请求get到一个指定索引下的多个id的文档

示例：

```json

#方式一

GET _mget
{
  "docs":[
    {
      "_index":"user",
      "_id":"1"
    },
        {
      "_index":"movies",
      "_id":"1163"
    }
  ]
}

#方式二

GET user/_mget
{
  "docs":[
    {
      "_id":"1"
    },
    {
      "_id":"2"
    }
  ]
}


```


* _msearch

请求格式如下：

```json

#方式一

POST _msearch
{"index":{index_name}}
{搜索表达式}
{"index":{index_name}}
{搜索表达式}
... ...

#方式二

POST {index_name1}/_msearch
{}
{搜索表达式}
{"index":{index_name2}}
{搜索表达式}
... ...


```

> index_name：指定索引  
> index_name1：指定的默认索引  
> index_name2：指定的特定索引


方式一：一次性请求对多个索引进行查询操作

方式二：一次性请求对多个索引进行查询操作，在请求的Url中包含了一个默认索引，在请求体中如果不指定索引名称，则就使用搜索表达式搜索默认索引


示例：

```json

#方式一

POST _msearch
{"index":"user"}
{"query" : {"match_all" : {}},"size":1}
{"index":"movies"}
{"query" : {"match_all" : {}},"size":2}

#方式二

POST user/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index":"movies"}
{"query" : {"match_all" : {}},"size":2}
{}
{"query" : {"match_all" : {}},"size":1}


```

**总结：**读完本文，对ES的基本概念，就会有个基本认识，同时也可以尝试自己去操作一下ES，掌握ES的基础API

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识，还有我的私密照片

  
> 觉得不错就点个赞叭QAQ
