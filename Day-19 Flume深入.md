---
title: Day-19 Flume深入 
tags: selector,intercepter,java
grammar_cjkRuby: true
---


# 电商平台的数据采集
分为两种:一是实时的进行数据采集,二是批量采集(通过读取日志文件spooling)
可以通过在web应用前加拦截器,当请求来的时候将用户的行为以http请求的形式发送给flume
埋点是常用的前段数据采集方式

![enter description here][1]

![enter description here][2]

![enter description here][3]



# 使用当地时间存储在hdfs上,并且动态生成文件目录

``` stylus
al.sources=r1
al.channels=c1 c2
al.sinks=s1 s2

al.sources.r1.type=avro
al.sources.r1.bind=master
al.sources.r1.port=8888

al.channels.c1.type=memory
al.channels.c1.capacity=1000
al.channels.c1.transactionCapacity=100

al.channels.c2.type=memory
al.channels.c2.capacity=1000
al.channels.c2.transactionCapacity=100

al.sinks.s2.type=logger

al.sinks.s1.type=hdfs
al.sinks.s1.hdfs.path=/flume/logdata/%Y-%m-%d
al.sinks.s1.hdfs.hdfs.fileSuffix=.log
al.sinks.s1.hdfs.hdfs.filePrefix=test_log
al.sinks.s1.hdfs.fileSuffix=0
al.sinks.s1.hdfs.rollSize=0
al.sinks.s1.hdfs.rollCount=100
al.sinks.s1.hdfs.fileType=DataStream
al.sinks.s1.hdfs.writeFormat=Text
al.sinks.s1.hdfs.useLocalTimeStamp=true

al.sources.r1.channels=c1 c2
al.sinks.s1.channel=c1
al.sinks.s2.channel=c2
```
# 匹配抓取内容,根据不同内容,分配到不同的channel中(按省份统计数据)(selector)
## conf文件的配置

``` stylus
al.sources=r1
al.channels=c1 c2 c3 c4
al.sinks=s1 s2 s3 s4

al.sources.r1.type=avro
al.sources.r1.bind=master
al.sources.r1.port=8888

al.channels.c1.type=memory
al.channels.c1.capacity=1000
al.channels.c1.transactionCapacity=100

al.channels.c2.type=memory
al.channels.c2.capacity=1000
al.channels.c2.transactionCapacity=100

al.channels.c3.type=memory
al.channels.c3.capacity=1000
al.channels.c3.transactionCapacity=100

al.channels.c4.type=memory
al.channels.c4.capacity=1000
al.channels.c4.transactionCapacity=100

al.sinks.s1.type=hdfs
al.sinks.s1.hdfs.path=/flume/logdata/%Y-%m-%d/henan
al.sinks.s1.hdfs.hdfs.fileSuffix=.log
al.sinks.s1.hdfs.hdfs.filePrefix=test_log
al.sinks.s1.hdfs.fileSuffix=0
al.sinks.s1.hdfs.rollSize=0
al.sinks.s1.hdfs.rollCount=0
al.sinks.s1.hdfs.fileType=DataStream
al.sinks.s1.hdfs.writeFormat=Text
al.sinks.s1.hdfs.useLocalTimeStamp=true

al.sinks.s2.type=hdfs
al.sinks.s2.hdfs.path=/flume/logdata/%Y-%m-%d/henbei
al.sinks.s2.hdfs.hdfs.fileSuffix=.log
al.sinks.s2.hdfs.hdfs.filePrefix=test_log
al.sinks.s2.hdfs.fileSuffix=0
al.sinks.s2.hdfs.rollSize=0
al.sinks.s2.hdfs.rollCount=0
al.sinks.s2.hdfs.fileType=DataStream
al.sinks.s2.hdfs.writeFormat=Text
al.sinks.s2.hdfs.useLocalTimeStamp=true

al.sinks.s3.type=hdfs
al.sinks.s3.hdfs.path=/flume/logdata/%Y-%m-%d/shandong
al.sinks.s3.hdfs.hdfs.fileSuffix=.log
al.sinks.s3.hdfs.hdfs.filePrefix=test_log
al.sinks.s3.hdfs.fileSuffix=0
al.sinks.s3.hdfs.rollSize=0
al.sinks.s3.hdfs.rollCount=0
al.sinks.s3.hdfs.fileType=DataStream
al.sinks.s3.hdfs.writeFormat=Text
al.sinks.s3.hdfs.useLocalTimeStamp=true

al.sinks.s4.type=hdfs
al.sinks.s4.hdfs.path=/flume/logdata/%Y-%m-%d/qita
al.sinks.s4.hdfs.hdfs.fileSuffix=.log
al.sinks.s4.hdfs.hdfs.filePrefix=test_log
al.sinks.s4.hdfs.fileSuffix=0
al.sinks.s4.hdfs.rollSize=0
al.sinks.s4.hdfs.rollCount=0
al.sinks.s4.hdfs.fileType=DataStream
al.sinks.s4.hdfs.writeFormat=Text
al.sinks.s4.hdfs.useLocalTimeStamp=true

al.sinks.s1.channel=c1
al.sinks.s2.channel=c2
al.sinks.s3.channel=c3
al.sinks.s4.channel=c4

al.sources.r1.selector.type=multiplexing
al.sources.r1.selector.header=province
al.sources.r1.selector.mapping.henan=c1
al.sources.r1.selector.mapping.hebei=c2
al.sources.r1.selector.mapping.shandong=c3
al.sources.r1.selector.default=c4

al.sources.r1.channels=c1 c2 c3 c4
```
## javaAPI发送数据以验证

``` stylus
package com.zhiyou.flume.client;

import java.nio.charset.Charset;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

import org.apache.flume.Event;
import org.apache.flume.EventDeliveryException;
import org.apache.flume.api.RpcClient;
import org.apache.flume.api.RpcClientFactory;
import org.apache.flume.event.EventBuilder;

/**  
* @ClassName: ForFanOutSelectorClient  
* @Description: TODO  
* @author zyz  
* @date 2017年11月7日 上午10:47:42  
*   
*/
public class ForFanOutSelectorClient {

	//agent 的selector 为multiplexing类型
	// 到event的header中匹配key为provice的value,然后发送到相应的channel
	private RpcClient client;
	private final String[] provinces={"henan","hebei","shandong","beijing"};
	private final Random random = new Random();
	public ForFanOutSelectorClient(String hostName,int port){
		this.client=RpcClientFactory.getDefaultInstance(hostName, port);
	}
	public Event getRandomEvent(String msg){
		Map<String, String> headers = new HashMap<>();
		String province = provinces[random.nextInt(4)];
		headers.put("province", province);
		Event result = EventBuilder.withBody(msg, Charset.forName("utf-8"), headers);
		return result;
	}
	public void sendEvent(Event event){
		try {
			client.append(event);
		} catch (EventDeliveryException e) {
			e.printStackTrace();
		}
	}
	public void close(){
		client.close();
	}
	public static void main(String[] args) {
		ForFanOutSelectorClient selectorClient = new ForFanOutSelectorClient("master", 8888);
	    String msg = "peopleinfo_";
	    for(int i=0;i<300;i++){
	    	selectorClient.sendEvent(selectorClient.getRandomEvent(msg+i+"_"));
	    }
		selectorClient.close();
	}

}

```

# 给抓取内容中,添加时间戳(interceptor)

``` stylus
al.sources=r1
al.sinks=s1
al.channels=c1

al.sources.r1.type=avro
al.sources.r1.bind=master
al.sources.r1.port=8888

al.channels.c1.type=memory
al.channels.c1.capacity=1000
al.channels.c1.transactionCapacity=100

al.sinks.s1.type=logger

al.sources.r1.channels=c1
al.sinks.s1.channel=c1

al.sources.r1.interceptors=i1
al.sources.r1.interceptors.i1.type=timestamp
```
# 利用interceptor(拦截器)extractor(提取器),只提取包含邮箱的信息,打印或是存储,不提取其他内容,只要邮箱


``` stylus
al.sources=r1
al.sinks=s1
al.channels=c1

al.sources.r1.type=spooldir
al.sources.r1.spoolDir=/bigdata/logdir
#al.sources.r1.fileHeader=true

al.channels.c1.type=memory
al.channels.c1.capacity=1000
al.channels.c1.transactionCapacity=100

al.sinks.s1.type=logger

al.sources.r1.channels=c1
al.sinks.s1.channel=c1

al.sources.r1.interceptors=i1
al.sources.r1.interceptors.i1.type=regex_extractor
al.sources.ri.interceptors.i1.regex=.*([\\w|\\d]+\\@[\\w|\\d|\\.]).*
al.sources.r1.interceptors.i1.serializers=e1
al.sources.ri.interceptors.i1.serializers.e1.name=one
```
# 利用interceptor(拦截器)fileter(过滤器),只打印包含匹配内容的记录

``` stylus
al.sources=r1
al.channels=c1
al.sinks=s1

al.sources.r1.type=spooldir
al.sources.r1.spoolDir=/bigdata/logdir
al.sources.r1.fileHeader=true

al.channels.c1.type=memory
al.channels.c1.capacity=1000
al.channels.c1.transactionCapacity=100

al.sinks.s1.type=logger

al.sources.r1.channels=c1
al.sinks.s1.channel=c1

al.sources.r1.interceptors=i1
al.sources.r1.interceptors.i1.type=regex_filter
al.sources.r1.interceptors.i1.regex=.*[\\w|\\d]+\\@[\\w|\\d]+.*
al.sources.r1.interceptors.excludeEvents=false
```
# 利用interceptor(拦截器)replace(替代器),进行手机号码的脱敏

``` stylus
a1.sources = r1
a1.sinks=s1
a1.channels=c1

a1.sources.r1.type = avro
a1.sources.r1.bind = master
a1.sources.r1.port = 8888

a1.channels.c1.type= memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.s1.type = logger

a1.sources.r1.channels = c1
a1.sinks.s1.channel = c1

a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = search_replace

a1.sources.r1.interceptors.i1.searchPattern = (\\d{3})\\d{4}(\\d{4})
a1.sources.r1.interceptors.i1.replaceString =$1xxxx$2
```


# 利用interceptor做一些聚合运算,例如wordcount

![enter description here][4]

步骤:
- 利用javaAPI写好排除规则和聚合规则
- 在conf中配置
- 将写好的java,打成jar包,发送到flume的lib目录下
- 测试是否完成聚合

``` stylus
package com.zhiyou.flume.interceptor;

import java.util.Arrays;
import java.util.List;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

/**  
* @ClassName: WordCountInterceptor  
* @Description: TODO  
* @author zyz  
* @date 2017年11月7日 下午4:06:01  
*   
*/
public class WordCountInterceptor implements Interceptor{

	//参数excludeWords可以填写多个单词,当填写多个单词的时候用逗号隔开
	private String excludeWord;
	private String[] excludeWordArray;
	private int eventCount;
	public WordCountInterceptor(String excludeWord){
		this.excludeWord=excludeWord;
		if(excludeWord!=null&& !excludeWord.equals("")){
			excludeWordArray=this.excludeWord.split(",");
		}
	}
	@Override
	public void initialize() {
		
		
	}

	//拦截过程数据处理逻辑
	@Override
	public Event intercept(Event event) {
		eventCount=0;
		String[] words=new String(event.getBody()).split("\\s");
		if(excludeWordArray==null || excludeWordArray.length<1){
			eventCount=words.length;
		}else{
			List<String> excludeList = Arrays.asList(excludeWordArray);
			if(!excludeList.contains(words)){
				eventCount ++;
			}
		}
		event.setBody(String.valueOf(eventCount).getBytes());
		return event;
	}

	//使用单个event拦截处理过程来实现list列表event的处理过程
	@Override
	public List<Event> intercept(List<Event> events) {
		for(Event event:events){
			intercept(event);
		}
		return events;
	}

	@Override
	public void close() {
	}
	
	public static class Builder implements Interceptor.Builder{

		private String excludeWords;
		@Override
		public void configure(Context context) {
			//excludeWords是一个变量,是自己定义的排除的单词
			excludeWords = context.getString("excludeWords");
			
		}

		@Override
		public Interceptor build() {
			return new WordCountInterceptor(excludeWords);
		}
		
	}
}

```
配置文件的配置($类名,代表内部类)

``` stylus
al.sources = r1
al.sinks = s1
al.channels = c1

al.sources.r1.type = netcat
al.sources.r1.bind = localhost
al.sources.r1.port = 44444

al.sinks.s1.type = logger

al.channels.c1.type = memory
al.channels.c1.capacity = 1000
al.channels.c1.transactionCapacity = 100

al.sources.r1.channels = c1
al.sinks.s1.channel = c1

al.sources.r1.interceptors=i1
al.sources.r1.interceptors.i1.type=com.zhiyou.flume.interceptor.WordCountInterceptor$Builder
al.sources.r1.interceptors.i1.excludeWords=abc
```

# 将telnet的内容写到hive中
存储在hive中,需要不断的append数据,因此需要hive支持事务,所以在建表的时候需要分桶,设置transaction
- 创建分桶表,支持事务
- 开放metastore的端口
- 配置文件

![创建分桶表,设置支持事务][5]

hive --service metastore  开启hive的metastore  端口是9083
netstat -alnp | grep 9083    来查看端口是否启动
hive --help可以查看帮助

``` stylus
al.sources = r1
al.sinks = s1
al.channels = c1

al.sources.r1.type = netcat
al.sources.r1.bind = localhost
al.sources.r1.port = 44444

al.sinks.s1.type=hive
al.sinks.s1.hive.metastore=thrift://master:9083
al.sinks.s1.hive.database=bd14
al.sinks.s1.hive.table=flume_user
al.sinks.s1.serializer=DELIMITED
al.sinks.s1.serializer.delimiter="\t"
al.sinks.s1.serializer.serdeSeparator='\t'
al.sinks.s1.serializer.fieldnames=user_id,user_name,age


al.channels.c1.type = memory
al.channels.c1.capacity = 1000
al.channels.c1.transactionCapacity = 100

al.sources.r1.channels = c1
al.sinks.s1.channel = c1
```
# 将telnet的数据存储到hbase上
- 进入hbase shell 中创建表指定列簇
- 配置文件

``` stylus
al.sources = r1
al.sinks = s1
al.channels = c1

al.sources.r1.type = netcat
al.sources.r1.bind = localhost
al.sources.r1.port = 44444

al.sinks.s1.type=hbase
al.sinks.s1.table=bd14:flume_user
al.sinks.s1.columnFamily=i

al.channels.c1.type = memory
al.channels.c1.capacity = 1000
al.channels.c1.transactionCapacity = 100

al.sources.r1.channels = c1
al.sinks.s1.channel = c1
```



  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510063656041.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510064001320.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510064076124.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510064162592.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510066906410.jpg