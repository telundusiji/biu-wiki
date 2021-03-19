> 操作系统：CentOS-7.8  
> 单机Hadoop版本：2.8.5  
> Hive版本：2.3.6

本文是Hive学习的基础篇，包含Hive的部分基础理论知识如：hive介绍，Hive应用场景，Hive的文件存储方式，Hive的基本操作，Hive的表类型，Hive中数据分区，以及Hive的自定义函数等，并配有演示代码帮助大家学习，文中代码地址：[https://github.com/telundusiji/dream-hammer/tree/master/module-7](https://github.com/telundusiji/dream-hammer/tree/master/module-7)

### 一、Hive理论概述

#### 什么是Hive

Hive是一个构建在Hadoop之上的数据仓库工具，用来进行数据提取、转化、加载，同时提供类sql的查询HiveQL可以对数据进行分析处理。

Hive可以将结构化的数据文件映射为一张数据库表，并向用户提供完整的SQL查询功能，Hive SQL是将SQL语句转换为MapReduce任务运行，在不用编写MapReduce程序的情况下可以方便地利用SQL语言进行数据查询、汇总和分析，同时对于更复杂的数据分析Hive也支持编写插件对其进行拓展。

Hive SQL并非标准SQL，它在支持了绝大多数标准SQL语句的基础上，还提供了数据提取、转化、加载，以及用来存储、查询和分析存储在Hadoop中的大规模数据集的特有语句。Hive支持UDF，可以实现对MR函数的定制，为数据操作提供了良好的伸缩性和可扩展性

#### Hive与关系型数据库区别

##### 1.应用场景

Hive是数据仓库，是为海量数据的离线分析设计的，而关系型数据库，是为实时业务设计的。Hive不支持OLTP(联机事务处理)所需的关键功能ACID，更接近于OLAP(联机分析技术)，适合离线处理大数据集，而关系型数据库的关键功能就是ACID，它更注重处理的实时性，适合数据量较少实时性要求高的应用场景。

##### 2.可扩展性

Hive中的数据存储在HDFS，Metastore元数据一般存储在独立的关系型数据库中，相对于关系型数据库的数据一般存储则是服务器本地的文件系统，因此在扩展性上Hive更具有优势（HDFS易扩展），而关系型数据库则由于本地文件系统和ACID语义的严格限制，扩展难度较大。

##### 3.读写模式

Hive为读时模式，数据的验证则是在查询时进行的，读时模式使数据的加载非常迅速，数据的加载仅是文件复制或移动。关系型数据库为写时模式，数据在写入数据库时会对照模式检查，写时模式数据库可以对列进行索引，有利于提升查询性能

##### 4.数据更新

由于数仓的内容是读多写少的，Hive中不支持对数据进行改写，所有数据都是在加载的时候确定好的，而关系型数据库支持更新数据，关系型数据库中的数据通常是需要经常进行修改更新的

##### 5.索引

Hive和关系型数据库都支持索引，但两者的索引设计并不相同。Hive中不支持主键或者外键，而关系型数据库支持主/外键。Hive提供了有限的索引功能，可以为一些列建立索引，一张表的索引数据存储在另外一张表中，这样的索引仅仅可以提升一些操作的效率，但并不能降低数据的访问延迟，而在关系型数据库中通常会针对一个或者几个列建立索引，对于少量的特定条件的数据的访问，关系型数据库可以有很高的效率，较低的延迟

##### 6.计算模型

Hive使用的计算模型支持多种，包括：MapReduce，spark，tez等，而关系型数据库通常都是使用自己设计的Executor计算模型

#### Hive数据存储格式

Hive中的数据文件的存储格式分为TextFile、SequenceFile和ORCFile三种

##### 1.TextFile

TextFile是Hive默认数据存储格式，可以通过制定的分隔符就可以对其进行解析，主要有CSV、文本类型等文件，可结合Gzip、Bzip2、Snappy等使用（系统自动检查，执行查询时自动解压），但使用这种方式，hive不会对数据进行切分，从而无法对数据进行并行操作。

##### 2.SequenceFile

SequenceFile是一种二进制文件，是由Hadoop API 提供的，它将数据以<key,value>的形式序列化到文件中，但是SequenceFile文件并不按照其存储的Key进行排序存储

SequenceFile支持文件的压缩和分片，它的压缩分为记录压缩和块压缩两种方式。Hive 中的SequenceFile 是继承自Hadoop API 的SequenceFile，与原始的SequenceFile不同的是Hive中的SequenceFile的key为空，使用value 存放实际的值， 这样是为了避免MR 在运行map 阶段的排序过程。

与TextFile相比，SequenceFile在存储上支持压缩和分片在存储的开销上优于TextFile，但是SequenceFile和TextFile的存储格式都还是基于行存储的，所以它不太满足能快速的查询响应时间的要求，当查询仅仅针对所有列中的少数几列时，它无法跳过不需要的列，直接定位到所需列，这对查询性能会有一定影响，同时在存储空间利用上，由于数据表中包含不同类型，不同数据值的列，行存储也不易获得一个较高的压缩比

##### 3.RCFile

RCFILE是基于行列混合思想的一种存储文件，在RCFile中ORCFile是在Hive中更加常使用的一种存储格式，ORCFile是对RCFile进行的优化，在一定程度上扩展了RCFile。ORCFile支持分片可以按列查询，也不易查看，由于ORCFile是列式存储格式，所以其更加适合大数据查询的场景。

下面我们详细了解一下ORCFile的设计，首先看下面一张图示一个ORCFile的结构信息

![测试表](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Hive%E5%9F%BA%E7%A1%80/orcfile.png)

ORCFile 是一个自描述的文件，在ORC文件中它的mate信息放到了文件的尾部，图中左侧是一个OrcFile我们可以看出，它的尾部是一个File Footer，它的元数据信息就放在这个File Footer当中。一个ORCFile包含多个Stripe，每个Strip是一个分片可以被一个mapper读取，每个Stripe包含三部分内容：列的索引，列的数据和Stripe Footer(元数据)。

ORCFile支持分区读，所以会为每个分区构建索引，主要分为三个级别索引：

* File level级别索引，主要保存的是File Footer中的信息，包括数据文件的mate信息、数据索引信息以及各列数据的范围信息，可以定位到Stripe

* Stripe level级别索引，主要保存的是Stripe Footer中的信息，包括该分区的索引信息和该分区数据范围信息

* Row-Group level级别索引，它是ORCFile最小的索引单位，默认每个Row-Group索引是由10000条数据组成的，当数据确认到某个Row-Group时，只需要扫描当前Row-Group的1万条数据即可

ORCFile的优点：

* 查询时只需要读取查询所涉及的列，可以降低IO销毁，同时索引中会保存每一列的统计信息，实现部分谓词下推，可以更快检索数据

* 每列数据类型一致，可以针对不同数据类型采用不同压缩算法，能够获得一个较高的压缩比和压缩效率

* 列式存储假设数据不会发生变化，支持分片，流式读取，可以更好的适用于分布式文件存储的特性


ORCFile与Parquet简单对比

* 两者都是Apache的顶级项目

* Parquet不支持ACID，不支持更新，ORCFile支持有限ACID和更新

* Parquet的压缩能力比较强，ORCFile的查询效率比较高

#### HQL到MR的转换过程

Hive 将SQL转换成MR任务主流有一下流程：

* Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree
* 遍历AST Tree，抽象出查询的基本组成单元QueryBlock
* 遍历QueryBlock，翻译为执行操作树OperatorTree
* 逻辑层优化器进行OperatorTree变换，合并不必要的ReduceSinkOperator，减少shuffle数据量
* 遍历OperatorTree，翻译为MapReduce任务
* 物理层优化器进行MapReduce任务的变换，生成最终的执行计划

### 二、Hive基础与操作

#### 数据类型

基础数据类型

* tinyint	-2^7 ~ 2^7-1

* smallint	-2^15 ~ 2^15-1

* int		-2^31 ~ 2^31-1

* bigint	-2^63 ~ 2^63-1

* boolean

* float	单精度浮点

* double	双精度浮点

* string	字符串

* binary	二进制类型

* timestamp	时间戳

* decimal	表示任意精度的不可修改的十进制数字	

* char	字符

* varchar	近似字符串类型，长度上只允许在1-65355之间

* date	日期类型


复杂数据类型：array，map，struct

* array：数组

* map：k-v映射

* struct：复杂数据类型

下面演示一个例子来使用者三种数据类型

```sql

#创建表data_table包含5个字段，其中hobby为array类型，score为map类型，info为struct类型
create table data_table(
	id string,
	name string,
	hobby array<string>,
	score map<string,int>,
	info struct<item:string,level:string>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':';

#加载数据
#样例数据如下：
#99b28506|张三|游泳,唱歌|语文:90,数学:90|CET4:通过,CET6:通过
#a4a7f1c9|李四|吃饭,睡觉|语文:80,数学:80|普通话:1甲,CET4:未通过
load data local inpath '/app/hive/data/data.csv' into table data_table;

#查看数据
select * from data_table;
#结果
99b28506	张三	["游泳","唱歌"]	{"语文":90,"数学":90}	{"item":"CET4:通过","level":"CET6:通过"}
a4a7f1c9	李四	["吃饭","睡觉"]	{"语文":80,"数学":80}	{"item":"普通话:1甲","level":"CET4:未通过"}

```

#### 表类型

Hive中的基本表类型有：内部表、外部表、分区表和分桶表，接下来我们就来分别学习一下这四种类型的表

##### 内部表

内部表就是Hive中的一般表，创建内部表不需要使用特殊的语法，直接使用create table就可以创建，内部表的数据，会存放在 HDFS 中的hive-site.xml配置项hive.metastore.warehouse.dir所配置的位置中，当内部表被删除时，其存储的数据文件也会一并删除

下面演示内部表的创建方式

```sql

#创建内部表inne_table，包含两个字段id、name
create table inner_table(id string,name string);
#向内部表插入一条数据
insert into inner_table(id,name) values('1','Tom');

#创建内部表inner_table2,包含两个字段id、name，并指定数据储存时字段之间分割符号为 ',' 
create table inner_table2(id string,name string) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',';

#从文件加载数据到inner_table2
load data local inpath '/app/hive/data/inner_table.csv' into table inner_table2;
#直接向表中插入数据
insert into inner_table2(id,name) values('4','Hbase');

```

##### 外部表

外部表适用于想要使用存储在 Hive之外的数据文件的情况，数据存在与否和表的定义互不约束，表仅仅是对hdfs上相应文件的一个引用，当删除外部表的时候，只是删除了表的元数据，它的数据并没有被删除，适合数据多部门组织共享的场景。外部表创建需要使用external关键字，使用create external table即可创建外部表

下面演示创建外部表

```sql

#创建外部表external_table，并指定存储字段之间分隔符为 ','
create external table external_table(id string,name string) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',';

#导入数据
load data inpath 'hdfs:///data.csv' into external_table;

```

##### 分区表

分区表即对表中的数据进行分区存储，相对于普通表分区表在通过where字句查询时，可以根据分区字段避免全表扫描的情况，可以通过合适的索引来扫描表中的一小部分，提高查询性能，创建分区表的时候我们需要指定分区字段，建表后在HDFS表目录下会生成一个使用分区字段名称作为目录名称的目录，如果有多级分区，子级分区的目录就是父分区目录的子目录。

分区表分为静态分区和动态分区两种，下面我们分别演示

1.静态分区

使用静态分区时，用户不仅需要指明分区字段，还需要在加载和插入数据时指明数据所属的分区，演示如下

```sql

#创建静态分区表part_table，分区字段为day、hour
create table part_table(id string,name string) 
partitioned by (day string,hour string) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',';

#导入数据到两个分区（20200730 19）和（20200730 20）
load data inpath 'hdfs:///data.csv' into table part_table partition (day='20200730',hour='19');
load data inpath 'hdfs:///data.csv' into table part_table partition (day='20200730',hour='20');

#查看分区
show partitions part_table;
#结果
day=20200730/hour=19
day=20200730/hour=20

#使用insert语句向分区表插入数据
insert into table part_table partition (day='20200731', hour='1') (id,name) values('7','HBase');

#删除分区
alert table part_table drop partiton(day='20200731',hour='1')

```

2.动态分区

使用动态分区时，我们只用指明分区字段即可，无需在插入数据时再去指明该数据所属的分区，Hive的动态分区默认是没有开启，所以我们要进行配置，开启动态分区后默认是以严格模式执行的，为了避免因设计错误导致查询产生大量的分区，在这种模式下需要至少一个分区字段是静态的

配置动态分区，三种方式可选

* hive-site.xml

```xml

<!--开启动态分区-->
<property>
	<name>hive.exec.dynamic.partition</name>
	<value>true</value>
</property>
<!--模式，strict：严格模式，至少一个静态分区字段；nonstrict：允许所有分区字段都是动态的-->
<property>
	<name>hive.exec.dynamic.partition.mode</name>
	<value>nonstrict</value>
</property>

```

* hive 启动参数：`./hive --hiveconf hive.exec.dynamic.partiton=true`

* hive命令行设置

```shell

#开启动态分区
set hive.exec.dynamic.partiton=true
#设置分区模式是nostrict
set hive.exec.dynamic.partiton.mode=nostrict

```

创建动态分区表演示

```sql

#建表语句与静态分区一致，无区别
create table part_table2(id string,name string) partitioned by (day string,hour string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

#将其他表的数据导入到该表并分区
insert overwrite table part_table2 partition(day,hour) select id,name,day,hour from part_table;

#查看分区
show partitions part_table2;

```

##### 分桶表

分桶表是通过对数据进行Hash，放到不同文件存储，方便抽样和join查询。分桶表主要是将内部表、外部表和分区表进一步组织，可以指定表的某些列通过Hash算法进一步分解成不同的文件存储。创建分桶表是需要使用关键字clustered by并指定分桶的个

创建分桶表的演示如下

```sql

#创建分桶表buk_table根据id分桶，放入3个桶中
create table buk_table(id string,name string) clustered by(id) into 3 buckets ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
#加载数据，将inner_table中数据加载到buk_table;
insert into buk_table select * from inner_table;
#桶中数据抽样：select * from table_name tablesample(bucket X out of Y on field);
# X表示从哪个桶中开始抽取，Y表示相隔多少个桶再次抽取
select * from buk_table tablesample(bucket 1 out of 1 on id);

```

#### 用户自定义函数

UDF（User Define Function）：用户自定义函数。

当Hive提供的内置函数无法满足业务场景需求，我们就可以使用UDF来自定义函数满足我们需求，Hive中提供了三种类型的自定义函数接口分别是：UDF、UDAF、UDTF。

我们先了解Hive提供的三种UDF的特点，然后对三种进行编码演示一个示例。

* UDF：一般类型函数，也可以理解为映射。接受单行输入，并产生单行输出，例：取字符串长度，日期格式化等
* UDAF：聚合类型函数（User Defined Aggregate Function）。接受多行输入，并产生单行输出，例：属性最大值，平均值，Count等
* UDTF：表生成函数（User Defined Table-generating Function）。接受单行输入，并产生多行输出（即一个表）

##### 1.代码中添加pom依赖

```xml

 <dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>2.3.6</version>
</dependency>

```

##### 2.UDF实现

UDF类型函数是需要继承抽象类org.apache.hadoop.hive.ql.udf.generic.GenericUDF的实现其中抽象方法，示例代码我们编写一个截取字符串的自定义函数

```java

/**
 * 截取指定长度字符串的UDF
 * @author 爱做梦的锤子
 * @create 2020/7/28
 */
public class StrSub extends GenericUDF {
    @Override
    public ObjectInspector initialize(ObjectInspector[] objectInspectors) throws UDFArgumentException {
        /**
         * 检验参数的个数不等于3个抛出异常，你也可以加入其它的校验，此处作为演示写的比较简略
         */
        if(objectInspectors.length!=3){
            throw new UDFArgumentException("Invalid num of arguments for StrSub");
        }
        /**
         * 返回该函数的返回结果的数据类型
         */
        return PrimitiveObjectInspectorFactory.javaStringObjectInspector;
    }

    @Override
    public Object evaluate(DeferredObject[] deferredObjects) throws HiveException {
        //取出三个参数，并转换成响应类型
        String sourceString = String.valueOf(deferredObjects[0].get());
        Integer start = Integer.valueOf(String.valueOf(deferredObjects[1].get()));
        Integer end = Integer.valueOf(String.valueOf(deferredObjects[2].get()));
        //对字符串截取，返回截取后的结果
        String targetString = sourceString.substring(start,end);
        return targetString;
    }

    @Override
    public String getDisplayString(String[] strings) {
        return "Function StrSub";
    }
}


```

##### 3.UDAF实现

UDAF类型函数是需要继承两个抽象类：org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver和org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator，并重写和实现其中的相关方法。示例代码如下，我们编写一个计算字符串长度和的聚合函数，代码中带有方法的注释和说明，可以参阅

```java

/**
 * 计算字符串长度和
 * @author 爱做梦的锤子
 * @create 2020/7/28
 */
public class StrLengthSum extends AbstractGenericUDAFResolver {
    static final Logger LOG = LoggerFactory.getLogger(StrLengthSum.class.getName());

    @Override
    public GenericUDAFEvaluator getEvaluator(TypeInfo[] info) throws SemanticException {
        //检验参数个数
        if (info.length != 1) {
            throw new UDFArgumentTypeException(info.length - 1, "Exactly one argument is expected.");
        }
        //校验参数类型，仅支持基础数据类型
        ObjectInspector oi = TypeInfoUtils.getStandardJavaObjectInspectorFromTypeInfo(info[0]);
        if (oi.getCategory() != ObjectInspector.Category.PRIMITIVE) {
            throw new UDFArgumentTypeException(0, "Only primitive type arguments are accepted but " + info[0].getTypeName() + " was passed as parameter 1.");
        }
        return new StrLengthSumEvaluator();
    }

    public static class StrLengthSumEvaluator extends GenericUDAFEvaluator {

        protected PrimitiveObjectInspector inputOI;
        protected PrimitiveObjectInspector outputOI;

        /**
         * 初始化，参数校验，定义输出类型
         */
        @Override
        public ObjectInspector init(Mode m, ObjectInspector[] parameters)
                throws HiveException {
            //检验参数个数
            assert (parameters.length == 1);
            super.init(m, parameters);
            //将输入参数赋值给inputOI
            inputOI = (PrimitiveObjectInspector) parameters[0];
            //设置输出结果数据类型
            outputOI = PrimitiveObjectInspectorFactory.writableIntObjectInspector;
            return outputOI;
        }

        /**
         * 缓冲区用来保存中间结果
         */
        @AggregationType(estimable = true)
        static class SumAgg extends AbstractAggregationBuffer {
            /**
             * 累加和
             */
            IntWritable sum;

            public void add(Integer integer) {
                sum.set(sum.get() + integer);
            }

            /**
             * 缓存区预分配内存大小
             * @return
             */
            @Override
            public int estimate() {
                return JavaDataModel.PRIMITIVES1;
            }
        }

        /**
         * 获取存放中间结果的缓冲对象
         */
        @Override
        public AggregationBuffer getNewAggregationBuffer() throws HiveException {
            SumAgg result = new SumAgg();
            reset(result);
            return result;
        }

        /**
         * 重置存放中间结果的缓冲类
         */
        @Override
        public void reset(AggregationBuffer agg) throws HiveException {
            SumAgg myagg = (SumAgg) agg;
            myagg.sum = new IntWritable(0);
        }

        /**
         * 处理一行数据
         */
        @Override
        public void iterate(AggregationBuffer agg, Object[] parameters)
                throws HiveException {
            //判断参数个数
            assert (parameters.length == 1);
            if (parameters[0] != null) {
                //取出参数和中间结果存储类，将参数转换成java原始类型，计算长度然后累加
                SumAgg myagg = (SumAgg) agg;
                Object primitiveJavaObject = inputOI.getPrimitiveJavaObject(parameters[0]);
                myagg.add(String.valueOf(primitiveJavaObject).length());
            }

        }

        /**
         * 返回部分聚合数据的持久化对象。
         * 因为调用这个方法时，说明已经是map或者combine的结束了，必须将数据持久化以后交给reduce进行处理。
         * 只支持JAVA原始数据类型及其封装类型、HADOOP Writable类型、List、Map，不支持自定义的类
         */
        @Override
        public Object terminatePartial(AggregationBuffer agg) throws HiveException {
            return terminate(agg);
        }

        /**
         * 将terminatePartial返回的部分聚合数据进行合并
         */
        @Override
        public void merge(AggregationBuffer agg, Object partial)
                throws HiveException {
            if (partial != null) {
                SumAgg myagg = (SumAgg) agg;
                Integer partialSum = PrimitiveObjectInspectorUtils.getInt(partial, outputOI);
                myagg.add(partialSum);
            }
        }

        /**
         * 生成最终结果
         */
        @Override
        public Object terminate(AggregationBuffer agg) throws HiveException {
            SumAgg myagg = (SumAgg) agg;
            return myagg.sum;
        }

        @Override
        public GenericUDAFEvaluator getWindowingEvaluator(WindowFrameDef wFrmDef) {
            return null;
        }
    }
}


```

##### 4.UDTF实现

UDTF类型函数是要继承抽象类site.teamo.learning.hive.udf.udtf.GenericUDTF，并重写和实现其中相关方法。示例代码如下，我们编写一个将字符串按照 ';' 和 ','分割成多行的函数

```java

/**
 * 将字符串按照 ; 和 , 分割成多行
 * @author 爱做梦的锤子
 * @create 2020/7/29
 */
public class Str2Table extends GenericUDTF {
    static final Logger LOG = LoggerFactory.getLogger(Str2Table.class.getName());
    private static final String ROW_SEPARATOR = ";";
    private static final String ATTR_SEPARATOR= ",";
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
        //校验参数个数
        List<? extends StructField> inputFields = argOIs.getAllStructFieldRefs();
        if(inputFields.size()!=1){
            throw new UDFArgumentException("Invalid num of arguments for Str2Table");
        }
        //构造输出结果的数据结构，字段名和字段类型
        ArrayList<String> fieldNames = new ArrayList<>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<>();
        fieldNames.add("col1");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        fieldNames.add("col2");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }

    /**
     * 处理输入数据的方法
     * @param args
     * @throws HiveException
     */
    @Override
    public void process(Object[] args) throws HiveException {
        assert (args.length == 1);
        //获取输入数据
        String input = String.valueOf(args[0]);
        //按 ； 进行分割成行
        String[] output= input.split(ROW_SEPARATOR);
        //遍历每一行
        for(int i=0; i<output.length; i++) {
            try {
                //行再按 , 分割获得属性
                String[] result = output[i].split(ATTR_SEPARATOR);
                //调用forward生成一行数据
                forward(result);
            } catch (Exception e) {
                LOG.warn("row format error:{}",output[i]);
            }
        }
    }

    @Override
    public void close() throws HiveException {

    }
}

```

##### 5.将用户自定义函数注册到Hive

注册自定义函数到Hive有两种方式：临时注册和永久注册。

临时注册仅在当前连接中生效，操作步骤如下：

```shell

#加载jar包
add jar hdfs:///hive/udf/module-7-1.0-SNAPSHOT.jar;
#临时注册三个自定义函数
create temporary function StrSub AS 'site.teamo.learning.hive.udf.udf.StrSub';

create temporary function StrLengthSum AS 'site.teamo.learning.hive.udf.udaf.StrLengthSum';

create temporary function Str2Table AS 'site.teamo.learning.hive.udf.udtf.Str2Table';

#永久注册自定义函数，需要将jar包放到HDFS上
#create function UDF_NAME as 'className' using jar 'jarPath';

```

##### 6.使用演示

准备测试表和数据如下所示

![测试表](http://file.te-amo.site/images/thumbnail/%E4%B8%80%E8%B5%B7%E5%AD%A6%E4%B9%A0Hive%E5%9F%BA%E7%A1%80/table.png)

使用自定义函数语句

```shell

#UDF函数测试Sql语句如下
select id,StrSub(str,0,2) from udf;
#效果如下
hive> select id,StrSub(str,0,2) from udf;
OK
1	ab
2	hi
Time taken: 0.737 seconds, Fetched: 2 row(s)


#UDAF函数测试Sql语句如下
select StrLengthSum(str) from udaf group by id;
#效果如下
hive> select id,StrLengthSum(str) from udaf group by id;
OK
1	6
2	6
Time taken: 1.567 seconds, Fetched: 2 row(s)


#UDTF函数测试Sql语句如下
select Str2Table(str) from udtf;
#效果如下
hive> select Str2Table(str) from udtf;
OK
attr1	attr2
bttr1	bttr2
cttr1	cttr2
dttr1	dttr2
Time taken: 0.131 seconds, Fetched: 4 row(s)

```

> 个人公众号【**爱做梦的锤子**】，全网同id，个站 [http://te-amo.site](http://te-amo.site/)，欢迎关注，里面会分享更多有用知识

> 觉得不错就点个赞叭QAQ
