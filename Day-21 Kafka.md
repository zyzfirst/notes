---
title: Day-21 Kafka
tags: 安装,架构,API
grammar_cjkRuby: true
---


# kafka的安装(0.10.1.1版本)
![安装流程][1]

- 解压安装(tar -xvf name)
- 配置环境变量(KAFKA_HOME和PATH)
- 配置文件server.properties(broker.id不能重复,跟zookeeper的myid类似,必须保证唯一,而log.dir和listeners只要保证在同一个节点上的broker不同即可)

## 测试是否安装成功
上图的命令,& 符号是切出当前窗口,因为master节点有两个broker需要启动,启动一个就把窗口站住了,所以用这种方式,kafka的启动适合参数一块启动和flume相似,也需要跟参数同时加载启动

## 安装问题
>1. /etc/profile 配置出错,导致命令找不到,在其他没问题的节点上,输入whereis vi 得到绝对命令,/bin/vi /etc/profile  使用绝对命令,进入编辑profile的文件,修改.然后/sbin/reboot 重启虚拟机,使得刚才的配置生效

![command cannot find][2]

>2. 安装完成,启动服务,报错说broker.id和已经存在,修改server.properties的broker.id

>3.提示broker.id和meta properties NOT match不匹配,那么进入server.properties查看log.dirs查看日志目录,修改meta propeties里的broker.id 和server.properties里的broker.id一样

# kafka的原理与架构

## 特性
Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写，它以可水平扩展和高吞吐率而被广泛使用。
>横向扩展(分布式开发,麻烦,难以维护) 纵向扩展(单节点增强功能,昂贵,好维护)

## 背景
Kafka是一个消息系统，原本开发自LinkedIn，用作LinkedIn的活动流（Activity Stream）和运营数据处理管道（Pipeline）的基础。现在它已被多家不同类型的公司 作为多种类型的数据管道和消息系统使用。
活动流数据是几乎所有站点在对其网站使用情况做报表时都要用到的数据中最常规的部分。活动数据包括页面访问量（Page View）、被查看内容方面的信息以及搜索情况等内容。这种数据通常的处理方式是先把各种活动以日志的形式写入某种文件，然后周期性地对这些文件进行统计分析。运营数据指的是服务器的性能数据（CPU、IO使用率、请求时间、服务日志等等数据)。

![应用场景][3]

可以处理流数据和批量数据,目的地可以是hdfs也可以使spark等流处理工具
>kafka是消息系统,用于收集数据和数据处理的中间过程,它由于是时间复杂度为0(1),所以快速的提供读写服务

![信息系统][4]

## 简介
Kafka是一种分布式的，基于发布/订阅的消息系统。主要设计目标如下：
- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。复杂度为0(1),只提供顺序读写,不能随即读写

![时间复杂度][5]

- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒100K条以上消息的传输。
- 支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。
- 同时支持离线数据处理和实时数据处理。
- Scale out：支持在线水平扩展。

## 分布式存在问题
分布式必然要解决 :容错性,一个节点挂掉,那么数据就会丢失,解决容错性有两种方式:一是副本replaction,另一个是wal(日志记录)      在kafka中,副本是有主次 的,祝福本负责数据的传输,副副本负责和主副本同步(一般是3个副本),在创建topic的时候就会选出谁是主副本

![主副副本][6]

# 一些常用的指令
![指令][7]

## 创建topic,需要连接zookeeper的2181服务端口

-  kafka-topics.sh  --zookeeper master:2181 --create --partition 2 --replication-factor 3 --topic bd14first
>需要指定分几个区和存储副本数量以及名字 2181端口后边是参数,无所谓顺序,但是这三部分必须包含

- kafka-topics.sh  --zookeeper master:2181 --list        
>查看所有的topic

- kafka-topics.sh --zookeeper master:2181 --describe --topic bd14first
>查看某个topic的描述

![topic描述][8]

## 开启producer服务,向topic传输数据,设计到数据,需要指定具体节点的端口

- kafka-console-producer.sh  --broker-list master:9092,master:9093,slaver1:9092 --topic bd14first
>端口必须是物理节点的服务端口,必须指定topic

## 开启consumer拂去,接收topic的数据
- kafka-console-consumer.sh  --bootstrap-server master:9093 --topic bd14first
>可以指定物理节点的服务端口和topic(官方推荐)

- kafka-console-consumer.sh --zookeeper master:2181 --topic bd14second
>也可以指定zookeeper的2181服务端口(不推荐)

- kafka-console-consumer.sh  --bootstrap-server master:9093 --topic bd14first --from-beginning
>可以设定从开启produce,传输的数据都会打印出来

- kafka-console-consumer.sh --bootstrap-server master:9093 --topic bd14first --offset 'earliest' --partition 0
>也可以这样设置,不过必须指定partition,因为每个partition都有自己的offset

## 删除topic,两种方式

- kafka-topics.sh --zookeeper master:2181 --delete --topic bd14first

![删除topic][9]

![提示][10]

- 第二种   zkCli.sh  进入zookeeper查看当前运行的kafka的broker

![enter description here][11]

![enter description here][12]

## kafka属于信息队列

### 消息队列的优点
- 解耦
在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
- 冗余
有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。
- 扩展性
因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。
- 灵活性 & 峰值处理能力
在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
- 可恢复性
系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
- 顺序保证
在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。
- 缓冲
在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。
- 异步通信
很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

### 常用Message Queue(消息队列)对比

- RabbitMQ
RabbitMQ是使用Erlang编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP, SMTP, STOMP，也正因如此，它非常重量级，更适合于企业级的开发。同时实现了Broker构架，这意味着消息在发送给客户端时先在中心队列排队。对路由，负载均衡或者数据持久化都有很好的支持。
- Redis
Redis是一个基于Key-Value对的NoSQL数据库，开发维护很活跃。虽然它是一个Key-Value数据库存储系统，但它本身支持MQ功能，所以完全可以当做一个轻量级的队列服务来使用。对于RabbitMQ和Redis的入队和出队操作，各执行100万次，每10万次记录一次执行时间。测试数据分为128Bytes、512Bytes、1K和10K四个不同大小的数据。实验表明：入队时，当数据比较小时Redis的性能要高于RabbitMQ，而如果数据大小超过了10K，Redis则慢的无法忍受；出队时，无论数据大小，Redis都表现出非常好的性能，而RabbitMQ的出队性能则远低于Redis。
- ZeroMQ
ZeroMQ号称最快的消息队列系统，尤其针对大吞吐量的需求场景。ZeroMQ能够实现RabbitMQ不擅长的高级/复杂的队列，但是开发人员需要自己组合多种技术框架，技术上的复杂度是对这MQ能够应用成功的挑战。ZeroMQ具有一个独特的非中间件的模式，你不需要安装和运行一个消息服务器或中间件，因为你的应用程序将扮演这个服务器角色。你只需要简单的引用ZeroMQ程序库，可以使用NuGet安装，然后你就可以愉快的在应用程序之间发送消息了。但是ZeroMQ仅提供非持久性的队列，也就是说如果宕机，数据将会丢失。其中，Twitter的Storm 0.9.0以前的版本中默认使用ZeroMQ作为数据流的传输（Storm从0.9版本开始同时支持ZeroMQ和Netty作为传输模块）。
- ActiveMQ
ActiveMQ是Apache下的一个子项目。 类似于ZeroMQ，它能够以代理人和点对点的技术实现队列。同时类似于RabbitMQ，它少量代码就可以高效地实现高级应用场景。
- Kafka/Jafka
Kafka是Apache下的一个子项目，是一个高性能跨语言分布式发布/订阅消息队列系统，而Jafka是在Kafka之上孵化而来的，即Kafka的一个升级版。具有以下特性：快速持久化，可以在O(1)的系统开销下进行消息持久化；高吞吐，在一台普通的服务器上既可以达到10W/s的吞吐速率；完全的分布式系统，Broker、Producer、Consumer都原生自动支持分布式，自动实现负载均衡；支持Hadoop数据并行加载，对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka通过Hadoop的并行加载机制统一了在线和离线的消息处理。Apache Kafka相对于ActiveMQ是一个非常轻量级的消息系统，除了性能非常好之外，还是一个工作良好的分布式系统。

# kafka架构
- Broker
Kafka集群包含一个或多个服务器，这种服务器被称为broker,一个节点上可以有多个broker,id,log,必须不一样
- Topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
- Partition
Parition是物理上的概念，每个Topic包含一个或多个Partition.每个partition都有自己的offset
- Producer
负责发布消息到Kafka broker,可以指定partition发布消息
- Consumer
消息消费者，向Kafka broker读取消息的客户端。一个consumer可以从不同的topic出消费
- Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。属于group的consumer不会重复消费同一个topic,一个topic要想有不同的去处,即存储hdfs,用于计算等等,只需要指定不同的分组即可,每个组会去不消费topic但是组内不会重复消费

![架构图][13]

## kafka的数据存储
被append的数据进入partition中,会被顺序写进磁盘,即本地的/tmp/kafka.logs--- .log中,因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证)

![partition格式][14]

- 此外,当消息被消费了,数据也不会删除,只是offset的指针位置发生改变,所以一般不会重复读,少读
- 两种从磁盘删除数据的方式

对于传统的message queue而言，一般会删除已经被消费的消息，而Kafka集群会保留所有的消息，无论其被消费与否。当然，因为磁盘限制，不可能永久保留所有数据（实际上也没必要），因此Kafka提供两种策略删除旧数据。一是基于时间，二是基于Partition文件大小。例如可以通过配置$KAFKA_HOME/config/server.properties，让Kafka删除一周前的数据，也可在Partition文件超过1GB时删除旧数据，配置如下所示。

``` stylus
　　
# The minimum age of a log file to be eligible for deletion
log.retention.hours=168
# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824
# The interval at which log segments are checked to see if they can be deleted according to the retention policies
log.retention.check.interval.ms=300000
# If log.cleaner.enable=true is set the cleaner will be enabled and individual logs can then be marked for log compaction.
log.cleaner.enable=false
```




  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510227679429.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510226480340.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510229014526.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510229217531.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510230691538.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510229576871.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510241288734.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510241919678.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510243453004.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510243473636.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510243550321.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510243562429.jpg
  [13]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510327714466.jpg
  [14]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510328162305.jpg