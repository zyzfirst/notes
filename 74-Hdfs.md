HDFS

# hdfs
## Hdfs概述

>1. Hdfs是Hadoop Distributed File System 的简称，它是Hadoop实现的一个分布式文件系统。
   2. Hdfs有高容错性的特点，并且设计用来部署在低廉的硬件上；而且它提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集的应用程序。
   3. Hdfs放宽了POSIX的要求，可以以流的形式访问文件系统中的数据。
   4. Hdfs总体上采用了master/slave 架构，主要由以下几个组件组成：Client 、NameNode 、Secondary 和DataNode

## Hdfs特性

>1. 保存多个副本，且提供容错机制，副本丢失或宕机自动恢复。默认存3份。
  2. 运行在廉价的机器上。
  3. 适合大数据的处理。多大？多小？HDFS默认会将文件分割成block，128M为1个block。然后将block按键值对存储在Hdfs上，并将键值对的映射存到内存中。如果小文件太多，那内存的负担会很重。

## Hdfs的写操作


## Hdfs的API

