---
title: Spark 的发行和java开发 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 总共5种发行模式

## 流程:
- 需要将程序打包成jar,发布到linux响应的目录下,执行命令spark-submit 命令
- 主要命令 :  spark-submit  --master(local[2])(spark://master:7077)(yarn)  --class 类的全包名  jar的名字  --deploy-mode client或者cluster两种模式
- local模式只有一种, standalone和yarn模式都有两种模式(client和cluster)
- 如果有参数可以加上参数
- 还有--jar  用来指定应用程序运行所依赖的jar的位置  **可以使用sc.addJar来指定hdfs上依赖jar包的位置**  --master用来指定发行的模式

## local模式

>命令:   spark-submit --class com.zhiyou.bd17.WordCount --master local sparkbd17.jar

**local模式是在本节点上运行,就是jar包发布到哪个节点就在哪个节点上运行,是单机模式,并没有使用到集群,因此driver的输出和excutor的输出都是在本节点上**

## standalone模式,分为client和cluster模式
- spark-submit --master spark://centos1:7077 --deploy-mode client --class com.zhiyou.bd17.WordCount sparkbd17.jar
- spark-submit --master spark://centos1:7077 --deploy-mode cluster --class com.zhiyou.bd17.WordCount sparkbd17.jar

![standalone模式][1]

### client模式

**client模式,是在spark集群上运行的,driver在提交任务的节点上,而excutor的输出在excutor上,即在其他的物理节点上.在哪里submit的spark,哪里就是client,并且需要等到所有的计算都完成,spark-submit任务才会结束**

### cluster模式

**cluster模式,也是在spark集群上运行,driver程序运行在某个节点上,所以driver的输出,不一定在本节点看到,并且只要提交任务成功,在master:8080那里可以看到任务了,spark-submit任务就结束了**

## yarn模式
- spark-submit --master yarn --deploy-mode client --class com.zhiyou.bd17.WordCount sparkbd17.jar
- spark-submit --master yarn --deploy-mode cluster --class com.zhiyou.bd17.WordCount sparkbd17.jar

![yarn模式][2]

### client模式

![client][3]

**Client和Driver运行在一起，AM只用来获取资源,驱动程序driver运行在client中,即submit的节点的上,AM只是来请求获取yarn的资源**

### cluster模式

![cluster][4]

**Driver和AM运行在一起,可以说是driver嵌套在AM中,相当于驱动程序跑在AM的一个县城中,在初始化完成后client会消失,即client的请求被响应,创建AM并启动driver,driver并不在submit的节点上,去master:8088  yarn的application的运行轻卡un个上查看任务进度**

## 需要打开yarn的history服务,查看历史

 ./mr-jobhistory-daemon.sh start historyserver
 
 ## 报错
 
 ![yarn模式client][5]
 
 是虚拟内存不足引起的
 
 在yarn-site.xml中增加配置,拷贝到各个节点就可以解决了
 

``` stylus
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false<alue>
    </description>
</property>

```



 
 
 # mapreduce任务的发行
 
 流程 :
 
 - 将程序打成jar,发布到linux上
 - 执行命令 hadoop jar rdbmsMR5.jar com.zhiyou.rdbms.mysql.ReadFromDB  需要指定全包名,同样可以在后边根上参数

**如果忘记,可以hadoop --help 查看命令,或是百度hadoop jar**

# hive的脚本的发布

**将写好的sql脚本编辑成后缀为.hql的文件,发布到linux上,然后执行命令 hive -f file.hql -hivevar 参数名字=参数值**



# 使用java开发spark
## 实现功能wordcount

>步骤
  1.构建maven项目
   2.添加spark依赖(spark-core)
   3.构建JavaSparkContext(scala语言构建的是SparkContext,java使用自己的环境)
   4.使用JavaSparkContext构建JavaRDD(RDD也是使用java的)
   5.对JavaRDD调用transformation和action 操作数据（大量的内部类写法实例化算子）
   6.计算结束，停用sparkcontext：sc.stop()

>**java开发需要使用大量的内部类,因为参数只能是对象,不能是函数,所以每次需要调用方法的时候 ,都需要定义一个内部类,重写类中的方法才能用,比较麻烦**

``` stylus
package com.zhiyou.sparktest;

import java.util.Arrays;
import java.util.Iterator;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;

import scala.Tuple2;

/**  
* @ClassName: SparkTest  
* @Description: TODO  
* @author zyz  
* @date 2017年11月23日 下午5:29:14  
*   
*/
public class SparkTest {

	public static void main(String[] args) {
		SparkConf conf = new SparkConf().setAppName("java-spark").setMaster("local[*]");
		JavaSparkContext sc = new JavaSparkContext(conf);
		//构建rdd
		JavaRDD<String> rdd = sc.textFile("/user-logs-large.txt", 2);
		//把一行转换成单词
		JavaRDD<String> wordRdd = rdd.flatMap(new FlatMapFunction<String, String>() {

			@Override
			public Iterator<String> call(String t) throws Exception {
				
				return Arrays.asList(t.split("\\s")).iterator();
			}
		});
		//把单词rdd转换成kv的rdd
		JavaPairRDD<String, Integer> javaPairRDD = wordRdd.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String t) throws Exception {
				return new Tuple2<String, Integer>(t, 1);
			}
		});
		//进行聚合
		JavaPairRDD<String, Integer> result = javaPairRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
			public Integer call(Integer v1, Integer v2) throws Exception {
				return v1+v2;
			}
		});
		result.saveAsTextFile("/java-spark-output");
	}

}

```


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1511446889721.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1511447322167.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1511447510041.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1511447658386.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1511449260972.jpg