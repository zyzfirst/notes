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
## avro可以作为容器,存储小文件,把小文件合并成一个大文件
>相当于write的过程,只不过是write过程的模式对象中的数据是我们set或者设置的,而在合并文件的过程中,模式对象的值是先去读文件,读出来的,然后把模式对象append一下.

### 合并文件的原理或者是流程
>1.用fileUtils把文件夹下的所有文件读取出来,获取文件的路径,把所有文件的路径放到一个list集合中
>2.创建writer和filewriter,然后创建输出目录传递参数schema(要声成大文件的schema)
>3.遍历list集合,new file(path) 一个path就是一个文件,用工具读取文件的内容,fileUtils.readFileToArray把文件读成字节数组,再用bytebuffer.warp把字节数组转换成bytebuffer,把文件的内容和和get的abslutpath作为文件名放到模式对象中
>filewriter把模式对象append和flush一下,就写入到一个大文件中了

### 代码实现,小文件的合并和大文件的读取

``` stylus
//把一个文件夹下的小文件合并成大文件
public class AvroMergeSmallFile {
   private Schema.Parser parser = new Schema.Parser();
   private Schema schema;
   //用来设置添加要合并的文件的名称
   private List<String> inputFilePaths = new ArrayList<>();
   //在构造方法中初始化schema
   public AvroMergeSmallFile(){
	   schema = SmallFile.getClassSchema();
   }
   //添加要合并的文件夹
   public void addInputFileDir(String inputDir) throws Exception{
	   //获取文件夹下的所有的问价
	   File[] files = FileUtil.listFiles(new File(inputDir));
	   //把文件的路径添加到inputFilePaths中
	   for (File file : files) {
		   inputFilePaths.add(file.getPath());
	}
   }
   //把inputFilePaths中的所有文件合并到一个avro文件中
   public void mergeFile(String outputPath) throws Exception{
	   DatumWriter<SmallFile> writer = new SpecificDatumWriter<>();
	   DataFileWriter<SmallFile> filewriter = new DataFileWriter<>(writer);
	   //创建avro文件
	   filewriter.create(SmallFile.getClassSchema(), new File(outputPath));
	   //把inputFilePaths的文件一个个读取出来根据模式放入到avro文件中
	   for (String filepath : inputFilePaths) {
		File inputFile = new File(filepath);
		//把文件读成字节数组,放入content
		//ByteBuffer调用wrap方法把字节数组封装成一个对象作为参数设置到content属性中去
		byte[] content = FileUtils.readFileToByteArray(inputFile);
		SmallFile oneSmallFile = SmallFile.newBuilder().setFileName(inputFile.getAbsolutePath())
		.setContent(ByteBuffer.wrap(content)).build();
		filewriter.append(oneSmallFile);
		//DigestUtils.md5Hex(content)得到md5加密,若内容相同则md5一样
		System.out.println("写入"+inputFile.getAbsolutePath()+"成功"+DigestUtils.md5Hex(content));
	}
	   filewriter.flush();
	   filewriter.close();
	   
   }
   //读取大avro的文件内容
   public void readMergeFile(String avroFile) throws Exception{
	   DatumReader<SmallFile> reader = new SpecificDatumReader<>();
	   DataFileReader<SmallFile> filereader = new DataFileReader<>(new File(avroFile), reader);
	   SmallFile smallFile = null;
	   while(filereader.hasNext()){
    	   smallFile = filereader.next();
    	   System.out.println("文件名"+smallFile.getFileName());
    	   //new String(bytes, charset)可以指定编码,因为存储的是字节数组,所以取出来也是,为了显示结果转换成string
    	   System.out.println("md5:"+DigestUtils.md5Hex(smallFile.getContent().array()));
    	  // System.out.println("文件内容"+new String(smallFile.getContent().array(),"utf-8"));
    	   
       }
      filereader.close();
   }
   public static void main(String[] args) throws Exception {
	AvroMergeSmallFile avroMergeSmallFile = new AvroMergeSmallFile();
	avroMergeSmallFile.addInputFileDir("C:\\Users\\Administrator\\Desktop\\input");
	avroMergeSmallFile.mergeFile("C:\\Users\\Administrator\\Desktop\\outputAvro.avro");
	avroMergeSmallFile.readMergeFile("C:\\Users\\Administrator\\Desktop\\outputAvro.avro");
}
}
```


## mapreduce读取和存储avro文件
### 首先pom添加依赖

``` stylus
<dependencies>
  	<dependency>
	    <groupId>org.apache.hadoop</groupId>
	    <artifactId>hadoop-client</artifactId>
	    <version>2.7.4</version>
	</dependency>
	<dependency>
  		<groupId>org.apache.avro</groupId>
  		<artifactId>avro</artifactId>
  		<version>1.8.2</version>
	</dependency>
	<dependency>
  		<groupId>org.apache.avro</groupId>
  		<artifactId>avro-mapred</artifactId>
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
	<plugin>
  		<groupId>org.apache.maven.plugins</groupId>
  		<artifactId>maven-compiler-plugin</artifactId>
  		<configuration>
   	 	<source>1.8</source>
    	<target>1.8</target>
  	</configuration>
	</plugin>
  	</plugins>
  
  </build>
```
### 创建schema文件,生成模式对象,后缀为.avsc

![][7]


### 序列化
>1.AvroKey(UserActionLog)或者avrovalue(UserActionLog)格式,是把数据作为key或者value存储起来,看需求这个对象由datum()方法,有模式对象会存储,没有的话就会得到模式对象,模式对象可以得到自己的属性
>2.在job上设置 outputformat/    job.setOutputFormatClass(AvroKeyOutputFormat.class);
 >3  设置输出key的schema AvroJob.setOutputKeySchema(job, UserActionLog.SCHEMA$);

### 反序列化
>设置接收kv的格式AvroKey(UserActionLog)或者avrovalue(UserActionLog) ,key.datum可以得到模式对象,然后得到数据,进行操作,往reduce端传送的数据跟之前一样
>设置inputformat   
   job.setInputFormatClass(AvroKeyInputFormat.class);
AvroJob.setInputKeySchema(job, UserActionLog.SCHEMA$);

###  用mapreduce读取大文件,即合并之后的文件

#### 1.设置avrokey<模式对象>
>这样读取的,一次读取一个文件,放在key里,格式是二进制文件,需要key.datum().getcontent().array变成字节数组,然后调用new string(字节数组).split("\\s")就得到我们熟悉的内容,做wordcount的操作

#### 2.设置输出的avrokey<模式对象>,或者不用模式对象
>1. 生成了模式对象:创建avrokey<>作为outKey,然后创建模式对象,把数据set进模式对象,把对象set进avrokey中,在context写一下
>2. 不生成模式对象:需要得到原声的封装模式的对象,new parser().paerser(new file(schema路径)),这个方法写在setup方法中,只需要初始化一次即可,这样得到schema对象,得到封装数据的类GenericRecord的对象,需要new GenericData.Record(schema对象),然后把数据put进GenericRecord对象,做之前的重复操作
>3. 设置input/outputformat为相应的类,然后avroJob.setinput/output schema(job,schema对象)

``` stylus
//计算avro文件的词频
public class AvroFileMr {
//map读取avro文件,没读取一条记录,其实是一个小文件,对其进行wordcount解析,并以word,count形式发送给reduce
	public static class AvroFileMrMap extends Mapper<AvroKey<SmallFile>, NullWritable, Text, IntWritable>{
		private Text outKey = new Text();
		private final IntWritable ONE = new IntWritable(1);
		//接收原声的字节数组
		private ByteBuffer content;
		//接收文件分割之后的string内容
		private String[] infos;
		@Override
		protected void map(AvroKey<SmallFile> key, NullWritable value,
				Mapper<AvroKey<SmallFile>, NullWritable, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			//因为是avro的大文件,所以得到的ByteBuffer,它会由很多的record,每次传过来是一个record,也就是一个文件
			//key.datum()得到模式对象,由模式对象get到内容
			content = key.datum().getContent();
			//把字节文件转换成字符文件,也就是string类型,然后再用原来的方法,对文件进行分割变成一个个的单词
			//ByteBuffer.array()变成字节数组,new string把自己数组变成字符串然后分割
			infos = new String(content.array()).split("\\s");
			for (String word : infos) {
				outKey.set(word);
				context.write(outKey, ONE);
			}
		}
		
	}
	public static class AvroFileMrReduceTwo extends Reducer<Text, IntWritable, AvroKey<WordCount>, NullWritable>{
		private AvroKey<WordCount> outKey = new AvroKey<WordCount>();
		private NullWritable none = NullWritable.get();
		private int sum;
		private WordCount wordcount = new WordCount();
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, AvroKey<WordCount>, NullWritable>.Context context)
				throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += 0;
			}
			wordcount.setCount(sum);
			wordcount.setWord(key.toString());
			outKey.datum(wordcount);
			context.write(outKey, none);
		}
		
	}
	public static class AvroFileMrReduce extends Reducer<Text, IntWritable, AvroKey<GenericRecord>, NullWritable>{

		private int sum;
		private NullWritable none = NullWritable.get();
		private GenericRecord record ;
		private Schema writerSchema;
		private AvroKey<GenericRecord> outKey = new AvroKey<GenericRecord>();
		
		@Override
		protected void setup(Reducer<Text, IntWritable, AvroKey<GenericRecord>, NullWritable>.Context context)
				throws IOException, InterruptedException {
			//只需要初始化一次
			Parser parser = new Parser();
			//把文件解析成schema对象,参数是file是一个文件
			writerSchema = parser.parse(new File("src/main/avro/wordcount.avsc"));
			//原生GenericRecord对象的获取方式是由GenericData调Record,传参schema对象得到
			record = new GenericData.Record(writerSchema);
		}

		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, AvroKey<GenericRecord>, NullWritable>.Context context)
				throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			//模式对象是set方法赋值,而原声对象需要put,并且自己写key的值
			record.put("word", key);
			record.put("count", sum);
			outKey.datum(record);
			context.write(outKey, none);
		}
      
	}
	public static void main(String[] args) throws Exception {
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		job.setJobName("avroAndMR");
		job.setJarByClass(AvroFileMr.class);
		job.setMapperClass(AvroFileMrMap.class);
		job.setReducerClass(AvroFileMrReduce.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(AvroKey.class);
		job.setOutputValueClass(NullWritable.class);
		
		job.setInputFormatClass(AvroKeyInputFormat.class);
		AvroJob.setInputKeySchema(job, SmallFile.getClassSchema());
		job.setOutputFormatClass(AvroKeyOutputFormat.class);
		//schema的获取方式有两种:一种是由模式对象SmallFile.SCHEMA$或者SmallFile.getClassSchema()
		//第二种方式是:由解析器parser调用parse方法,传schema文件的路径,解析得到schema对象
		AvroJob.setOutputKeySchema(job, new Parser().parse(new File("src/main/avro/wordcount.avsc")));
		FileInputFormat.addInputPath(job, new Path("/outputAvro.avro"));
		Path path = new Path("/bd80/avroAndMr");
		path.getFileSystem(configuration).delete(path, true);
		FileOutputFormat.setOutputPath(job, path);
		System.exit(job.waitForCompletion(true)?0:1);
	}
	
}
```


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508331555482.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333095514.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333817781.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508333952211.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508334218105.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508334387163.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508414067163.jpg