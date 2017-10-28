---
title: Day-15 Zookeeper 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 安装与配置
- 配置主机名与ip
- 解压与配置环境边量
- 复制与配置zoo.cfg
- 拷贝文件与配置项到其他节点
- 运行zookeeper与状态检查

>三台服务器：
192.168.15.5 jokeros1
192.168.15.6 jokeros2
192.168.15.7 jokeros3
在每台服务器的host中添加：vi /etc/hosts
192.168.15.5 jokeros1
192.168.15.6 jokeros2
192.168.15.7 jokeros3


>随便在某一台上如：192.168.15.5
解压zookeeper压缩文件：
tar –zxvf zookeeper-3.4.8.tar.gz
配置环境变量：vi /etc/profile
#zookeeper
export ZOOKEEPER=/usr/tools/zookeeper-3.4.8
export PATH=$PATH:$ZOOKEEPER/bin
使修改生效：
source /etc/profile

>到zookeeper的conf目录下面，新增一个zoo.cfg文件
cp zoo_sample.cfg zoo.cfg
然后进去自己创建的zoo.cfg文件修改：
dataDir=/usr/tools/zookeeper-3.4.8/data
添加：
server.1=jokeros1:2888:3888
server.2=jokeros2:2888:3888
server.3=jokeros3:2888:3888

>配置完以后将上述内容全部拷贝到另外两台服务的相同位置
使用scp 赋值目录需要递归,且不能有空格,如果是文件就不用-r了
scp -r/usr/tools/zookeeper-3.4.8 root@jokeros2: /usr/tools/
scp /usr/tools/zookeeper-3.4.8 root@jokeros3: /usr/tools/
在这个dataDir=/usr/tools/zookeeper-3.4.8/data目录下创建文件
三台机器下面的data目录里面各自建一个myid的文件
然后里面填上相应的数字
如jokeros1是server.1，里面的数字是1
Jokeros2是server.2，里面的数字是2
/etc/profile环境变了也可以用scp来完成，或者可以各自修改成一致的

>三台分别启动zookeeper
zkServer.sh start
每台机器上查看状态：
zkServer.sh status
结果：
ZooKeeper JMX enabled by default
Using config: /usr/tools/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower
使用jps查看：
jps
结果
QuorumPeerMain

>Leader选举小结
 1.server启动时默认选举自己，并向整个集群广播
2.收到消息时，通过3层判断：选举轮数，zxid，server id大小判断是否同意对方，如果同意，则修改自己的选票，并向集群广播
3.QuorumCnxManager负责IO处理，每2个server建立一个连接，只允许id大的server连id小的server，每个server启动单独的读写线程处理，使用阻塞IO
4.默认超过半数机器同意时，则选举成功，修改自身状态为LEADING或FOLLOWING
5.Obserer机器不参与选举

# 概论
>ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。
ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

# 解决问题
>Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等

# 架构
>每个server以2181端口对外提供服务,连接的显示是localhost,其实每个server互相通讯,拥有共同且全部的资源

![][1]

# 数据结构

![][2]

Zookeeper 这种数据结构有如下这些特点：
- 1.每个子目录项如 NameService 都被称作为 znode，这个 znode 是被它所在的路径唯一标识，如 Server1 这个 znode 的标识为 /NameService/Server1
- 2.znode 可以有子节点目录，并且每个 znode 可以存储数据，注意 EPHEMERAL 类型的目录节点不能有子节点目录,临时节点没有子节点
- 3.znode 是有版本的，每个 znode 中存储的数据可以有多个版本，也就是一个访问路径中可以存储多份数据
- 4.znode 可以是临时节点，一旦创建这个 znode 的客户端与服务器失去联系，这个 znode 也将自动删除，Zookeeper 的客户端和服务器通信采用长连接方式，每个客户端和服务器通过心跳来保持连接，这个连接状态称为 session，如果 znode 是临时节点，这个 session 失效，znode 也就删除了
- 5.znode 的目录名可以自动编号，如 App1 已经存在，再创建的话，将会自动命名为 App2 ,命令是create -s /bd14/app  即统一命名服务,必须由-s才会启动这个服务,给你的节点自动增加编号
- 6.znode 可以被监控，包括这个目录节点中存储的数据的修改，子节点目录的变化等，一旦变化可以通知设置监控的客户端，这个是 Zookeeper 的核心特性，Zookeeper 的很多功能都是基于这个特性实现的(watch,方法的调用)

# 具体的服务

## 统一命名服务
>分布式系统的zookeeper具有完善的命名服务,内置的name service,利用树形名称结构,是一个有层次的目录结构,保证名字不会重复且对人有好,而我们只需要实现一个create接口即可,调用方法 create -s '/bd14/app'会返回带有编号的目录

## 配置服务
>对于分布式系统来说多台pc server必须保证配置相同以便出现问题容易找出问题,所以单个配置在scp的方式比较麻烦,因此可以把配置信息保存在zookeeper的目录节点,像这样的配置信息完全可以交给 Zookeeper 来管理，将配置信息保存在 Zookeeper 的某个目录节点中，然后将所有需要修改的应用机器监控配置信息的状态，一旦配置信息发生变化，每台应用机器就会收到 Zookeeper 的通知，然后从 Zookeeper 获取新的配置信息应用到系统中。

![][3]

## 集群管理
>zookeeper帮我们管理集群,通过两种方式,一是利用选举机制选出leader管理集群,二是监控机制,当有节点挂掉的时候会替换原来的节点,避免了单节点出现故障的问题(节点坏掉调用wath方法)

### 原理概括
>Zookeeper的核心是原子广播，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。
为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。
每个Server在工作过程中有三种状态：
LOOKING：当前Server不知道leader是谁，正在搜寻
LEADING：当前Server即为选举出来的leader
FOLLOWING：leader已经选举出来，当前Server与之同步

#### Leader主要有三个功能：
- 恢复数据；
- 维持与Learner的心跳，接收Learner请求并判断Learner的请求消息类型；
- Learner的消息类型主要有PING消息、REQUEST消息、ACK消息、REVALIDATE消息，根据不同的消息类型，进行不同的处理。

#### Follower主要有四个功能：
- 向Leader发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）；
- 接收Leader消息并进行处理；
- 接收Client的请求，如果为写请求，发送给Leader进行投票；
- 返回Client结果。

###  队列管理

>Zookeeper 可以处理两种类型的队列：
1.当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列。
2.队列按照 FIFO 方式进行入队和出队操作，例如实现生产者和消费者模型。
同步队列用 Zookeeper 实现的实现思路如下：
创建一个父目录 /synchronizing，每个成员都监控标志（Set Watch）位目录 /synchronizing/start 是否存在，然后每个成员都加入这个队列，加入队列的方式就是创建 /synchronizing/member_i 的临时目录节点，然后每个成员获取 / synchronizing 目录的所有目录节点，也就是 member_i。判断 i 的值是否已经是成员的个数，如果小于成员个数等待 /synchronizing/start 的出现，如果已经相等就创建 /synchronizing/start。

![][4]

## 常用的命令

- zkCli.sh  连接zookeeper的服务端 ,对外端口2181
- help  产看命令帮助
- ls /hbase 查看目录结构
- get /habase/master 得到目录下的内容, 以二进制形式存储信息,如果get的是一个目录那么get到是空,若不是的话,是一个文件 那么就会得到内容,相当于文件夹和文件的关系
- create /bd14 aa 创建子节点 参数(保存的内容)  可以在/bd14后加子目录即子节点
- set /bd14 zyyz  set可以向节点中set内容
- delete /bd14  删除子节点,如果这个节点有子节点,那么会提示 Node not empty 只能删除空的节点
- rmr /bd14  这个删除是递归删除
- getAcl /hbase  查看节点的权限 cdrwa 创建删除
- create -e /empnodetest zyz  创建临时节点,可以保存数据,不能有子节点
- create -s /bd14/app1 '' 统一命名服务,名称不会重复 会自动在app后边加序号,所以能重读执行
- ls /hbase/rs  监听哪些zookeeper节点在运行

  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509194795456.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509195688644.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509196824360.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509198158493.jpg