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