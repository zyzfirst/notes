---
title: Day-15 HBase
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 安装配置
- 解压配置环境变量
- 进入hbse的conf的目录配置hbase.env(环境,不启用自带zookeeper) ,hbase-site.xml(节点,HMaster,data目录,tmp目录等),regionservers(配置节点名称)
- scp配置信息启动服务,必须先启动zookeeper

>在某一台上解压hbase的压缩文件，如在192.168.15.5
tar –zxvf hbase-1.2.0-bin.tar.gz
配置添加环境变量：
#hbase
export HBASE_HOME=/usr/tools/hbase-1.2.0
export PATH=$PATH:$HBASE_HOME/bin
使环境变量生效
source /etc/profile

>进入hbase的conf目录，需要修改三个文件：hbase-env.sh、hbase-site.xml和regionservers
其中hbase-env.sh中，在文档的十多行位置处添加：
export JAVA_HOME=/usr/tools/jdk1.8.0_73
然后在后面添加：
export HBASE_MANAGES_ZK=false
regionservers文件中添加各个从属服务器的ip或者hostname：
jokeros1
jokeros2
jokeros3
hbase-site.xml中

``` stylus
<configuration>
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>jokeros1,jokeros2,jokeros3</value>
                <description>The directory shared by RegionServers.</description>
        </property>
        <property>
                <name>hbase.zookeeper.property.dataDir</name>
                <value>/usr/tools/hbase-1.2.0/zookeeperdata</value>
                <description>Property from ZooKeeper config zoo.cfg.
                The directory where the snapshot is stored.
                </description>
        </property>
        <property>
                <name>hbase.tmp.dir</name>
                <value>/usr/tools/hbase-1.2.0/tmpdata</value>
        </property>
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://jokeros1:9000/hbase</value>
                <description>The directory shared by RegionServers.</description>
        </property>
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
                <description>The mode the cluster will be in. Possible values are
                false: standalone and pseudo-distributed setups with managed Zookeeper
                true: fully-distributed with unmanaged Zookeeper Quorum (see hbase-env.sh)
                </description>
        </property>
</configuration>
```

>保存后分别把hbase的整个文件夹拷贝到其他服务器：
scp /usr/tools/hbase-1.2.0 root@jokeros2: /usr/tools/
scp /usr/tools/hbase-1.2.0 root@jokeros3: /usr/tools/
在hadoop的namenode节点上启动hbase服务
start-hbase.sh
启动后：jps
HRegionServer
HMaster
子节点
HRegionServer
启动顺序
Hadoop-hdfs-------》hadoop-yarn------》zookeeper------》hbase

## 设置备用节点
在hbase的conf目录下,touch一个自定义文件backup-masters 然后编辑slaver1(备用节点的名称) 然后把它cp到其它节点

![][1]


# hbase的数据存储结构

![][2]

>kv的格式

![][3]

## hbase的特点
>hbase是反模式的数据库,列式存储,用空间换性能,储存空间需要比较大,查询比较快,而数据库是模式,为了减少冗余数据
 >hbase是稀疏的,有字段就存储没有的话就不在存储,而rdbms不管有没有值都会存在字段,所以hbase也节约了空间,灵活性比较高,但是规则低,所以由自行的约定规则,不在强加

# 命令
>hbase shell 命令
 create_namespace 'bd14'     //创建数据库
 create 'bd14:user','i','c'        //创建表 指定列簇
 list_namespace_tables 'bd14'  //查看表 指定数据库
 list_namespace                  //查看当前数据库
 describe 'bd14:user'     //查看表结构 指定表名字  数据库:表名 并且用单引号
 put 'bd14:user','1','i:username','zhangsan' 插入数据 '表名''rowkey''列名''时间戳' 如果不写列名,那么也会加进去,只不过是列名为空
 get 'bd14:user','1' //查看数据  在value前边由一个时间戳
 'bd14:user','1','c:email' +'时间戳' 可以加上时间戳更精确,前边是必填 rpwkey+列名 确定一条数据
 truncate 'bd14:user' //清空表数据
 disable 'bd14:user' drop 'bd14:user'  //删除表结构,需要先disable,然后再执行drop命
 scan 'bd14:user'  查看表中数据,是看整张表
 
 # hbase系统
 
 ![][4]

>其中HBase位于结构化存储层，Hadoop HDFS为HBase提供了高可靠性的底层存储支持，Hadoop MapReduce为HBase提供了高性能的计算能力，Zookeeper为HBase提供了稳定服务和failover机制。此外，Pig和Hive还为HBase提供了高层语言支持，使得在HBase上进行数据统计处理变的非常简单。 Sqoop则为HBase提供了方便的RDBMS数据导入功能，使得传统数据库数据向HBase中迁移变的非常方便。

# hbase的简介
>它的使用场景: 一次写入,多次读写,历时数据的查询,电台平台的订单查询(一次写入多次读写,话单信息等,日志信息)并且内部是以kv对的形式进行存储,它的索引即key的值,会进行排序,在存储之前会进行排序,这样就是有序的所以能快速查询,另外汇出吸纳冗余存储rowkey和列簇等,所以为了节省空间,尽可能的短一点,最好一个字母代表,在增删改的时候,请求过来会先发送大zookeeper上,得到meta表的位置,然后在访问meta表得知data的信息
>发送请求--->zookeeper的hbase/meta-region-server找到hbase上的meta表的位置-->查询meta表,找到数据在哪个regionserver上--->查询rowkey等key来获取value

# Hbase接口
1.       Native Java API，最常规和高效的访问方式，适合Hadoop MapReduce Job并行批处理HBase表数据
2.       HBase Shell，HBase的命令行工具，最简单的接口，适合HBase管理使用
3.       Thrift Gateway，利用Thrift序列化技术，支持C++，PHP，Python等多种语言，适合其他异构系统在线访问HBase表数据
4.       REST Gateway，支持REST 风格的Http API访问HBase, 解除了语言限制
5.       Pig，可以使用Pig Latin流式编程语言来操作HBase中的数据，和Hive类似，本质最终也是编译成MapReduce Job来处理HBase表数据，适合做数据统计
6.       Hive，当前Hive的Release版本尚没有加入对HBase的支持，但在下一个版本Hive 0.7.0中将会支持HBase，可以使用类似SQL语言来访问HBase

# hbase的数据模型
>HBase中有两张特殊的Table，-ROOT-和.META.
Ø  .META.：记录了用户表的Region信息，.META.可以有多个regoin
Ø  -ROOT-：记录了.META.表的Region信息，-ROOT-只有一个region
Ø  Zookeeper中记录了-ROOT-表的location

![][5]

Client访问用户数据之前需要首先访问zookeeper，然后访问-ROOT-表，接着访问.META.表，最后才能找到用户数据的位置去访问，中间需要多次网络操作，不过client端会做cache缓存。

# hbase的系统架构
![][6]

## Client
HBase Client使用HBase的RPC机制与HMaster和HRegionServer进行通信，对于管理类操作，Client与HMaster进行RPC；对于数据读写类操作，Client与HRegionServer进行RPC
## Zookeeper
Zookeeper Quorum中除了存储了-ROOT-表的地址和HMaster的地址，HRegionServer也会把自己以Ephemeral方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的健康状态。此外，Zookeeper也避免了HMaster的单点问题，见下文描述
## HMaster
HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的Master Election机制保证总有一个Master运行，HMaster在功能上主要负责Table和Region的管理工作：
1.       管理用户对Table的增、删、改、查操作
2.       管理HRegionServer的负载均衡，调整Region分布
3.       在Region Split后，负责新Region的分配
4.       在HRegionServer停机后，负责失效HRegionServer 上的Regions迁移

## HRegionServer
HRegionServer主要负责响应用户I/O请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。

![][7]

**注意:HRegin是表的一部分,它由rowkey划分范围,HR**

RegionServer内部管理了一系列HRegion对象，每个HRegion对应了Table中的一个Region，HRegion中由多个HStore组成。每个HStore对应了Table中的一个Column Family的存储，可以看出每个Column Family其实就是一个集中的存储单元，因此最好将具备共同IO特性的column放在一个Column Family中，这样最高效。
HStore存储是HBase存储的核心了，其中由两部分组成，一部分是MemStore，一部分是StoreFiles。MemStore是Sorted Memory Buffer，用户写入的数据首先会放入MemStore，当MemStore满了以后会Flush成一个StoreFile（底层实现是HFile），当StoreFile文件数量增长到一定阈值，会触发Compact合并操作，将多个StoreFiles合并成一个StoreFile，合并过程中会进行版本合并和数据删除，因此可以看出HBase其实只有增加数据，所有的更新和删除操作都是在后续的compact过程中进行的，这使得用户的写操作只要进入内存中就可以立即返回，保证了HBase I/O的高性能。当StoreFiles Compact后，会逐步形成越来越大的StoreFile，当单个StoreFile大小超过一定阈值后，会触发Split操作，同时把当前Region Split成2个Region，父Region会下线，新Split出的2个孩子Region会被HMaster分配到相应的HRegionServer上，使得原先1个Region的压力得以分流到2个Region上。下图描述了Compaction和Split的过程：

![][8]

在理解了上述HStore的基本原理后，还必须了解一下HLog的功能，因为上述的HStore在系统正常工作的前提下是没有问题的，但是在分布式系统环境中，无法避免系统出错或者宕机，因此一旦HRegionServer意外退出，MemStore中的内存数据将会丢失，这就需要引入HLog了。每个HRegionServer中都有一个HLog对象，HLog是一个实现Write Ahead Log的类，在每次用户操作写入MemStore的同时，也会写一份数据到HLog文件中（HLog文件格式见后续），HLog文件定期会滚动出新的，并删除旧的文件（已持久化到StoreFile中的数据）。当HRegionServer意外终止后，HMaster会通过Zookeeper感知到，HMaster首先会处理遗留的 HLog文件，将其中不同Region的Log数据进行拆分，分别放到相应region的目录下，然后再将失效的region重新分配，领取 到这些region的HRegionServer在Load Region的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。

# HBase存储格式
HBase中的所有数据文件都存储在Hadoop HDFS文件系统上，主要包括上述提出的两种文件类型：
1.       HFile， HBase中KeyValue数据的存储格式，HFile是Hadoop的二进制格式文件，实际上StoreFile就是对HFile做了轻量级包装，即StoreFile底层就是HFile
2.       HLog File，HBase中WAL（Write Ahead Log） 的存储格式，物理上是Hadoop的Sequence File

## HFile
首先HFile文件是不定长的，长度固定的只有其中的两块：Trailer和FileInfo。正如图中所示的，Trailer中有指针指向其他数据块的起始点。File Info中记录了文件的一些Meta信息，例如：AVG_KEY_LEN, AVG_VALUE_LEN, LAST_KEY, COMPARATOR, MAX_SEQ_ID_KEY等。Data Index和Meta Index块记录了每个Data块和Meta块的起始点。
Data Block是HBase I/O的基本单元，为了提高效率，HRegionServer中有基于LRU的Block Cache机制。每个Data块的大小可以在创建一个Table的时候通过参数指定，大号的Block有利于顺序Scan，小号Block利于随机查询。每个Data块除了开头的Magic以外就是一个个KeyValue对拼接而成, Magic内容就是一些随机数字，目的是防止数据损坏。后面会详细介绍每个KeyValue对的内部构造。

![][9]

HFile里面的每个KeyValue对就是一个简单的byte数组。但是这个byte数组里面包含了很多项，并且有固定的结构。我们来看看里面的具体结构：

![][10]

开始是两个固定长度的数值，分别表示Key的长度和Value的长度。紧接着是Key，开始是固定长度的数值，表示RowKey的长度，紧接着是RowKey，然后是固定长度的数值，表示Family的长度，然后是Family，接着是Qualifier，然后是两个固定长度的数值，表示Time Stamp和Key Type（Put/Delete）。Value部分没有这么复杂的结构，就是纯粹的二进制数据了。

## HLog File

![][11]

上图中示意了HLog文件的结构，其实HLog文件就是一个普通的Hadoop Sequence File，Sequence File 的Key是HLogKey对象，HLogKey中记录了写入数据的归属信息，除了table和region名字外，同时还包括 sequence number和timestamp，timestamp是“写入时间”，sequence number的起始值为0，或者是最近一次存入文件系统中sequence number。
HLog Sequece File的Value是HBase的KeyValue对象，即对应HFile中的KeyValue

# Hbase的应用场景
hbase的两个应用场景:第一是对接web应用用于查询结果的展示,第二个是作为数据来源和数据分析的保存

>**数据来源:**hbase表中数据的来源:业务数据和批处理数据

![][12]

# Hbase的行键设计原则
## 原理
HBase是一个分布式的、面向列的数据库，它和一般关系型数据库的最大区别是：HBase很适合于存储非结构化的数据，还有就是它基于列的而不是基于行的模式。
既然HBase是采用KeyValue的列存储，那Rowkey就是KeyValue的Key了，表示唯一一行。Rowkey也是一段二进制码流，最大长度为64KB，使用上一般不超过100个字节,内容可以由使用的用户自定义。数据加载时，一般也是根据Rowkey的二进制序由小到大进行的。
HBase是根据Rowkey来进行检索的，系统通过找到某个Rowkey (或者某个 Rowkey 范围)所在的Region，然后将查询数据的请求路由到该Region获取数据。HBase的检索支持3种方式：
（1） 通过单个Rowkey访问，即按照某个Rowkey键值进行get操作，这样获取唯一一条记录；
（2） 通过Rowkey的range进行scan，即通过设置startRowKey和endRowKey，在这个范围内进行扫描。这样可以按指定的条件获取一批记录；
（3） 全表扫描，即直接扫描整张表中所有行记录。
HBASE按单个Rowkey检索的效率是很高的，耗时在1毫秒以下，每秒钟可获取1000~2000条记录，不过非key列的查询很慢。

![][13]

## 行键原则

### Rowkey长度原则
Rowkey是一个二进制码流，Rowkey的长度被很多开发者建议说设计在10~100个字节，不过建议是越短越好，不要超过16个字节。
1.不宜过长,在100字节以内
2.定长 方便划分范围
3.长度最好是8的整数倍(每次读8个字节)
>原因如下：
（1）数据的持久化文件HFile中是按照KeyValue存储的，如果Rowkey过长比如100个字节，1000万列数据光Rowkey就要占用100*1000万=10亿个字节，将近1G数据，这会极大影响HFile的存储效率；
（2）MemStore将缓存部分数据到内存，如果Rowkey字段过长内存的有效利用率会降低，系统将无法缓存更多的数据，这会降低检索效率。因此Rowkey的字节长度越短越好。
（3）目前操作系统是都是64位系统，内存8字节对齐。控制在16个字节，8字节的整数倍利用操作系统的最佳特性。

### Rowkey散列原则
原因:时间是顺序的,所以会进入一个regin,所以高位随机产生保证散列

Rowkry散列的方法:
1.随机数
2.Uuid
3.md5, hash等加密算法
4.业务有序数据反向(对业务有序数据进行反向 reverse)
如果Rowkey是按时间戳的方式递增，不要将时间放在二进制码的前面，建议将Rowkey的高位作为散列字段，由程序循环生成，低位放时间字段， 这样将提高数据均衡分布在每个Regionserver实现负载均衡的几率。如果没有散列字段，首字段直接是时间信息将产生所有新数据都在一个 RegionServer上堆积的热点现象，这样在做数据检索的时候负载将会集中在个别RegionServer，降低查询效率。
### Rowkey唯一原则
必须在设计上保证其唯一性。
Rowkey作为索引原则
Rowkey是hbase里面的唯一索引,对于某些查询频繁的限定条件数据需要把它的内容存放在roqkey里面

### Rowkey 的字符串原则

![][14]

# 二级索引
## 关系型数据库的有关功能实现
数据库的业务逻辑,不便与迁移(events(存储过程),triggers触发器(检测插入等))
协处理器coprocesser:endpoint(存储过程) observer(触发器)
>关系型数据库是通过event和triggers来实现 表关联,在插入数据的时候,对关联的表做响应的操作
>phoenix通过coprocesser来实现  表和表的关联,当插入数据的时候,同时插入到索引库

![][15]

>二级索引hbase不支持(经常查询,又不能放在rowkey中)
>hbase中不支持其他索引,索引就是rowkey,只支持单行的事务,对于事务型的应用支持不好,支持查询型的一些应用,并且不支持表关联(phoenix) phoenix解决这些问题(二级索引,表关联)
>**habase的元数据在phoenix维护,像hive的元数据维护在mysql,**

## javaAPI来读取,存储hbase
首先需要引入依赖hbase-server和hbase-client

### map
- 继承tableMap 
- 值封装在value中,通过get.row和get.value可以得到rowkey和响应的值
- 需要TableMapReduceUtil.initTableMapperJob("bd14:orderItemsByRow",scan, OrderItemsIndexMap.class, Text.class, Text.class, job);

### reduce
- 继承tablereduce
- 创建Put对象,put=rowkey,put.addcolumn
- key为nullwriable,value就是Mutation类型
- TableMapReduceUtil.initTableReducerJob("bd14:orderIndexTest", OrderItemsIndexReduce.class, job);指定输出表和reduce的类加载器

# 协处理器
## 目的
HBase作为列族数据库最经常被人诟病的特性包括：无法轻易建立“二级索引”，难以执行求和、计数、排序等操作。,为了解决这些问题,推出了协处理器

## 特性:分布式
包括以下特性:
每个表服务器的任意子表都可以运行代码
客户端的高层调用接口(客户端能够直接访问数据表的行地址，多行读写会自动分片成多个并行的RPC调用)
提供一个非常灵活的、可用于建立分布式服务的数据模型
能够自动化扩展、负载均衡、应用请求路由
HBase的协处理器灵感来自bigtable，但是实现细节不尽相同。HBase建立了一个框架，它为用户提供类库和运行时环境，使得他们的代码能够在HBase region server和master上处理。

## 原理
协处理器分两种类型，系统协处理器可以全局导入region server上的所有数据表，表协处理器即是用户可以指定一张表使用协处理器。协处理器框架为了更好支持其行为的灵活性，提供了两个不同方面的插件。一个是观察者（observer），类似于关系数据库的触发器。另一个是终端(endpoint)，动态的终端有点像存储过程。

### observer

观察者的设计意图是允许用户通过插入代码来重载协处理器框架的upcall方法，而具体的事件触发的callback方法由HBase的核心代码来执行。协处理器框架处理所有的callback调用细节，协处理器自身只需要插入添加或者改变的功能。
以HBase0.92版本为例，它提供了三种观察者接口：
RegionObserver：提供客户端的数据操纵事件钩子(即回调函数)：Get、Put、Delete、Scan等,一般使用BaseRegionObserver。
WALObserver：提供WAL相关操作钩子(监听日志记录)。
MasterObserver：提供DDL-类型的操作钩子。如创建、删除、修改数据表等(对表的操作是hmaster来负责,具体表的数据操作是reginserver来负责)。

#### javaAPI的实现

##### 是自定义的协处理器生效,不完美如果是update操作,或是delete操作,没考虑callback的操作
步骤1.创建项目,添加hbase依赖,在项目定义observer类,继承basereginonserver类,重写方法监听触发功能
	   2.项目打成jar包,放到hdfs上,或是hbase的lib下(三个节点都要有,所以放到hdfs上)
	    3.把协处理器添加到源表  alter 'bd14:orderItemsByRow','coprocessor'=>'hdfs:///comprocess.jar|
	     com.zhiyou.hbase.comprocess.SecondryIndexAutoUpdate|1001|'

``` stylus
package com.zhiyou.hbase.comprocess;

import java.io.IOException;
import java.util.List;

import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Durability;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.coprocessor.BaseRegionObserver;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.regionserver.wal.WALEdit;
import org.apache.hadoop.hbase.util.Bytes;

/**  
* @ClassName: SecondryIndexAutoUpdate  
* @Description: TODO  
* @author zyz  
* @date 2017年11月4日 下午3:07:59  
*   
*/
//使用observer的comprocess来自动更新order_items的二级索引数据
//把这个协处理器添加到order_items表上,索引数据自动更新到order_items_index里
public class SecondryIndexAutoUpdate extends BaseRegionObserver {

	//在表数据被put之前执行索引数据的添加,也可以在之后添加索引,因为同属一个transation,所以会保证同时成功
	//e 是上下文环境,put是传过来要更新或是插入的put对象,edit是日志,druablity是表明两者关系,先写日志还是先插入数据
	@Override
	public void prePut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability)
			throws IOException {
		//获取此次写操作的subtotal的值的cell.做判断,如果不含subtotal那么此次操作无效(加入原表的数据中截取index表中的rowkey)
	    List<Cell> subtotalCell = put.get(Bytes.toBytes("i"), Bytes.toBytes("order_items_subtotal"));
	    if(subtotalCell!=null && subtotalCell.size()>0){
	    	RegionCoprocessorEnvironment environment = e.getEnvironment();
			Connection connection = ConnectionFactory.createConnection(environment.getConfiguration());
		    Table table = connection.getTable(TableName.valueOf("bd14:orderIndex"));
	    	//把获取的subtotal的值,作为rowkey赋值给put
		    Put indexPut = new Put(CellUtil.cloneValue(subtotalCell.get(0)));
	        //添加原表的rowkey作为index表的列名
		    indexPut.addColumn("i".getBytes(), put.getRow(), null);
	        table.put(indexPut);
	        table.close();
	    }
	
	}
	//步骤1.创建项目,添加hbase依赖,在项目定义observer类,继承basereginonserver类,重写方法监听触发功能
	//     2.项目打成jar包,放到hdfs上,或是hbase的lib下(三个节点都要有,所以放到hdfs上)
	//     3.把协处理器添加到源表  alter 'bd14:orderItemsByRow','coprocessor'=>'hdfs:///comprocess.jar|
	//     com.zhiyou.hbase.comprocess.SecondryIndexAutoUpdate|1001|'
}
```


### endpoint(全局:求和、计数、排序)
终端是动态RPC插件的接口，它的实现代码被安装在服务器端，从而能够通过HBase RPC唤醒。客户端类库提供了非常方便的方法来调用这些动态接口，它们可以在任意时候调用一个终端，它们的实现代码会被目标region远程执行，结果会返回到终端。用户可以结合使用这些强大的插件接口，为HBase添加全新的特性。终端的使用，如下面流程所示：
定义一个新的protocol接口，必须继承CoprocessorProtocol.
实现终端接口，该实现会被导入region环境执行。
继承抽象类BaseEndpointCoprocessor.
在客户端，终端可以被两个新的HBase Client API调用 。单个region：HTableInterface.coprocessorProxy(Class<T> protocol, byte[] row) 。rigons区域：HTableInterface.coprocessorExec(Class<T> protocol, byte[] startKey, byte[] endKey, Batch.Call<T,R> callable)


#### 配置  :启用协处理器 Aggregation(Enable Coprocessor Aggregation)

>我们有两个方法：1.启动全局aggregation，能过操纵所有的表上的数据。通过修改hbase-site.xml这个文件来实现，只需要添加如下代码：
<property> <name>hbase.coprocessor.user.region.classes</name> <value>org.apache.hadoop.hbase.coprocessor.AggregateImplementation</value> </property>

>2.启用表aggregation，只对特定的表生效。通过HBase Shell 来实现。
(1)disable指定表。hbase> disable 'mytable'
(2)添加aggregation hbase> alter 'mytable', METHOD => 'table_att','coprocessor'=>'|org.apache.hadoop.hbase.coprocessor.AggregateImplementation||'
(3)重启指定表 hbase> enable 'mytable'

``` stylus
package com.zhiyou.hbase.aggregation;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.coprocessor.AggregationClient;
import org.apache.hadoop.hbase.client.coprocessor.LongColumnInterpreter;

/**  
* @ClassName: RowCountAggregation  
* @Description: TODO  
* @author zyz  
* @date 2017年11月4日 下午4:54:44  
*   
*/
public class RowCountAggregation {

	public static Configuration configuration = HBaseConfiguration.create();
    public AggregationClient aggregationClient;
    public RowCountAggregation(){
    	aggregationClient = new AggregationClient(configuration);
    }
    public void getRowCount() throws Throwable{
    	Scan scan = new Scan();
    	scan.addFamily("i".getBytes());
    	//new LongColumnInterpreter() 这个类把数值转换成long类型的再去计算
    	long rowCount = aggregationClient.rowCount(TableName.valueOf("bd14:orderIndex"), new LongColumnInterpreter(), scan);
       System.out.println(rowCount);
    }
    public static void main(String[] args) throws Throwable {
		RowCountAggregation rowCountAggregation = new RowCountAggregation();
	    rowCountAggregation.getRowCount();
    }
}
```





  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509721184008.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509272728879.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509272746527.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509368831230.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509370482965.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372327310.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372392241.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372416651.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372820410.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372851487.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509372888816.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509722866292.jpg
  [13]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509807640809.jpg
  [14]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509723887527.jpg
  [15]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509724348720.jpg