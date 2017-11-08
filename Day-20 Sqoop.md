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
>因为呢需要连接数据库,所以需要连接数据库的jar包,将jar包拷贝到sqoop的lib目录下,是第三方jar默认的存放位置

## 创建mysql的link(因为sqoop会发布到yarn上运行,且只有master有mysql,所以不能使用localhost)

![标识符][13]

![创建历程][14]
标志符是为了 表名字段名等名称不和关键字一样,避免出错,mysql的默认标识符是逗号,  但是在sql编译过程会报错,所以可以设置为空格,

## 创建hdfs的link

![流程][15]
需要指定配置文件的目录,因为依赖hadoop的配置,所以给出的是hadoop的conf

## 创建job,指定from和to

![创建job][16]

## 修改job update job -n jobName(因为不认识数据库hive,所以不适用schme,update下)
**在创建link的时候已经指定hive数据库**

![修改job][17]

## 启动job(首先开启hadoop的history)
>因为sqoop的任务是发布到yarn上去执行的,所以任务的进度需要查看yarn的8088端口,并且希望看到运行的日志,所以需要开启history,进入到hadoop的sbin目录下mr-jobhistory-daemon.sh start historyserver 启动history服务
>一般查看最后一个日志syslog(系统记录)stderr(标准错误,信息不详细)

![查看日志][18]

![详细日志][19]

- start job -n flocalmysqltobd14hdfs      开启job任务

![提交到yarn][20]

- status job  -n flocalmysqltobd14hdfs  查看job的状态

![查看状态][21]

## 因为To Hdfs,所以去查看master:50070

![查看结果][22]

# mysql连接不上问题,远程登录权限问题
## 错误查找方向
- 一运行就报错(面板报错),那么就在sqoop的@LOGDIR日志中查看原因

![connector失败][23]

- 成功提交到yarn上,但是task失败,查看yarn日志(不认识hive数据库,通过删除schema解决)

## 远程登录权限
- 查看user表,把其他的记录删除,只留下localhost和%
- flush 一下,生效

![解决connector][24]

报错信息:

![sqoop日志][25]

![yarn日志][26]

![正确信息][27]

# 关于sqoop的@BASEDIR和@LOGSDIR
- 首先这两个文件,会生成在启动sqoop服务的目录下
- @BASEDIR中存放元数据
- @LOGSDIR sqoop的操作日志
- 若启动服务的目录不一致,那么会导致元数据丢失,之前创建的link,job就不存在 了

# 把hdfs中的数据导入到mysql(支持CSV)
- CSV格式是逗号隔开的文本格式(1,账单,10)
- 将数据导入到mysql中,只支持这种格式的数据

# 把hdfs上的文件导入到hive的mysql中

![][28]

## 打开mysql,创建表

![创建表][29]

# 常用命令和注意事项
- show link;列出当前所有link
- show job ; 列出当前所有job
- show link -n linkName;列出此名称的link
- update link -n Jobname ;更新link,需要重新输入密码
- 删除link,需要先删除job,删除job,需要先diaable,然后再删除,然后diaable link -n name disable 一下link,然后才能删除

![删除link][30]

- sqoop2-server start  开启服务
- sqoop2-server stop  关闭服务
- sqoop2-shell 连接服务
- ctrl+c 断开连接

# javaAPI实现从hdfs上读文件,写到本地的mysql中

``` stylus
package com.zhiyou.sqoop.client;

import java.util.List;

import org.apache.sqoop.client.SqoopClient;
import org.apache.sqoop.model.MConfig;
import org.apache.sqoop.model.MFromConfig;
import org.apache.sqoop.model.MInput;
import org.apache.sqoop.model.MJob;
import org.apache.sqoop.model.MLink;
import org.apache.sqoop.model.MLinkConfig;
import org.apache.sqoop.model.MSubmission;
import org.apache.sqoop.model.MToConfig;
import org.apache.sqoop.validation.Status;

/**  
* @ClassName: SqoopTest  
* @Description: TODO  
* @author zyz  
* @date 2017年11月8日 下午5:02:48  
*   
*/
public class SqoopTest {

	//sqoop的服务端url地址
	private final String URL = "http://master:12000/sqoop/";
	//创建客户端对象
	private SqoopClient client = new SqoopClient(URL);
	//创建一个link
	private void createLink(){
		//创建一个generic-jdbc-connector类型的link
		MLink link = client.createLink("generic-jdbc-connector");
		//link.getConnectorLinkConfig()获取的是connector的配置信息
		link.setName("to_window_mysql");
		MLinkConfig linkConfig = link.getConnectorLinkConfig();
		//根据配置项名称来获取配置项
		//取出所有的配置项
		List<MConfig> list = linkConfig.getConfigs();
		/*for (MConfig mConfig : list) {
			List<MInput<?>> inputs = mConfig.getInputs();
			for (MInput<?> mInput : inputs) {
				System.out.println(mInput);
			}
		}*/
		//MLinkconfig的相关配置项并设置相应的值
		linkConfig.getStringInput("linkConfig.jdbcDriver").setValue("com.mysql.jdbc.Driver");
		linkConfig.getStringInput("linkConfig.connectionString").setValue("jdbc:mysql://192.168.6.186:3306/test");
		linkConfig.getStringInput("linkConfig.username").setValue("root");
		linkConfig.getStringInput("linkConfig.password").setValue("root");
		linkConfig.getStringInput("dialect.identifierEnclose").setValue(" ");
		//保存到sqoop服务器
		Status status = client.saveLink(link);
		if(status.canProceed()){
			System.out.println("创建link"+link.getName()+"成功");
		}else{
			System.out.println("创建link失败");
		}
	}
	
	public void createJob(){
		MJob job = client.createJob("bd14hdfs", "to_window_mysql");
		job.setName("towindowmysql");
		//job.setCreationUser("root");
		//配置fromJob
		MFromConfig fromJobConfig = job.getFromJobConfig();
		List<MConfig> configs = fromJobConfig.getConfigs();
		/*for (MConfig mConfig : configs) {
			List<MInput<?>> inputs = mConfig.getInputs();
			for (MInput<?> mInput : inputs) {
				System.out.println(mInput);
			}
		}*/
		fromJobConfig.getStringInput("fromJobConfig.inputDirectory").setValue("/bd14/exptomysql");
	  
		//配置toJob
	   MToConfig toJobConfig = job.getToJobConfig();
	   List<MConfig> configs2 = toJobConfig.getConfigs();
	   /*for (MConfig mConfig : configs2) {
		List<MInput<?>> inputs = mConfig.getInputs();
		for (MInput<?> mInput : inputs) {
			System.out.println(mInput);
		}
	}*/
	toJobConfig.getStringInput("toJobConfig.schemaName").setValue("from_java_sqoop");
	toJobConfig.getStringInput("toJobConfig.tableName").setValue("j_user");
	Status status = client.saveJob(job);
	if(status.canProceed()){
		System.out.println("成功创建"+job.getName());
	}else{
		System.out.println("创建失败");
	}
	}
	public void startJob(){
		MSubmission submission = client.startJob("towindowmysql");
	}
	public static void main(String[] args) {
		SqoopTest sqoopTest = new SqoopTest();
		sqoopTest.createLink();
	    sqoopTest.createJob();
		sqoopTest.startJob();
	}

}

```


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
  [16]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510126780244.jpg
  [17]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510126885796.jpg
  [18]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127159593.jpg
  [19]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127197747.jpg
  [20]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127402593.jpg
  [21]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127471495.jpg
  [22]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127680559.jpg
  [23]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510127954249.jpg
  [24]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510128160933.jpg
  [25]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510128220534.jpg
  [26]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510128240722.jpg
  [27]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510128259364.jpg
  [28]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510148303959.jpg
  [29]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510149486187.jpg
  [30]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510148688521.jpg