---
title: Day-09 Avro
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# Avro概述
## 作用
>1.作为大数据平台的文件格式mapreduce读avro文件和写avro文件  
   >因为如果存取text文本格式,要进行io读写,而avro本身就是二进制数据格式,不用进行io的序列化与反序列化等操作
   >2.合并小文件,作为一个文件容器容纳多个小文件,本身是hdfs上是一个大文件
   >hadoop对多个小文件的支持不太好,因为namenode存储元数据太占空间,所以不宜处理多个小文件,因此可以将多个小文件放进一个大文件中储存

## 数据处理流程

![][1]

>1. 数据采集(爬虫,数据产生方推送),
>2. 数据分析(数据源是文本的可能性比较大,分析聚合过程不适用文本文件,需要解析频繁的io,即中间步骤不用文本最后得出)在分析过程中可能由多个mapreduce,在存储的时候,一般不使用textoutputformat,因为会存成文本,下次再去读取数据的时候,还需要解析成二进制流的形式,比较耗费资源和时间,所以一般使用avro(二进制格式文件)或是sequceoutputformat,sequnceinputformat,会保留数据和数据的格式,不用进行频繁的格式解析
>3. 数据展现(rdbms),此处rdbms就是各种数据库

### sequenceFile和Avro的比较
>sequencefile是hadoop框架自带的,保存数据和保存kv的数据格式,但是只能保存kv两个字段
>avro本身是二进制文件,所以展现的时候需要反序列化解析一下,才能转换成看得懂的text文本,它可以保存多个字段的格式

## avro的特性
>arvo的特性和功能：
1：丰富的数据结构类型
2：快速可压缩的二进制数据格式
3：存储持久数据的文件容器
4：远程过程调用（RPC）
5：简单的动态语言结合功能
6 :Code generation(自动代码生成)
7 :arvo数据存储到文件中时，模式随着存储，这样保证任何程序都可以对文件进行处理。

# Avro的使用流程

## 添加依赖在pom.xml,相当于spring等框架,添加依赖即可使用

``` stylus
<dependencies>
		<dependency>
			<groupId>org.apache.avro</groupId>
			<artifactId>avro</artifactId>
			<version>1.8.2</version>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.avro</groupId>
				<artifactId>avro-maven-plugin</artifactId>
				<version>1.8.2</version>
				<executions>
					<execution>
						<phase>generate-sources</phase>
						<goals>
							<goal>schema</goal>
						</goals>
						<configuration>
							<sourceDirectory>${project.basedir}/src/main/avro/</sourceDirectory>
							<outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```
>包括依赖和插件(自动生成schema文件的插件),generate-sources是自动生成代码的命令

![][2]

## 在配置的路径下创建schema文件后缀为.avsc  例如user_action_log.avsc

``` stylus
{
"type":"record",
"name":"UserActionLog",
"namespace":"com.zhiyou.bd14.schema",
"fields":[
    {"name":"userName","type":"string"},
    {"name":"actionType","type":"string"},
    {"name":"ipAddress","type":"string"},
    {"name":"gender","type":"int"},
    {"name":"provience","type":"string"}
]
}
```
>它是json格式,key必须是string,fields是一个数组[],而数组里边是json对象{},这就是格式,type是类型,是一个个的record就是记录,在mapreduce中用avroKey或是avroValue读存,还有type是map,用avroKeyValue读存,会把map的kv分别解析成kv对传给map

## 使用命令自动生成schema的java文件是约束文件,有两种方式

### 使用window的cmd命令
>1.找到项目的src路径 ,在cmd中   cd: 复制的地址
>2.d:进入到盘符,这样就进入了src的目录下
>3.执行命令 mvn generate-sources  因为是maven项目所以执行mvn命令

![][3]

![][4]

### 使用maven的configer,配置命令,然后直接run就可以

![][5]

# 存储数据格式

![][6]

>Doug Cutting创建了Avro项目，它有数据序列化工具和RPC库组成，用于改进MapReduce中的数据交换频率、互操作性和版本控制。Avro使用压缩二进制数据格式（可以通过配置选项设置压缩，压缩可以快速提高序列化时间）。Avro可以直接在MR中该处理，可以使用通用的数据模型处理schema数据。
>它的数据分为header和打他block,header中存放魔数和scheama

## 魔数和achema
>**魔数**: 表明是那种格式是二进制16位,java的class文件的魔数是coffee,就跟txt是文本,不过这种window的识别方式比较low,,header datablock也是sequence的格式
>**schema**: 定义schema就是定义数据库中的模式(元数据,表中的字段名,类型与约束等相关信息),它使用json形式来定义schema,在java中json是一个对象,而在数据交互中json作为一种数据存储格式,保存kv对,xml(本身标签就占很大得空间,比数据本身还大),properties(配置文件格式)json(也可以作为配置文件的格式,比较轻量,没有复杂的标签)
>**修改数据结构需要在schema文件中修改,把原来的生成的文件删除,重新生成**

# 模式即schema必须定义,模式对象可以生成可以不生成
## 生成模式对象

### 序列化流程:
>1.new UserActionLog对象,这个对象由schema文件生成,也能得到schema文件,这个类用来封装数据,保证数据的格式,若没有模式对象,则用系统提供的类来封装对象
>2.DatumWriter (UserActionLog) writer = new SpecificDatumWriter<>();这个是创建模式对象,用特殊的datumwriter它是一个接口,由多个实现类
>3.DataFileWriter(UserActionLog> fileWriter = new DataFileWriter<>(writer);这个是固定的,用来进行文件的读写,第二部可以理解为用那种格式
>4.DataFileWriter对象创建输出路径,并将schema文件传给它,把这个文件写进header中
>5.DataFileWriter对象执行append操作,把UserActionLog写进去,然后flush和close

``` stylus
public class WriteAvro {
	public static void main(String[] args) throws Exception {
		UserActionLog uall = new UserActionLog();
		uall.setActionType("login");
		uall.setGender(1);
		uall.setIpAddress("192.168.6.163");
		uall.setProvience("河北");
		//没值会报null错误,所以在配置文件中可以设置defalutvalue,这样就会有默认值,即使某个字段不赋值,也会有默认值
		uall.setUserName("小风");
		UserActionLog uall2 = UserActionLog.newBuilder().setActionType("longout")
		.setGender(0).setIpAddress("192.168.6.254").setProvience("henan").setUserName("mingming").build();
	    //把两天记录写入文件中(序列化),生成avro模式对象之后用specificdatumwriter
		//不生成avro模式对象的情况下,用genericdatumwriter
		DatumWriter<UserActionLog> writer = new SpecificDatumWriter<>();
		DataFileWriter<UserActionLog> fileWriter = new DataFileWriter<>(writer);
		//创建序列化文件,相对路径,会建在项目根目录下
		fileWriter.create(UserActionLog.getClassSchema(), new File("userlogaction.avro"));
	   //写入内容
		fileWriter.append(uall);
		fileWriter.append(uall2);
		fileWriter.flush();
		fileWriter.close();
	
	}

}
```
### 反序列化流程
>1.new file指定要读取的文件路径
>2.DatumReader(UserActionLog) reader = new SpecificDatumReader<>();用创建模式对象的方式读取数据
>3.DataFileReader(UserActionLog) filereader = new DataFileReader<>(file, reader),把文件和读取的格式要求传给她
>4.while(hasnext())和next()方法读取,运用迭代器

``` stylus
public class ReadFromAvro {
	public static void main(String[] args) throws Exception {
		//因为模式就在文件中,所以解析文件就能知道schema,不需要指定
		File file = new File("userlogaction.avro");
		//SpecificDatumReader已经生成了模式对象用这个类,否则用genericdatum
		DatumReader<UserActionLog> reader = new SpecificDatumReader<>();
		DataFileReader<UserActionLog> filereader = new DataFileReader<UserActionLog>(file, reader);
	    UserActionLog readlog = null;
	    while(filereader.hasNext()){
	    	readlog = filereader.next();
	    	System.out.println(readlog);
	    }
	   filereader.close();
	
	}
}
```
## 不生成模式对象

### 序列化过程:schema对象需要写入header,所以用到parser把avro文件解析成schema对象
>1.和生成模式对象差不多,原理一样,其实本质还是生成了模式对象,不过这个类是=系统的,我们给她传相应的东西就自动生成了,然后把数据封装进去
>2.首先new 自己的类AvroWriter,参数是schema的路径,这个类中由schema的属性,通过schema属性生成GenericRecord这个类的对象,它相当于模式对象用来封装数据
>3.创建datumwriter,把数据放到GenericRecord,用GenericDatumWriter实现类,然后datafilewriter把数据放到GenericRecord中

``` stylus
public class AvroWriter {
	private Schema schema;
	// parser是专门用来把字符串或者avro文件转换成schema对象的工具类
	private Schema.Parser parser = new Schema.Parser();
    //初始化schema,根据要序列化的数据而定
	public AvroWriter(String schemafile) throws Exception {
		this.schema = parser.parse(new File(schemafile));
	}
 public void writerData(GenericRecord record) throws Exception{
	//GenericRecord 是schema文件中的record,没有模式对象不用声称,所以用GenericDatumWriter
     DatumWriter<GenericRecord> writer = new GenericDatumWriter<>();
     DataFileWriter<GenericRecord> fileWriter = new DataFileWriter<>(writer);
     fileWriter.create(schema, new File("noobjuserlogaction.avro"));
     fileWriter.append(record);
     fileWriter.flush();
     fileWriter.close();
 }
	public static void main(String[] args) throws Exception {
		//用模式文件位置作为参数初始化writer序列化类
		AvroWriter avroWriter = new AvroWriter("src/main/avro/user_action_log.avsc");
	    //创建GenericRecord对象
		GenericRecord record = new GenericData.Record(avroWriter.schema);
		record.put("userName", "Jim");
		record.put("actionType", "Jim");
		record.put("ipAddress", "Jim");
		record.put("gender", 2);
		record.put("provience", "hebei");
		avroWriter.writerData(record);
	}

}
```
### 反序列化
>接收的类不一样

``` stylus
public static void main(String[] args) throws Exception {
       DatumReader<GenericRecord> reader = new GenericDatumReader<>();
       DataFileReader<GenericRecord> filereader = new DataFileReader<>(new File("noobjuserlogaction.avro"), reader);
       GenericRecord record = null;
       while(filereader.hasNext()){
    	   record = filereader.next();
    	   System.out.println(record);
       }
	filereader.close();
	}

}
```
## mapreduce读取和存储avro文件

### 序列化
>1.AvroKey(UserActionLog)或者avrovalue(UserActionLog)格式,是把数据作为key或者value存储起来,看需求这个对象由datum()方法,有模式对象会存储,没有的话就会得到模式对象,模式对象可以得到自己的属性
>2.在job上设置 outputformat/    job.setOutputFormatClass(AvroKeyOutputFormat.class);
 >3  设置输出key的schema AvroJob.setOutputKeySchema(job, UserActionLog.SCHEMA$);

### 反序列化
>设置接收kv的格式AvroKey(UserActionLog)或者avrovalue(UserActionLog) ,key.datum可以得到模式对象,然后得到数据,进行操作,往reduce端传送的数据跟之前一样
>设置inputformat   
   job.setInputFormatClass(AvroKeyInputFormat.class);
AvroJob.setInputKeySchema(job, UserActionLog.SCHEMA$);






  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508331555482.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333095514.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333817781.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333952211.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508334218105.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508334387163.jpg