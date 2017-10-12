---hadoop
title:hadoop第一天 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# Hadoop

## hadoop有三个主要的框架

>hdfs分布式文件存储系统.mapreduce是分布式文件解析系统,yarn是资源调度系统,起初是在mapreduce中负责给mapreduce分配cpu资源和容器资源和内存资源,后来2.x以后分离出来,不仅可以管理mapreduce还可以管理spark(迭代内存计算)和storm(实时流式计算),提供了对其他框架的支持

# 数据存储方式
>  两种方式,第一种是nfs,是网络文件系统,是通过tcp/ip协议通过网络共享,缺点:不能高可用,不能高并发,单个节点访问量过高等问题.另一种方式是hdgs,分布式文件存储系统,是多个集群中分了多个block,而每个block中有多个副本,那么就解决了高可用和高并发,在哪个里边都可以访问到资源,一个block中损坏可以使用其他的block,namenode是存放的映射关系,找到对应的datanode.

> 数据读取方式,两中方式,第一种不现实,用一台计算机去读取,需要内存很大,读取的结果存放在内存中,第二种方式是mapreduce的方式,写代码,发布到各个节点,各个数据节点去依次计算自己的所有block,计算完成后返回reduce只返回结果,所以比较小,可以返回多个reduce进行并发操作

# 图形化界面和服务器界面的转化

## 在root用户权限下

>root和用户的相互转换,root--->到用户 su - 用户名    用户--->root  su 命令,很简单
>执行命令init 3 转换为命令行界面 init 5 转换成图形化界面 ,注意这种转换是暂时的,当重启系统的时候就会变成默认的界面,所以要想开机就是某种界面需要设置一下,执行命令 vi /etc/inittab 3代表命令行5代表图形化

## 在用户权限下

>1 . 需要先root赋予执行sudo操作的权限,执行命令 vi /etc/sudoers 赋值root All All 改为用户名zyz,这样就有了执行sudo操作的权限
>2 . 执行sudo操作,执行命令 sudo init 3 or 5  ,同样要想开机也生效,那么需要修改配置文件,执行命令vi /etc/inittab

## 修改主机名

>执行命令 vi /etc/sysconfig/network  将HOSTNAME=主机名 ,然后不会生效,需要执行命令 hostname 主机名,然后重新连接一下即可,重新连接并没有重启系统
>要想通过主机名访问资源,需要修改host文件,使ip和主机名有映射关系 ,执行命令 vi ./etc/hosts 添加域名和ip即可如 :192.168.162.103 hadoop 这样就映射完毕,可以通过主机名访问资源,要i向在window中访问虚拟机中的系统的东西,还需要在window中修改host文件,建立映射关系,在window>system32>driver>tec>host  然后添加域名和ip即可

## 集群的安装流程

### 1.配置服务器
>1个主节点：master(192.168.162.103)，2个（从）子节点，slaver1(192.168.162.104)，slaver2(192.168.162.105)
配置主节点名(192.168.162.103)
vi /etc/sysconfig/network
添加内容：
NETWORKING=yes
HOSTNAME=master
配置两台子节点名(192.168.162.104)和(192.168.162.105)
vi /etc/sysconfig/network
添加内容：
NETWORKING=yes
HOSTNAME=slaver1
vi /etc/sysconfig/network
添加内容：
NETWORKING=yes
HOSTNAME=slaver2
在三个节点上分别操作,可以使用xshell 的快捷方式,将命令发送到所有的窗口
2配置hosts
打开主节点的hosts文件，要将文件的前两行注释掉 (注释当前主机的信息)并在文件中添加所有hadoop集群的主机信息。
vi /etc/hosts
192.168.162.103   master
192.168.162.104  slaver1
192.168.162.105   slaver2
保存之后，将主节点的hosts分别拷贝到其他两个子节点
scp /etc/hosts root@192.168.162.104:/etc/
scp /etc/hosts root@192.168.162.105:/etc/
然后分别执行(重启服务器也可以不执行下面的语句): ./bin/hostsname hostname
例如：master上执行 ./bin/hostsname master，使之生效。相当于执行开启关闭服务,用./ ,由于之前配置过环境变量,那么在任何path中都可以找到此命令,所以可以直接执行命令 hostname master即可,也可以重启服务器

### 2.配置ssh免密码访问

>1. 生成公钥密钥对
在每个节点上分别执行：
ssh-keygen -t rsa ,一直按回车直到生成结束.执行结束之后每个节点上的/root/.ssh/目录下生成了两个文件 id_rsa 和 id_rsa.pub,其中前者为私钥，后者为公钥.
2. 在主节点上执行：
cp id_rsa.pub authorized_keys(目录在当前目录)
将子节点的公钥拷贝到主节点并添加进authorized_keys
将两个子节点的公钥拷贝到主节点上，分别在两个子节点上执行：
scp /root/.ssh/ id_rsa.pub root@master:/root/.ssh/id_rsa_slaver1.pub
scp /root/.ssh/ id_rsa.pub root@master:/root/.ssh/id_rsa_slaver2.pub
然后在主节点上，将拷贝过来的两个公钥合并到authorized_keys文件中去
主节点上执行：
cat id_rsa_slaver1.pub>> authorized_keys
cat id_rsa_slaver2.pub>> authorized_keys
3. 这里的配置方式可以有多种操作步骤，最终目的是每个节点上的/root/.ssh/authorized_keys文件中都包含所有的节点生成的公钥内容。
将主节点的authorized_keys文件分别替换子节点的authorized_keys文件
主节点上用scp命令将authorized_keys文件拷贝到子节点的相应位置
scp authorized_keys root@slaver1:/root/.ssh/
scp authorized_keys root@slaver2:/root/.ssh/
4.  最后测试是否配置成功
在master上分别执行
ssh slaver1
ssh slaver2
能正确跳转到两台子节点的操作界面即可，同样在每个子节点通过相同的方式登录主节点和其他子节点也能无密码正常登录就表示配置成功。
5. 原理解析
![][1]


 >安门的过程就是将公钥追加到authorized_keys中的过程,所以达到3的最终目的就能实现各个节点免密登录
 
 ### 3.安装jdk
 
 >1. 卸载jdk
查看系统已经装的jdk： 
rpm -qa|grep jdk
卸载jdk:
rpm -e --nodeps java-1.6.0-openjdk-javadoc-1.6.0.0-1.66.1.13.0.el6.x86_64
安装JDK（三台机器都要安装）
安装在同一位置/opt/java/jdk1.7.0_72
下载JDK
解压JDK ： tar -zxvf /opt/java/jdk-7u72-linux-x64.gz
配置环境变量, 编辑profile文件：
vi /etc/profile
>2.配置环境变量 vi /etc/profile
在profile文件末尾添加以下代码:
export JAVA_HOME=/opt/java/jdk1.7.0_72
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
Hadoop和其他的一些软件也是一样的配置环境变量,java_home配路径,path中加bin,bin里面是可执行的脚本,因为hadoop的可执行的开启关闭服务在sbin中,所以path需要配置sbin,而hadoop的一些命令在bin中,所以也要配置bin,这样就配置完毕了
保存后，使刚才编辑的文件生效：profile是shell脚本文件,修改完成需要重新加载一下,使其生效.
source /etc/profile
测试是否安装成功：java –version

### 4.安装hadoop

>和jdk的安装类似,直接解压,然后配置环境变量,配置path,上边已经提到了bin和sbin,所以配置一下即可.

### 5. 配置hadoop

>1. 首先前提是几个节点已经能相互ping通,并且免密登录,然后需要配置一下
>2. ![][2]


  >3. 配置core-site.xml
  <configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
        </property>
		// 配置默认文件系统,为hdfs并且指定主节点为192.168.162.103:9000
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
		//配置缓冲区的大小,
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/temp</value>
        </property>
		// 指定数据存放位置的目录
        <property>
                <name>hadoop.proxyuser.root.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.groups</name>
                <value>*</value>
        </property>
		// 配置执行权限
</configuration>
configuration中的配置格式就是property然后两个自标签name和value
>4.配置hdfs-site.xml 
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>master:9001</value>
        </property>
		// 主节点的进程,用来监督各个子节点的情况,高可用的实现,死掉就换其他的
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/usr/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/dfs/data</value>
        </property>
		//  配置namenode和datanode的文件存放位置
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
		// 配置备份副本的数量,默认是3,一般不大于子节点的数量
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
		// 启动web服务,可以通过浏览器查看hdfs的状态
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
		// 权限的控制,false的话意味着不控制权限,可以通过window等访问在linux中的hadoop
        <property>
                <name>dfs.web.ugi</name>
                <value> supergroup</value>
        </property>
		// 配置web的权限
</configuration>
>5.配置mapred-site.xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
		// 配置mapreduce在yarn框架上运行
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>master:19888</value>
        </property>
		//开启端口来记录mapreduce的处理过程,方便有问题了去查看
</configuration>
>6.配置yarn-site.xml
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
		//配置mapreduce的执行乱序
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value> master:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value> master:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value> master:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value> master:8088</value>
        </property>
		//指定各项服务的端口
</configuration>
>7.配置slaves
slaver1
slaver2
master
配置mastr,那么主节点即是namenode也是datanode,进程会多出两个nodemanager和datanode,以及自身的resourcemanager,namenode,sencond(监督) 和同游的jps
>8. 小结
 ![][3]


  ### 6.配置分节点
  
  >1.拷贝hadoop文件夹
 主节点上执行：
scp -r /usr/hadoop-2.6.4 root@slaver1:/usr
scp -r /usr/hadoop-2.6.4 root@slaver2:/usr
>2.拷贝profile到子节点,即环境变量
主节点上执行：
scp /etc/profile root@slaver1:/etc/
scp /etc/profile root@slaver2:/etc/
在两个子节点上分别使新的profile生效：
source /etc/profile

### 7.格式化和启动

>1.格式化,主节点上执行命令 hadoop namenode -format      新版本可以使用命令hdfs namenode -format 
提示：successfully formatted表示格式化成功
>2.启动hadoop,可以start-all.sh全部启动jps查看那进程,也可以分开启动,
先开启start-dfs.sh会启动NameNode,SecondaryNameNode,datanode
再开启start-yarn.sh 会有ResourceManager,NodeManager两个进程,属于yarn
>3.关闭stop-all.sh 关闭所有,若不清楚可以访问sbin目录查看.

### 8.通过windows访问linux的hadoop

>1.先在linux中关闭防火墙,因为需要访问端口50070,所以执行命令service iptables stop  chkconfig iptables off 开机不自启
>2.如果通过主机名访问hadoop,那么需要在windows的hosts的文件中建立映射192.168.162.103 master等,hosts的位置在c:/windows/system32/driver/etc/host  在里边修改即可

## 课程回顾

### SVN
>1.SVN url 用户 密码(公司),通过这三个东西和公司的仓库建立联系,可以从仓库下载项目,或是上传本地的项目
2.SVN checkout,是将服务端的目录下载并和本地关联,可以本地commit文件,然后本地update再将服务端的经过修改的文件下载下来
3.修改文件先上锁,防止其他人也能修改,修改完成后再解锁.(测试)
4.提交或是update失败的话,可能是本地和服务端 不一致,那么就tor即本地,clearup一下,即清除一下,再重新去服务端update一下就可以了.

### hadoop

>1.mapreduce是计算模型,即我们写的算法,yarn是框架,用来执行mapreduce算法.
2.流程HDFS-->mapreduce(hadoop重点)-->hive(将sql(hive的重点)转换成mapreduce,大数据的重点)-->hbase(数据库提供快速的读写,文件不支持并发,所以储存在数据库速度快,支持高并发,mysql的数据多的话就会很慢,hbase的话100T也会很快,属于no-sql){项目,数据采集,数据精集,图表的展示(数据展示,echarts官方文档)}--->flume(分布式,稳定性,只能采集文本文件,包括爬虫的html,不能采集视频),kafka(spark,storm流处理,redis),sqoop(数据的导入导出,从其他数据平台,如mysql等关系型数据库的数据抽取到大数据平台){数据采集},Zookeeper(高可用,掌握用途,配置,hbase的前提,节点挂掉即时检测更新)--->(scala语言)spark(速度快,简单上手,可能和mapreduce功能相似)--->cdh(hadoop的另一个版本,收费,华为的hadoop版本){spark的项目}--->面试阶段
3.elk,impala,rabbitMQ,kylin,oracle,shell语言,r语言,kettle,redis,solr,flink,druid框架
4.爬虫就是发送http请求,得到html得到数据建立索引储存.搜索引擎nutch(技术框架)
5.nutch 演化出来:hadoop  lucene全文检索 solr和elasticsearch  序列化avro(hadoop本身的框架)
![][4]
6.分布式的应用层场景:大数据常规机器满足不了,高并发 单节点单服务器满足不了
7.分布式:通过网络把多台机器联系起来,多台机器通过信息系统通信,以一个整体对外提供同意的服务,有两项技术,一个是序列化,是用来通信,以流的形式,多台机器的通信,另一个通信需要协议IPC协议包括LPC(本地过程调用)RPC(远程过程调用)
8.Mahout把数学公式封装成函数,传参数即可,用来做一些分析
9.jps 是java的指令,显示进程,在window中只会显示java的相关进程
10.大数据的计算形式,先分快,存储在不同的节点上,当需要计算的时候,在各个节点上运行mapreduce程序,数据不移动,程序移动,哪里有数据,就去哪里运行程序
11.克隆系统,首先关闭要克隆的机器,选择完整克隆,克隆结束后执行命令vi  /etc/sysconfig/network-scripts/ifcfg-eth0  需要修改HWADDR ,执行ifconfig 获取HWADDR的值,复制到上个操作中,然后重启一下系统reboot即可,一般操作是先ifconfig查
看信息,然后复制HWADDR,再执行命令vi /etc/sysconfig/network-scripts/ifcfg-eth0 ,修改HWADDR和ip 静态的ip,之后reboot重启生效,ping一下,测试一下是否连接成功.
12.在linux系统中,以.点开头的文件是隐藏文件.
13.window中用+号连接两个字段,%%代表引用变量.在linux中用:号连接两个字段,$表示引用变量.
14.env.sh是运行环境配置(hadoop-env.sh yarn-env.sh) *.xml是hadoop各组件配置文件(4个) slaves(子节点的配置)
15.core-site.xml  fs是filesystem文件系统,apch是9000端口,最后两个是执行权限
16.hdfs.xml  zookeeper的主节点,namenode的地址和datanode的地址和副本数量,默认是3,一般小于子节点的数量,启动web服务即通过浏览器看 hdfs的状态,权限控制false不控制,可以通过windoe访问hadoop在linux中,下边也是web
17.mapred-site.xml  运行在yarn中,启动端口来记录处理过程以后有问题可以方便查看,rpc的服务端口,各个节点的互相通讯
18.HDFS 的常用命令
hadoop fs -ls /  查看HDFS根目录
hadoop fs -mkdir /test 在根目录创建一个目录test
hadoop fs -mkdir /test1 在根目录创建一个目录test1
hadoop fs -put ./test.txt /test 或者 hadoop fs -copyFromLocal ./test.txt /test
hadoop fs -get /test/test.txt 或者 hadoop fs -getToLocal /test/test.txt
hadoop fs -cp /test/test.txt /test1
hadoop fs -rm /test1/test.txt
hadoop fs -mv /test/test.txt /test1
hadoop fs -rm -r /test1 递归删除
19.格式化hadoop,前提先删除dfs中的name和data文件夹,以免格式化不成功.

## hadoop特点

>Hadoop是一个能够让用户轻松架构和使用的分布式计算平台。用户可以轻松地在Hadoop上开发和运行处理海量数据的应用程序。它主要有以下几个优点：
高可靠性。Hadoop按位存储和处理数据的能力值得人们信赖。
高扩展性。Hadoop是在可用的计算机集簇间分配数据并完成计算任务的，这些集簇可以方便地扩展到数以千计的节点中。
高效性。Hadoop能够在节点之间动态地移动数据，并保证各个节点的动态平衡，因此处理速度非常快。
高容错性。Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分配。
低成本。与一体机、商用数据仓库以及QlikView、Yonghong Z-Suite等数据集市相比，hadoop是开源的，项目的软件成本因此会大大降低。

## hadoop的核心进程

>1.NameNode:它是Hadoop 中的主服务器，管理文件系统名称空间和对集群中存储的文件的访问.元数据节点，是系统唯一的管理者。负责元数据的管理;与client交互进行提供元数据查询;分配数据存储节点等。
>2.Client:客户端，系统使用者，调用HDFS API操作文件;与NN交互获取文件元数据;与DN交互进行数据读写。通过端口和hadoop连接,通过流的形式进行读写操作
>3.DataNode:它负责管理连接到节点的存储（一个集群中可以有多个节点）。每个存储数据的节点运行一个 datanode 守护进程。数据存储节点，负责数据块的存储与冗余备份;执行数据块的读写操作等。
>4.JobTracker:JobTracker负责调度 DataNode上的工作。每个 DataNode有一个TaskTracker，它们执行实际工作。
JobTracker和 TaskTracker采用主-从形式，JobTracker跨DataNode分发工作，而 TaskTracker执行任务。
JobTracker还检查请求的工作，如果一个 DataNode由于某种原因失败，JobTracker会重新调度以前的任务。
>5.TsakTracker:TaskTracker是在网络环境中开始和跟踪任务的核心位置。与Jobtracker连接请求执行任务而后报告任务状态。
>6.ResourceManager :
ResourceManager包含两个主要的组件：定时调用器(Scheduler)以及应用管理器(ApplicationManager)。
定时调用器(Scheduler)：
定时调度器负责向应用程序分配置资源，它不做监控以及应用程序的状态跟踪，并且它不保证会重启由于应用程序本身或硬件出错而执行失败 的应用程序。
应用管理器(ApplicationManager)：
应用程序管理器负责接收新任务，协调并提供在ApplicationMaster容 器失败时的重启功能。
>7.NodeManager:
节点管理器（NodeManager）：
NodeManager是ResourceManager在每台机器的上代理，负责容器的管理，并监控他们的资源使用情况（cpu，内存，磁盘及网络等），以及向 ResourceManager/Scheduler提供这些资源使用报告。
应用总管（ApplicationMaster）：
每个应用程序的ApplicationMaster负责从Scheduler申请资源，以及跟踪这些资源的使用情况以及任务进度的监控。


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507642029248.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507643995907.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507645850788.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507649053793.jpg