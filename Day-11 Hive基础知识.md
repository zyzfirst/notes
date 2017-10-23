---
title: Day-11 Hive基础知识
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

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

![enter description here][4]

![enter description here][5]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508771064119.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508771231653.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772368859.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772488438.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508772499713.jpg