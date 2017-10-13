---
title: hdfs_wxd
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


## Hdfs的API

>创建文件系统的配置文件`CONF = new Configuration();`
	- 默认加载`classpath`下`core-site.xml`文件,所以需要将hadoop的配置文件拷贝下来
	- 也可以通过`CONF`的`set`方法进行设置如:`conf.set("fs.defaultFS", "hdfs://hadoop:9000");`
- 创建文件系统`hdfs = FileSystem.get(CONF);`
	- `FileSystem`是抽象类,返回值为具体的子类,如果是`HDFS`返回值的类型是`DistributedFileSystem`
	- ![](./1507819690505.png)
- 创建路径`Path path = new Path(fileName);`
- 判断路径是否存在`boolean orExist = hdfs.exists(path);`
- 判断是否是文件夹`hdfs.isDirectory(path)`
- 判断是否是文件`hdfs.isFile(path)`
- 创建输入流`FSDataInputStream input = hdfs.open(path);`
- 创建输出流`FSDataOutputStream output = hdfs.create(path);`
- 上传文件`hdfs.copyFromLocalFile(srcPath, desPath);`
- 下载文件`hdfs.copyToLocalFile(srcPath, desPath);`
- 递归删除文件`hdfs.delete(path, true);`
- 创建文件夹`boolean orSuccess = hdfs.mkdirs(path)`
- 获取目录下所有FileSatus`RemoteIterator<LocatedFileStatus> listFiles = hdfs.listFiles(path, true);`或者`FileStatus[] status = hdfs.listStatus(path);`
- Path相关
	- 通过Path获取路径`path.toString()`
	- 通过Path获取文件名`path.getName()`
- FileStatus相关
	- 通过Path生成FileStatus`hdfs.getFileStatus(path)`
	- 通过FileSatus获取Path`fileStatus.getPath()`

-
	* [概论](#概论)
		* [集群中的==角色==:也就是进程或者功能名称,](#集群中的角色也就是进程或者功能名称)
		* [==主从架构==](#主从架构)
		* [==错误信息==](#错误信息)
		* [==查看Hadoop是否启动成功==](#查看hadoop是否启动成功)
	* [hdfs](#hdfs)
		* [分布式文件存储](#分布式文件存储)
		* [HDFS优缺点](#hdfs优缺点)
			* [HDFS优点](#hdfs优点)
			* [HDFS缺点](#hdfs缺点)
	* [写读的原理](#写读的原理)
		* [原理](#原理)
			* [==写的原理==](#写的原理)
			* [==读的原理==](#读的原理)
		* [命令](#命令)

## 概论

### **集群中的==角色==**:也就是进程或者功能名称,
`hadoop`是统称

==hdfs==下面有三个角色:(主从架构的集群)
 - `namenode`(主节点)
 - `datanode`(子节点)
 - `secondarynamenode`(附属进程,一般在主节点)


==yarn==下面两个角色:(主从架构的集群)
 - `resourceManager`(主节点)
 - `nodeManager` (子节点)

**hdfs和yarn中的进程互相不影响**

### ==**主从架构**==

==公平节点的缺点:==
 - 公平的节点间的通信会出现多余的通信
 - 通信时会发生两个节点去同一个,先到的节点会锁住块,则另一个会再去别的块,就会浪费时间,会产生抢占式浪费资源

==主从架构缺点:==
 - 主节点产生故障则子节点也没有作用了

**高可用的方式用来解决主节点故障**

### ==错误信息==
  1  在hadoop中看不到实时的报错,则需要知道出现故障的原因,则在日志中可以找到原因(日志目录logs)
  
![][1]

  2 看日志从尾部看,执行`tail -200`指令,表示看后200行

  3 不抛异常,看日志级别,warn,erro等
  
  4 看进程角色和节点的hostName的log文件

### ==查看Hadoop是否启动成功==
 - web端:在浏览器中输入主机的ip或者在window中配置的hostname名加50070端口`master:50070`

![][2]

![][3]

![][4]

![][5]

![][6]

快照:(备份是为了数据安全,在生产环境下是必须的)
   虚拟机中的快照:把指定时间的的所有数据内容和状态备份和存储下来
  
 - yarn:在浏览器中输入主机的ip或者在window中配置的hostname名加8088端口`master:8088`

## hdfs
  ### 分布式文件存储
 - ==**原理**==:把文件分割,分别存到不同的块中
 - ==**块block**==:
      1.默认block存储大小为128M,不宜过大过小
	  2. block包含有id,
	  3.尺寸,
	  4. 归属哪个文件,
	  5.块存储的位置
	  6.散布在集群中的不同的节点上
 - ==**元数据**==(描述数据的数据,还可以进行数据分析,质量控制)
   1 在数据库中存储元数据的表为==数据字典表==
   2 数据库中记录表中数据的为==元组==
   3 关系型数据库中有==模式==和==元组==
   4 nosql中只有==元组==
 
 - ==**特点**:==
 1.保存多个副本
     - 没有副本的话就会产生读取失败,会出现不稳定,风险性高,在生产环境下就是可用性低
     - 副本的方式来解决稳定性问题,其中一个节点出现故障,hdfs就会去找它的副本,再转到副本上操作
	 
   2.副本保存原则
     - 存储在不同的物理节点上()
     - 机架感知(第一个副本保存在一个机架后,其他副本会优先在其他机架中备份)
     - 副本丢失或宕机自动恢复。默认存3份
     - 运行在廉价的机器上。
     - 适合大数据的处理,不宜放过多的小文件,大量小文件会加重内存负担


   3.运行架构

![][7]

 - `namenode`
   1.处理请求,
   2.分配处理任务,
   3.负责心跳连接(namenode 发送的心跳信号,如果datanode响应就返回信号,如果没有响应就出故障了),
   4.均衡(自动均衡,手动均衡),
   5.副本
   6.==.负荷最重==
   7.==最重要的是元数据的保存处理==:
  

``` dns
A. 元数据的保存路径
  1.fsimage文件存元数据的具体内容:
    - 文件名称
    - 文件的大小
    - 路径
    - 创建者
    - 时间
    - 文件的分块信息
  2.edits存元数据的操作日志(以操作记录的形式存在edits里面)
    - 创建文件
    - 删除,修改
  3.每一个block的位置:在集群中的哪一个节点上(datanode提供)
```
  ![][8]
	  
``` gams
B.namenode启动过程:
  1 读取加载fsimage(在磁盘中)到内存
  2 加载edits
  3 存储文件时,文件的元数据以事务的过程把元数据信息放到两个地方(内存,edits:元数据更新的操作记录和操作内容)
  4 第二次启动时, 读取加载fsimage到内存, 加载edits,按edits里面的操作过程再操作一遍,
  5 启动完后,内存里会有一个最全的最新的元数据数据,则会删除fsimage,再新建一个fsimage,把元数据的数据保存到里面
  6 然后再把edits删除,再创建一个空的edits,然后运行的话就会把元数据放如edits中
  7 每次重新启动 都会循环
  8 每个元数据的更新都会保存到新的edits中
  9 长时间的不启动会造成edits里面内容非常大
```

	
	 

``` gams
 C.secondarynamenode:
   1 会监控namenode
   2 当edits中数据量达到一定量时,拷贝到secondarynamenode
   3 在secondarynamenode创建一个新的edits
   4 然后还会把fsimage拷贝到secondarynamenode
   5 再把edits和fsimage合并,包括程序的内存和所有元数据的内容
   6 再把合并后的放入新建的fsimage,再把这个fsimage给namenode中的fsimage
   7 safe model保证了在启动过程中不会被操作,启动结束后safe model会退出,就可以对元数据进行操作了
```



``` lsl
D.元数据不在
  1 当HDFS启动,会启动namenode 还会启动datanode
  2 datanode启动后会自动检查data.dir目录下的所有的block信息
  3 汇报给namenode
```
			
 - `datanode`
    1.数据的读写请求执行
    2.数据的保存操作
 - `SecondaryNameNode`
    1.元数据的备份
    
	### HDFS优缺点
	
 #### HDFS优点
 - 高容错性
    1.数据会自动保存多个副本(如果其中节点出现故障,则会找副本,如果发现副本没有两个,就会再找个节点,保存一份副本)
    2.副本丢失后,自动回复
 - 适合批处理
    1.移动的计算和操作
    2.数据位置暴露

 - 适合大数据的处理
    1.GB,TB，PB甚至更大
    2.百万规模以上的文件数量
    3.lOK＋节点
 - 
  #### HDFS缺点
  

 - 低延迟数据访问
    1.   毫秒级读取
    2.   低延迟与高吞吐量

 - 小文件存取(不能过多)
     1.占用NameNode内存空间
     2.寻址时间超过读取时间
 - 并发写入、文件随即修改(不支持)
     1. 一个文件同时只能由一个写入者
     2.  仅支持append(指追加)
     3.  不支持随机修改,只支持顺序写

## 写读的原理
### 原理


>####  ==写的原理==

![][9]

> 当客户端要存储文件时,Hadoop集群收到请求,会发送给namenode信息(元数据的信息),然后namenode会根据元数据的信息和分的块的信息分配节点,再返回给客户端节点的uil,客户端拿到uil后就会一个一个块地进行发送,找到相应的节点,然后第一个节点收到第一个块后存储一份再给第二个节点发送,第二个节点也存储一份,继续发送给第三个节点,在第一个节点把第一个块给第二个节点的时候,第二个块就又开始进行发送,就这样循环到所有数据都存储备份到不同的节点中.当节点接收到块时还会响应给namenode,这样就不会存在有故障的节点中了.全部文件存储成功后namenode会响应给客户端信息表示完成

> 在块到节点进行传送的时候是先把块分为一个一个数据包(数据包一般小于64k),然后进行传送

>当客户端为外部的就是上面的步骤,当客户端为某个机架内部的时候,则请求发送到namenode后, namenode会先在发送请求的机架中找其他的可用的datanode,然后再备份到其他的机架中的datanode中,然后第二个块也是相同的分法,但是本机架中已经无其他节点的时候就会直接存在其他机架中,读也是一样的

>#### ==读的原理==

![][10]

>HDFS采用的是“一次写入多次读取”的文件访问模型。一个文件经过创建、写入和关闭之后就不需要改变。这一假设简化了数据一致性问题，并且使高吞吐量的数据访问成为可能。
>当客户端要读文件时,发送请求到Hadoop,然后把文件名给namenode,namenode得到文件名后去找文件块存放的最近的节点,然后列成列表(列表是以块1在哪个节点上有存储,块2在哪个节点上有存储···即block1:host2,host1,host3      block2:host7,host8,host4 )给了客户端,客户端拿着列表去访问对应的节点,然后读取元数据,最后就得到了文件

### 常用命令

``` java
1.一般命令
hdfs dfs -ls /  查看根目录下的文件
hdfs fs -ls -R /  递归查看根目录下的文件
hdfs dfs -mkdir /test01    在HDFS中必须是绝对路径,在根目录下新建一个文件夹
hdfs dfs -put a.txt /test01/   斜杠要加,上传本地的文件到HDFS的目录下
hdfs dfs -get /user/hadoop/a.txt/   从HDFS上下载文件
hdfs fs -cp src dst   拷贝HDFS上的文件到HDFS上的某个文件下,只能拷贝HDFS
hdfs dfs -df   查看磁盘利用情况
hdfs dfs -mv src dst
hdfs dfs -cat /user/hadoop/a.txt
hdfs dfs -rm /user/hadoop/a.txt
hdfs dfs -rmr /user/hadoop/a.txt
hdfs dfs -text /user/hadoop/a.txt
hdfs dfs -copyFromLocal localsrc dst 与hadoop fs -put功能类似。
hdfs dfs -moveFromLocal localsrc dst 将本地文件上传到hdfs，同时删除本地文件。
2.hdfs dfsadmin  文件系统管理
hdfs dfsadmin -report
hdfs dfsadmin -safemode enter | leave | get | wait
hdfs dfsadmin -setBalancerBandwidth?1000
3.hdfs fsck  查看文件使用状况
4.?start-balancer.sh
负载均衡,可以使DataNode节点上选择策略重新平衡DataNode上的数据块的分布
```


  [1]: https://www.github.com/wxdsunny/images/raw/master/1507688049872.jpg
  [2]: https://www.github.com/wxdsunny/images/raw/master/1507688964607.jpg
  [3]: https://www.github.com/wxdsunny/images/raw/master/1507689071769.jpg
  [4]: https://www.github.com/wxdsunny/images/raw/master/1507689314125.jpg
  [5]: https://www.github.com/wxdsunny/images/raw/master/1507689380660.jpg
  [6]: https://www.github.com/wxdsunny/images/raw/master/1507689469076.jpg
  [7]: https://www.github.com/wxdsunny/images/raw/master/1507694087255.jpg
  [8]: https://www.github.com/wxdsunny/images/raw/master/1507708635960.jpg
  [9]: https://www.github.com/wxdsunny/images/raw/master/1507725380308.jpg
  [10]: https://www.github.com/wxdsunny/images/raw/master/1507727862816.jpg