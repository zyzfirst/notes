---
title: Day-20 Sqoop
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 安装与配置
## 解压并配置环境变量
- tar -xvf 名字
- 配置/etc/profile,若logs和base配置文件没有生效,那么会生成历时文件,你在哪个目录下启动sqoop就会在哪里生成,为了防止数据丢失,应当每次都在相同的目录下启动服务
- 配置hadoop信息,因为sqoop是依赖hadoop,因为配置了hadoop_home,所以不用再配置了

``` stylus
＃导出HADOOP_HOME变量
export HADOOP_HOME = / ...

＃或者HADOOP _ * _ HOME变量
export HADOOP_COMMON_HOME = / ...
 export HADOOP_HDFS_HOME = / ...
 export HADOOP_MAPRED_HOME = / ...
 export HADOOP_YARN_HOME = / ...
```


``` stylus
export SQOOP_HOME=/opt/Software/Sqoop/sqoop-1.99.7-bin-hadoop200
export PATH=$PATH:$SQOOP_HOME/bin
export LOGDIR=$SQOOP_HOME/logs
export BASEDIR=$SQOOP_HOME/base
```
![启动服务的目录hadoop下][1]

## 配置hadoop的core-site,并把它拷贝到其他节点
>需要代理hadoop来访问资源,配置hadoo允许此模拟

``` stylus
<property>
  <name>hadoop.proxyuser.sqoop2.hosts</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.sqoop2.groups</name>
  <value>*</value>
</property>
```

## 配置sqoop.properties文件
- org.apache.sqoop.submission.engine.mapreduce.configuration.directory=/opt/Software/Hadoop/hadoop-2.7.4/etc/hadoop(指定hadoop的配置文件目录)
- 可以配置额外(extra)jar的存储位置,不设置默认在lib下

![配置第三方jar的存放目录][2]

## 先初始化tool
- sqoop2-tool upgrade

![初始化tool][3]

## 启动sqoop服务,并连接服务,查看相应进程
- sqoop2-server start
- sqoop2-shell
- netstat -alnp | grep 1200

![到相应目录启动][4]

![退出命令和端口][5]

# sqoop概述
sqoop是Apache顶级项目，主要用来在Hadoop和关系数据库中传递数据。通过sqoop，我们可以方便的将数据从关系数据库导入到HDFS，或者将数据从HDFS导出到关系数据库。
**本质是将commond命令转换成mapreduce来进行处理**

![流程图][6]

# sqoop架构和两个版本的比较
![sqoop1][7]

![sqoop2][8]

## sqoop1和sqoop2区别
    这两个版本是完全不兼容的，其具体的版本号区别为1.4.x为sqoop1，1.99x为sqoop2。sqoop1和sqoop2在架构和用法上已经完全不同。
在架构上，sqoop2引入了sqoop server（具体服务器为tomcat），对connector实现了集中的管理。其访问方式也变得多样化了，其可以通过REST API、JAVA API、WEB UI以及CLI控制台方式进行访问。另外，其在安全性能方面也有一定的改善，在sqoop1中我们经常用脚本的方式将HDFS中的数据导入到mysql中，或者反过来将mysql数据导入到HDFS中，其中在脚本里边都要显示指定mysql数据库的用户名和密码的，安全性做的不是太完善。在sqoop2中，如果是通过CLI方式访问的话，会有一个交互过程界面，你输入的密码信息不被看到。

# sqoop操作的重要对象
![重要对象][9]

![基本信息][10]

## 核心对象
connector指当前系统sqoop支持的连接器和连接类型,可以自定义connector

![核心对象connector][11]
kite-connector  是ETL工具,也是处理关系型数据库的数据转换
ftp或者sftp不是存储,是发送给其他人

![job和link][12]
job和link   是数据导入导出配置对象,过程设置在这两个对象中    并且link是可以重复使用,存储着连接信息
submission查看当前已提交的sqoop的导入导出任务

# 创建一个job,from mysql To hdfs
## 创建mysql的link

![标识符][13]

![创建历程][14]
标志符是为了 表名字段名等名称不和关键字一样,避免出错,mysql的默认标识符是逗号,  但是在sql编译过程会报错,所以可以设置为空格,

## 创建hdfs的link

![流程][15]
需要指定配置文件的目录,因为依赖hadoop的配置,所以给出的是hadoop的conf


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510115169659.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510115748584.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510115842008.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116042132.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116074725.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116399301.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116431529.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116451050.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116657425.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116711744.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116756408.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510116960112.jpg
  [13]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510117101141.jpg
  [14]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510117118041.jpg
  [15]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510117280762.jpg