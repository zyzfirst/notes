---
title: Day-18 Flume
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 大数据数据采集

数据来源形式         数据来源格式             文件传输
网络爬虫                 html文档 f lume、     kaf ka
日志数据 ,log文件   日志流 f lume、           kaf ka
业务数据                  关系型数据库           sqoop
传感数据                  数据流                     kaf ka

# flume
Flume是Cloudera提供的一个高可用的(无单节点)，高可靠的(中断传输可以继续)，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

## flume采集数据的来源:
- log文件
- 网络端口
- 消息队列

## flume的数据发送来源

- hdfs
- hive
- hbase
- strom
- 网络端口
- 消息队列

## telnet的使用

yum list telnet   列出当前的telnet网络端口,用来测试
yum install 选择一个版本           安装telnet
telnet localhost 44444      选择端口启动telnet(远程登录)
然后可以在控制台输出内容
启动flume的命令: flume-ng agent -c conf -f flume_avro_log.conf --name al    后便可以加上default信息

![enter description here][1]

![enter description here][2]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509979333316.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509979457372.jpg