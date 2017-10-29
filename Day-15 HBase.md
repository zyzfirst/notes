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

# hbase的数据存储结构

![][1]

>kv的格式

![][2]

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



  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509272728879.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509272746527.jpg