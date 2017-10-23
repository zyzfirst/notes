---
title: Day-11 Hive基础知识
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# Hive的相关知识拓展
## oracle和mysql的区别
>1.  Oracle是大型数据库而Mysql是中小型数据库，Oracle市场占有率达40%，Mysql只有20%左右，同时Mysql是开源的而Oracle价格非常高。
2. Oracle支持大并发，大访问量，是OLTP最好的工具。
3. 安装所用的空间差别也是很大的，Mysql安装完后才152M而Oracle有3G左右，且使用的时候Oracle占用特别大的内存空间和其他机器性能。
4.Oracle也Mysql操作上的一些区别
①主键 Mysql一般使用自动增长类型，在创建表时只要指定表的主键为auto increment,插入记录时，不需要再指定该记录的主键值，Mysql将自动增长；Oracle没有自动增长类型，主键一般使用的序列，插入记录时将序列号的下一个值付给该字段即可；只是ORM框架是只要是native主键生成策略即可。
②单引号的处理 MYSQL里可以用双引号包起字符串，ORACLE里只可以用单引号包起字符串。在插入和修改字符串前必须做单引号的替换：把所有出现的一个单引号替换成两个单引号。
③翻页的SQL语句的处理 MYSQL处理翻页的SQL语句比较简单，用LIMIT 开始位置, 记录个数；ORACLE处理翻页的SQL语句就比较繁琐了。每个结果集只有一个ROWNUM字段标明它的位置, 并且只能用ROWNUM小于100不能用ROWNUM>80
④ 长字符串的处理 长字符串的处理ORACLE也有它特殊的地方。INSERT和UPDATE时最大可操作的字符串长度小于等于4000个单字节, 如果要插入更长的字符串, 请考虑字段用CLOB类型，方法借用ORACLE里自带的DBMS_LOB程序包。插入修改记录前一定要做进行非空和长度判断，不能为空的字段值和超出长度字段值都应该提出警告,返回上次操作。 ⑤空字符的处理 MYSQL的非空字段也有空的内容，ORACLE里定义了非空字段就不容许有空的内容。按MYSQL的NOT NULL来定义ORACLE表结构, 导数据的时候会产生错误。因此导数据时要对空字符进行判断，如果为NULL或空字符，需要把它改成一个空格的字符串。
⑥字符串的模糊比较 MYSQL里用 字段名 like '%字符串%',ORACLE里也可以用 字段名 like '%字符串%' 但这种方法不能使用索引, 速度不快。
⑦Oracle实现了ANSII SQL中大部分功能，如，事务的隔离级别、传播特性等而Mysql在这方面还是比较的弱

## 数据库和数据仓库的区别
>数据库：传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。
数据仓库：数据仓库系统的主要应用主要是OLAP（On-Line Analytical Processing）联机事务处理，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。

>对于某个非常具体，且能够对公司决策起到关键性作用的问题，基本很难从业务数据库从调取出来。原因在于：
业务数据库中的数据结构是为了完成交易而设计的，不是为了而查询和分析的便利设计的。
业务数据库大多是读写优化的，即又要读（查看商品信息），也要写（产生订单，完成支付）。因此对于大量数据的读（查询指标，一般是复杂的只读类型查询）是支持不足的。

>数据仓库的作用在于：
数据结构为了分析和查询的便利；
只读优化的数据库，即不需要它写入速度多么快，只要做大量数据的复杂查询的速度足够快就行了。
那么在这里前一种业务数据库（读写都优化）的是业务性数据库，后一种是分析性数据库，即数据仓库。
最后总结一下：
数据库 比较流行的有：MySQL, Oracle, SqlServer等
数据仓库 比较流行的有：AWS Redshift, Greenplum, Hive等
这样把数据从业务性的数据库中提取、加工、导入分析性的数据库就是传统的 ETL 工作。现在也有一些新的方法，这展开说又是另一件事情了，有机会再详细说说。

## HUE的简单介绍
### 概括
>即 **HUE=Hadoop User Experience**

>Hue是一个开源的Apache Hadoop UI系统，由Cloudera Desktop演化而来，最后Cloudera公司将其贡献给Apache基金会的Hadoop社区，它是基于Python Web框架Django实现的。

>通过使用Hue我们可以在浏览器端的Web控制台上与Hadoop集群进行交互来分析处理数据，例如操作HDFS上的数据，运行MapReduce Job，执行Hive的SQL语句，浏览HBase数据库等等。

### 核心功能
>1.SQL编辑器，支持Hive, Impala, MySQL, Oracle, PostgreSQL, SparkSQL, Solr SQL, Phoenix…
2.搜索引擎Solr的各种图表
3.Spark和Hadoop的友好界面支持
4.支持调度系统Apache Oozie，可进行workflow的编辑、查看

### hdfs文件浏览
>HUE可以很方便的浏览HDFS中的目录和文件，并且进行文件和目录的创建、复制、删除、下载以及修改权限等操作。
>HDFS实现了一个和POSIX系统类似的文件和目录的权限模型。每个文件和目录有一个所有者（owner）和一个组（group）。文件或目录对其所有者、同组的其他用户以及所有其他用户分别有着不同的权限。
>使用HUE访问HDFS时，HDFS简单的将HUE上的用户名和组的名称进行权限的校验。

![][1]

### hive查询
>HUE的beeswax app提供友好方便的Hive查询功能，能够选择不同的Hive数据库，编写HQL语句，提交查询任务，并且能够在界面下方看到查询作业运行的日志。在得到结果后，还提供进行简单的图表分析能力。

>点击”Data Browsers”->”Metastore表”，还可以看到Hive中的数据库，数据库中的表以及各个表的元数据等信息。

![][2]

## Impala,  PostgreSQL, SparkSQL的比较
>首先共同点是源于hive,并且没有自己的metastore,采用的是hive的metastore

>**SparkSQL:** 删去了hive的mapreduce阶段,只采用metastore和hdfs机制,它是基于内存的,属于Spark框架的一部分.,提供通过Apache Hive的SQL变体Hive查询语言（HiveQL）与Spark进行交互的API。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。而不是mapreduce操作

>**Impala:** 它是基于SparkSQL,只是在SparkSQL的基础上增加量缓存机制,因此处理速度更加快速

>**PostgreSQL:** 同样底层也不是mapreduce,是基于‘execute执行器去执行任务

## 常见错误
>**1.sql执行不成功,提示权限不够** 那么去修改hadoop的配置文件

![][3]

>2.发送MR任务到yarn上执行,提醒内存不够,那么去修改默认内存

![][4]

![][5]

![][6]

# Hive的基础知识
## 概述
>Hive 是建立在 Hadoop 上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL），这是一种可以存储、查询和分析存储在 Hadoop 中的大规模数据的机制。Hive 定义了简单的类 SQL 查询语言，称为 hQL，它允许熟悉 SQL 的用户查询数据。同时，这个语言也允许熟悉 MapReduce 开发者的开发自定义的 mapper 和 reducer 来处理内建的 mapper 和 reducer 无法完成的复杂的分析工作。

>Hive 并不适合那些需要低延迟的应用，例如，联机事务处理（OLTP）。Hive 查询操作过程严格遵守Hadoop MapReduce 的作业执行模型，Hive 将用户的HiveQL 语句通过解释器转换为MapReduce 作业提交到Hadoop 集群上，Hadoop 监控作业执行过程，然后返回作业执行结果给用户。Hive 并非为联机事务处理而设计，Hive 并不提供实时的查询和基于行级的数据更新操作。Hive 的最佳使用场合是大数据集的批处理作业，例如，网络日志分析。

>hive最初就是facebook,用来处理海量日志文件的,后来开源给了Apache

## hive的架构,包括三个组件

![][7]

![][8]

### 客户端,用户接口
>(1) cli 命令行客户端：采用交互窗口，用hive 命令行和Hive 进行通信。
(2) HiveServer2 客户端：用Thrift 协议进行通信，Thrift 是不同语言之间的转换器，是连接不
同语言程序间的协议，通过JDBC 或者ODBC 去访问Hive。
(3) HWI 客户端：hive 自带的一个客户端，但是比较粗糙，一般不用。
(4) HUE 客户端：通过Web 页面来和Hive 进行交互，使用的比较多。

### metastore元数据储存
>metastore里边保存的是hive的数据模式:表和表结构,字段类型,存储位置 有了元数据库才能使用sql进行查询  数据分析的质量控制通过元数据,分析元数据(数据量大,速度慢),分析元数据过程先要进行数据清洗,保证数据的完整准确,以确保数据分析能提供决策支持
>Hive 的元数据是一般是存储在MySQL 这种关系型数据库上的，Hive 和MySQL 之间通过MetaStore

![][9]

### 核心组建是Driver
>Hive 的核心是Driver驱动引擎，驱动引擎由四部分组成：
(1) 解释器：解释器的作用是将HiveSQL 语句转换为语法树（AST）。
(2) 编译器：编译器是将语法树编译为逻辑执行计划。
(3) 优化器：优化器是对逻辑执行计划进行优化。
(4) 执行器：执行器是调用底层的运行框架执行逻辑执行计划。

### hive和hadoop的关系
>1.HQL 中对查询语句的解释、优化、生成查询计划是由 Hive 完成的 
2.所有的数据都是存储在 Hadoop 中 的hdfs中,Hive 中的库和表可以看做是对HDFS 上数据做的一个映射。
3.查询计划被转化为 MapReduce 任务，发布到yarn上,由yarn分配资源执行,在 Hadoop 中执行（有些查询没有 MR 任务，如：select * from table） 
4.Hadoop和Hive都是用UTF-8编码的

### hive和普通数据库的异同
>**1.查询语言**。由于 SQL 被广泛的应用在数据仓库中，因此，专门针对 Hive 的特性设计了类 SQL 的查询语言 HQL。熟悉 SQL 开发的开发者可以很方便的使用 Hive 进行开发。 
2**.数据存储位置**。Hive 是建立在 Hadoop 之上的，所有 Hive 的数据都是存储在 HDFS 中的。而数据库则可以将数据保存在块设备或者本地文件系统中。 
**3.数据格式**。Hive 中没有定义专门的数据格式，数据格式可以由用户指定，用户定义数据格式需要指定三个属性：列分隔符（通常为空格、”\t”、”\x001″）、行分隔符（”\n”）以及读取文件数据的方法（Hive 中默认有三个文件格式 TextFile，SequenceFile 以及 RCFile）。由于在加载数据的过程中，不需要从用户数据格式到 Hive 定义的数据格式的转换，因此，Hive 在加载的过程中不会对数据本身进行任何修改，而只是将数据内容复制或者移动到相应的 HDFS 目录中。而在数据库中，不同的数据库有不同的存储引擎，定义了自己的数据格式。所有数据都会按照一定的组织存储，因此，数据库加载数据的过程会比较耗时。 
**4.数据更新**。由于 Hive 是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，Hive 中不支持对数据的改写和添加，所有的数据都是在加载的时候中确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用 INSERT INTO ...  VALUES 添加数据，使用 UPDATE ... SET 修改数据。 
**5.索引**。之前已经说过，Hive 在加载数据的过程中不会对数据进行任何处理，甚至不会对数据进行扫描，因此也没有对数据中的某些 Key 建立索引。Hive 要访问数据中满足条件的特定值时，需要暴力扫描整个数据，因此访问延迟较高。由于 MapReduce 的引入， Hive 可以并行访问数据，因此即使没有索引，对于大数据量的访问，Hive 仍然可以体现出优势。数据库中，通常会针对一个或者几个列建立索引，因此对于少量的特定条件的数据的访问，数据库可以有很高的效率，较低的延迟。由于数据的访问延迟较高，决定了 Hive 不适合在线数据查询。 
**6.执行**。Hive 中大多数查询的执行是通过 Hadoop 提供的 MapReduce 来实现的（类似 select * from tbl 的查询不需要 MapReduce）。而数据库通常有自己的执行引擎。 
**7.执行延迟**。之前提到，Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是 MapReduce 框架。由于 MapReduce 本身具有较高的延迟，因此在利用 MapReduce 执行 Hive 查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候，Hive 的并行计算显然能体现出优势。 
8**.可扩展性**。由于 Hive 是建立在 Hadoop 之上的，因此 Hive 的可扩展性是和 Hadoop 的可扩展性是一致的（世界上最大的 Hadoop 集群在 Yahoo!，2009年的规模在 4000 台节点左右）。而数据库由于 ACID 语义的严格限制，扩展行非常有限。目前最先进的并行数据库 Oracle 在理论上的扩展能力也只有 100 台左右。 
**9.数据规模**。由于 Hive 建立在集群上并可以利用 MapReduce 进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

![][10]

## Hive的工作原理

![][11]

>流程大致步骤为：
1. 用户提交查询等任务给Driver。
2. 编译器获得该用户的任务Plan。
3. 编译器Compiler根据用户任务去MetaStore中获取需要的Hive的元数据信息。
4. 编译器Compiler得到元数据信息，对任务进行编译，先将HiveQL转换为抽象语法树，然后将抽象语法树转换成查询块，将查询块转化为逻辑的查询计划，重写逻辑查询计划，将逻辑计划转化为物理的计划（MapReduce）, 最后选择最佳的策略。
5. 将最终的计划提交给Driver。
6. Driver将计划Plan转交给ExecutionEngine去执行，获取元数据信息，提交给JobTracker或者SourceManager执行该任务，任务会直接读取HDFS中文件进行相应的操作。
7. 获取执行的结果。
8. 取得并返回执行结果。

## SQuirrel的安装配置与使用

### 下载位置
>language Manual-->beeline CLI--->integration with SQuirrel SQL Client-->SQuirrel SQL website---.download and install --->选择版本安装即可

### 解压安装,两种方式
>1.直接双击jar包,可以引导安装
>2.提示jar包用压缩文件的方式打开安装,**==**解决办法**==**那么使用cmd命令打开  ,找到jar所在位置,执行命令java -jar jar的包名,那么就会打开安装的界面,按照提示安装即可

### 配置SQuirrel
>**1.配置Driver:** 点击driver,然后点击+号,新增一个驱动,然后将两个jar包导入,点击ListDriver自动出现Class Name,点击oj即可,jar包位置,hadoop的common包

![enter description here][12]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508771064119.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508771231653.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772368859.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772488438.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772499713.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772675604.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508773472762.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508773489594.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508773743730.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508774179375.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508774273846.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508774744806.jpg