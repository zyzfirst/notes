---
title: mapreduce 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# MapReduce

## mapreduce 的由来

>MapReduce最早是由Google公司研究提出的一种面向大规模数据处理的并行计算模型和方法。Google公司设计MapReduce的初衷，主要是为了解决其搜索引擎中大规模网页数据的并行化处理。

>Google公司发明了MapReduce之后，首先用其重新改写了其搜索引擎中的Web文档索引处理系统。但由于MapReduce可以普遍应用于很多大规模数据的计算问题，因此自发明MapReduce以后，Google公司内部进一步将其广泛应用于很多大规模数据处理问题。到目前为止，Google公司内有上万个各种不同的算法问题和程序都使用MapReduce进行处理。

>2003年和2004年，Google公司在国际会议上分别发表了两篇关于Google分布式文件系统和MapReduce的论文，公布了 Google的GFS和MapReduce的基本原理和主要设计思想。

>2004年，开源项目Lucene（搜索索引程序库）和Nutch（搜索引擎）的创始人Doug Cutting发现MapReduce正是其所需要的解决大规模Web数据处理的重要技术，因而模仿Google MapReduce，基于Java设计开发了一个称为Hadoop的开源MapReduce并行计算框架和系统。

>自此，Hadoop成为Apache开源组织下最重要的项目，自其推出后很快得到了全球学术界和工业界的普遍关注，并得到推广和普及应用。
MapReduce的推出给大数据并行处理带来了巨大的革命性影响，使其已经成为事实上的大数据处理的工业标准。

## mapreduce 的思想

###  对付大数据并行处理：分而治之：
>一个大数据若可以分为具有同样计算过程的数据块，并且这些数据块之间不存在数据依赖关系，则提高处理速度的最好办法就是采用“分而治之”的策略进行并行化计算。

>MapReduce采用了这种“分而治之”的设计思想，对相互间不具有或者有较少数据依赖关系的大数据，用一定的数据划分方法对数据分片，然后将每个数据分片交由一个节点去处理，最后汇总处理结果。

### 上升到抽象模型：Map与Reduce：

>MapReduce借鉴了函数式程序设计语言Lisp的设计思想。

>用Map和Reduce两个函数提供了高层的并行编程抽象模型和接口，程序员只要实现这两个基本接口即可快速完成并行化程序的设计。

>MapReduce的设计目标是可以对一组顺序组织的数据元素/记录进行处理。

>现实生活中，大数据往往是由一组重复的数据元素/记录组成，例如，一个Web访问日志文件数据会由大量的重复性的访问日志构成，对这种顺序式数据元素/记录的处理通常也是顺序式扫描处理。

### MapReduce提供了以下的主要功能：

>数据划分和计算任务调度：系统自动将一个作业（Job）待处理的大数据划分为很多个数据块，每个数据块对应于一个计算任务（Task），并自动调度计算节点来处理相应的数据块。作业和任务调度功能主要负责分配和调度计算节点（Map节点或Reduce节点），同时负责监控这些节点的执行状态，并负责Map节点执行的同步控制。

>数据/代码互定位：为了减少数据通信，一个基本原则是本地化数据处理，即一个计算节点尽可能处理其本地磁盘上所分布存储的数据，这实现了代码向数据的迁移；当无法进行这种本地化数据处理时，再寻找其他可用节点并将数据从网络上传送给该节点（数据向代码迁移），但将尽可能从数据所在的本地机架上寻 找可用节点以减少通信延迟。

>MapReduce提供了以下的主要功能：出错检测和恢复：以低端商用服务器构成的大规模MapReduce计算集群中，节点硬件（主机、磁盘、内存等）出错和软件出错是常态，因此 MapReduce需要能检测并隔离出错节点，并调度分配新的节点接管出错节点的计算任务。同时，系统还将维护数据存储的可靠性，用多备份冗余存储机制提 高数据存储的可靠性，并能及时检测和恢复出错的数据

## mapreduce的运行原理

>MapReduce 框架的核心步骤主要分两部分：Map 和Reduce。当你向MapReduce 框架提交一个计算作业时，它会首先把计算作业拆分成若干个Map 任务，然后分配到不同的节点上去执行，每一个Map 任务处理输入数据中的一部分，当Map 任务完成后，它会生成一些中间文件，这些中间文件将会作为Reduce 任务的输入数据。Reduce 任务的主要目标就是把前面若干个Map 的输出汇总到一起并输出。

![][1]

![][2]

![][3]

>解析上图,首先spilt分片是一个逻辑概念,分块即block是一个物理概念,默认是由分块来决定,一个快对应一个分片,这样的话,数据不用通过网络传输,提高效率,不过后续的倒排索引需要设置分片和分块的分离,分片是有inputformat来决定,我们可以通过定义inputformat来配置分片

>首先程序运行在内存中,在内存中完成kv的读取,这个过程设计4次磁盘的io操作inputformat,map完成,reduce开始,和reduce完成,网络传输:需要获取不是本机的数据,即不是同一个节点上的数据,所以reduce读取map的值,会从不同的节点上通过网络传输得到kv对

### map阶段

#### 分为shuffle(排序)和spill(溢写)

>首先从Map 端开始分析。当Map 开始产生输出时，它并不是简单的把数据写到磁盘，因为频繁的磁盘操作会导致性能严重下降。它的处理过程更复杂，数据首先是写到内存中的一个缓冲区，并做了一些预排序，以提升效率。
> 每个Map 任务都有一个用来写入输出数据的循环内存缓冲区。这个缓冲区默认大小是100MB，可以通过io.sort.mb 属性来设置具体大小。当缓冲区中的数据量达到一个特定阀值(io.sort.mb * io.sort.spill.percent，其中io.sort.spill.percent 默认是0.80)时，系统将会启动一个后台线程把缓冲区中的内容spill 到磁盘。在spill 过程中，Map 的输出将会继续写入到缓冲区，但如果缓冲区已满，Map 就会被阻塞直到spill 完成。spill 线程在把缓冲区的数据写到磁盘前，会对它进行一个二次快速排序，首先根据数据所属的partition 排序，然后每个partition 中再按Key 排序。输出包括一个索引文件和数据文件。如果设定了Combiner，将在排序输出的基础上运行。Combiner 就是一个Mini Reducer，它在执行Map 任务的节点本身运行，先对Map 的输出做一次简单Reduce，使得Map 的输出更紧凑，更少的数据会被写入磁盘和传送到Reducer。spill 文件保存在由mapred.local.dir指定的目录中，Map 任务结束后删除。
>每当内存中的数据达到spill 阀值的时候，都会产生一个新的spill 文件，所以在Map任务写完它的最后一个输出记录时，可能会有多个spill 文件。在Map 任务完成前，所有的spill 文件将会被归并排序为一个索引文件和数据文件，如图3 所示。这是一个多路归并过程，最大归并路数由io.sort.factor 控制(默认是10)。如果设定了Combiner，并且spill文件的数量至少是3（由min.num.spills.for.combine 属性控制），那么Combiner 将在输出文件被写入磁盘前运行以压缩数据。
?>对写入到磁盘的数据进行压缩（这种压缩同Combiner 的压缩不一样）通常是一个很好的方法，因为这样做使得数据写入磁盘的速度更快，节省磁盘空间，并减少需要传送到Reducer 的数据量。默认输出是不被压缩的， 但可以很简单的设置mapred.compress.map.output 为true 启用该功能。压缩所使用的库由mapred.map.output.compression.codec 来设定，
目前主要有以下几个压缩格式:

>DEFLATE 无DEFLATE .deflate 不支持不可以
gzip gzip DEFLATE .gz 不支持不可以
ZIP zip DEFLATE .zip 支持可以
bzip2 bzip2 bzip2 .bz2 不支持可以
LZO lzop LZO .lzo 不支持不可以
bbs.hadoopor.com --------Hadoop?技术论坛
>当spill 文件归并完毕后，Map 将删除所有的临时spill 文件，并告知TaskTracker 任务已完成。Reducers 通过HTTP 来获取对应的数据。用来传输partitions 数据的工作线程数由tasktracker.http.threads 控制，这个设定是针对每一个TaskTracker 的，并不是单个Map，默认值为40，在运行大作业的大集群上可以增大以提升数据传输速率。

##### combiner阶段

>combiner阶段是程序员可以选择的，combiner其实也是一种reduce操作，因此我们看见WordCount类里是用reduce进行加载的。Combiner是一个本地化的reduce操作，它是map运算的后续操作，主要是在map计算出中间文件前做一个简单的合并重复key值的操作，例如我们对文件里的单词频率做统计，map计算时候如果碰到一个hadoop的单词就会记录为1，但是这篇文章里hadoop可能会出现n多次，那么map输出文件冗余就会很多，因此在reduce计算前对相同的key做一个合并操作，那么文件会变小，这样就提高了宽带的传输效率，毕竟hadoop计算力宽带资源往往是计算的瓶颈也是最为宝贵的资源，但是combiner操作是有风险的，使用它的原则是combiner的输入不会影响到reduce计算的最终输入，例如：如果计算只是求总数，最大值，最小值可以使用combiner，但是做平均值计算使用combiner的话，最终的reduce计算结果就会出错。

>贴出shuffle和spill的图片

![][4]

![][5]

### reduce阶段

#### 也分为shuffle和spill

>现在让我们转到Shuffle 的Reduce 部分。Map 的输出文件放置在运行Map 任务的TaskTracker 的本地磁盘上（注意：Map 输出总是写到本地磁盘，但Reduce 输出不是，一般是写到HDFS），它是运行Reduce 任务的TaskTracker 所需要的输入数据。Reduce 任务的输入数据分布在集群内的多个Map 任务的输出中，Map 任务可能会在不同的时间内完成，只要有其中的一个Map 任务完成，Reduce 任务就开始拷贝它的输出。这个阶段称之为拷贝阶段。Reduce 任务拥有多个拷贝线程， 可以并行的获取Map 输出。可以通过设定mapred.reduce.parallel.copies 来改变线程数，默认是5。
?>Reducer 是怎么知道从哪些TaskTrackers 中获取Map 的输出呢？当Map 任务完成之后，会通知它们的父TaskTracker，告知状态更新，然后TaskTracker 再转告JobTracker。这些通知信息是通过心跳通信机制传输的。因此针对一个特定的作业，JobTracker 知道Map 输出与TaskTrackers 的映射关系。Reducer 中有一个线程会间歇的向JobTracker 询问Map 输出的地址，直到把所有的数据都取到。在Reducer 取走了Map 输出之后，TaskTrackers 不会立即删除这些数据，因为Reducer 可能会失败。它们会在整个作业完成后，JobTracker告知它们要删除的时候才去删除。
??> 如果Map 输出足够小，它们会被拷贝到Reduce TaskTracker 的内存中（缓冲区的大小
由mapred.job.shuffle.input.buffer.percent 控制，制定了用于此目的的堆内存的百分比）；如果缓冲区空间不足，会被拷贝到磁盘上。当内存中的缓冲区用量达到一定比例阀值（由mapred.job.shuffle.merge.threshold 控制），或者达到了Map 输出的阀值大小（由mapred.inmem.merge.threshold 控制），缓冲区中的数据将会被归并然后spill 到磁盘。
??>拷贝来的数据叠加在磁盘上，有一个后台线程会将它们归并为更大的排序文件，这样做节省了后期归并的时间。对于经过压缩的Map 输出，系统会自动把它们解压到内存方便对其执行归并。
?? > ?当所有的Map 输出都被拷贝后，Reduce 任务进入排序阶段（更恰当的说应该是归并阶段，因为排序在Map 端就已经完成），这个阶段会对所有的Map 输出进行归并排序，这个工作会重复多次才能完成。
?? > 假设这里有50 个Map 输出（可能有保存在内存中的），并且归并因子是10（由io.sort.factor 控制，就像Map 端的merge 一样），那最终需要5 次归并。每次归并会把10个文件归并为一个，最终生成5 个中间文件。在这一步之后，系统不再把5 个中间文件归并压缩格式工具算法扩展名支持分卷是否可分割成一个，而是排序后直接“喂”给Reduce 函数，省去向磁盘写数据这一步。最终归并的数据可以是混合数据，既有内存上的也有磁盘上的。由于归并的目的是归并最少的文件数目，使得在最后一次归并时总文件个数达到归并因子的数目，所以每次操作所涉及的文件个数在实际中会更微妙些。譬如，如果有40 个文件，并不是每次都归并10 个最终得到4 个文件，相反第一次只归并4 个文件，然后再实现三次归并，每次10 个，最终得到4 个归并好的文件和6 个未归并的文件。要注意，这种做法并没有改变归并的次数，只是最小化写入磁盘的数据优化措施，因为最后一次归并的数据总是直接送到Reduce 函数那里。
??>? 在Reduce 阶段，Reduce 函数会作用在排序输出的每一个key 上。这个阶段的输出被直接写到输出文件系统，一般是HDFS。在HDFS 中，因为TaskTracker 节点也运行着一个DataNode 进程，所以第一个块备份会直接写到本地磁盘。

#### 总结

>尽量直接放到内存中,尽量避免少的磁盘io读写和网络传输数据,会比较耗时.最后reduce阶段,其实目的也是尽可能多的在内存中完成,尽量减少磁盘io操作

## Mapreduce的代码实现

### 创建一个类MapreduceTestMap,继承mapper父类

>longwritable,text,intwritable,等其实就是java中的long,string和int,因为传递参数过程要进行网络传输,因此需要序列化和反序列化,而以上几个类是实现了writable的接口,因此可以支持序列化和反序列化.map的方法是每个kv对调用一次

``` stylus
//定义map,输入kv和输出kv
	public static class MapreduceTestMap extends Mapper<LongWritable, Text, Text, IntWritable>{

		private String[] infos;
		private Text oKey = new Text();
		private IntWritable oValue = new IntWritable(1);
		//key是偏移量,首个字符在读到的一整行,首个字符在全文的偏移量即占的位置,第几个字符.value是读到的一行的内容,context负责写进磁盘,传给reduce,context是引擎的上下文参数,用来传递参数,即kv对
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			//解析一行数据,转换成一个单词组成数组 \\s是一个正则表示任意空白字符,此时infos是一个字符数组里边由多个单词,单词对应索引,把需要的单词放进Text中,因为要传走
			infos = value.toString().split("\\s");
			//遍历是因为要所有的数据
			for (String i : infos) {
				//把单词形成一个kv对发送给reduce(单词,1)
				oKey.set(i);
				context.write(oKey, oValue);
			}
		}
	}
```
### 创建MapreduceTestReduce类,继承reducer类

>传过来的kv对,key是相同的key,而value是一个迭代器的集合,即是好多的value,所有的value属于同一个key,在map阶段定义每个value为1,那么统计单词出现次数,那么将value累加即可,得到总数,并且reduce方法是不同的key调用一次,因此调用次数会比map少.并且map阶段进行的合并的目的就是为了减少网络传输,尽可能少的数据传输给reduce,所以在传输之前可以进行小型的reduce计算

``` stylus
public static class MapreduceTestReduce extends Reducer<Text, IntWritable, Text, IntWritable>{

		private int sum;
		private IntWritable oValue = new IntWritable(0);
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
			sum = 0;
			//values是1,1,1,的数组,所以是定义成1,直接加,就可以统计总数
			for (IntWritable value : values) {
				sum += value.get();
			}
			//输出kv(单词,单词的计数)
			oValue.set(sum);
			context.write(key, oValue);
		}
```
### 创建job对象

``` stylus
//组装一个job到引擎上执行
	public static void main(String[] args) throws Exception {
		//构建一个configruation,用来配置hdfs的位置和mr的各项参数
		Configuration configuration = new Configuration();
		//创建job对象,不能直接new,需要getInstance
		Job job = Job.getInstance(configuration);
		//默认去这个类下去找包,因为此前导过包
		job.setJarByClass(MapreduceTest.class);
		//yarn本身会给任务分配id,不过我们配置名称,自己可以查看进程
		job.setJobName("第一个mr作业");
		//配置mr的执行类,map和reduce
		job.setMapperClass(MapreduceTestMap.class);
		job.setReducerClass(MapreduceTestReduce.class);
		//设置map输出kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		//配置reduce的输出kv类型,reduce是整个过程的输出故省略reduce
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		//设置数据源,等待被处理的数据,建多个path
		//可以调用多次,批处理数据
		Path path = new Path("/README.txt");
		FileInputFormat.addInputPath(job, path);
		
		//设置目标数据位置,path为一个目录,并且不能存在,重新创建
		Path path2 = new Path("/bd21/outputResult/seven");
		//不管存在不存在删除,可以保证多次运行,true代表若为目录会递归操作,全部删除
		FileSystem.get(configuration).delete(path2, true);
		FileOutputFormat.setOutputPath(job, path2);
		
		//启动作业,分布式计算提交给引擎,true代表处理过程
		boolean result = job.waitForCompletion(true);
        //执行成功,关闭服务,否则不关闭
		System.exit(result?0:1);
	}
```
>不同的包里的job代表不同的版本,用新版本即可

![][6]

>需要注意的问题,输入目录可以为多个,输出目录只能为一个,不清楚是写入还是覆盖,因为处理结果写入非常耗时,所以可以执行前先检查是否存在,若存在就删除.若map和reduce的输出kv一样,那么可以不配置map的outputclass

![][7]

![][8]

![][9]

### 实现功能:统计用户访问量

>1. 单词出现的次数,即上边贴出的代码,主要是接收map的value累加,存到新的value中
>2. 统计用户登录的次数,加一个判断
>3.IntWritable=1,可以保证reduce和combiner的逻辑和内容相同,reduce可以直接作为combiner在map上进行聚合减少reduce的工作量,因为map是分布式的,而reduce个数少,尽量做少的工作.

#### map

``` stylus
public static class UserVisitTimesMap extends Mapper<LongWritable, Text, Text, IntWritable>{

		private String[] infos;
		private Text oKey = new Text();
		private IntWritable oValue = new IntWritable(1);
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			infos = value.toString().split("\\s");
			if(infos[1].equals("login")){
				oKey.set(infos[0]);
				context.write(oKey, oValue);
			}
		}
	}
```
#### reduce

``` stylus
public static class UserVisitTimesReduce extends Reducer<Text, IntWritable,IntWritable,Text>{

		private int sum;
		private IntWritable oValue = new IntWritable(0);
		@Override
		protected void reduce(Text key, Iterable<IntWritable> values,
				Reducer<Text, IntWritable, IntWritable, Text>.Context context) throws IOException, InterruptedException {
			sum = 0;
			for (IntWritable value : values) {
				sum += value.get();
			}
			oValue.set(sum);
			context.write(oValue,key);
		}
		
}
```
### inputformat方式,读取源文件,需要指定源文件的地址fileinputformat.addInputPath(job, path);,然后由几种读取方式

>Textfileinputformat,这种是默认的生成的kv对格式是()longwriable,text)key是首字符在全文中的偏移量,value是一整行内容

>keyvaluefileinputformat,会把文本读成kv对,默认分隔符是"\t",如果有则分为两部分,前边是key,如果不存在,则一整行都是key,value为空,并且传给map的数据格式是Text,Text

>sequencefileinputformat,读的源文件,必须是sequenceoutputformat生成的文件,它读取数据会保留数据格式,那么就可以随意的定义读取的格式,不用因为key ,value的限制,而去重写好多方法

### 全排序和倒序排序

#### 抽样 和 分区
>定义抽样方式和数量,创建抽样数据的储存地址,把抽样规则加到job中,在job启动之前完成抽样并存储,最后把抽样的数据加到缓存中
>设置分区器,为TotalOrderPartitioner.class,设置分区规则为TotalOrderPartitioner.setPartitionFile(configuration, partitionfile);根据抽样文件进行分区,默认是hash,随机分配,即%reduce的数量

#### 代码实现,主要是设置规则,不用自己重写方法,若是倒序排序的话,需要重写比较规则,在此处都是直接用的框架自带的已经定义好的抽样或是分区方法,只是配置了一下规则,同样可以完全自定义,因为可能要实现其他的reader方法,比较麻烦,所以使用sequenceinputformat和totalorderPartition.

``` stylus
public static void main(String[] args) throws Exception {
		Configuration configuration = new Configuration();
		FileSystem hdfsFileSystem = FileSystem.get(configuration);
		
		//定义抽样,方式(用sequce读,必须和上次的输出格式一样,因为它会保留读取到数据的格式)和数量
		InputSampler.Sampler<IntWritable, Text> sampler = new InputSampler.RandomSampler<>(0.8, 5);
		//设置分区文件的路径,抽样的结果写到文件
		Path partitionfile = new Path("/bd76/total2/partition");
		hdfsFileSystem.delete(partitionfile, true);
		//设置后全排序的partition程序就会读取这个分区文件来完成按顺序进行分区???默认是hash
		TotalOrderPartitioner.setPartitionFile(configuration, partitionfile);
		
		//设置job
		Job job = Job.getInstance(configuration);
		job.setJarByClass(TotalSort.class);
		job.setMapperClass(Mapper.class);
		job.setReducerClass(TotalSortReduce.class);
		job.setJobName("全排序");
		job.setMapOutputKeyClass(IntWritable.class);
		job.setMapOutputValueClass(Text.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		
		//把分区文件加入分布式的缓存中????
		//设置分区器,默认是hashpartition
		job.setPartitionerClass(TotalOrderPartitioner.class);
		job.addCacheFile(partitionfile.toUri());
		//设置reduce节点个数,默认是1
		job.setNumReduceTasks(2);
		
		//如果是倒叙排序,那么指定job的sortcompare方法
		job.setSortComparatorClass(WritableDesx.class);
		
		Path path = new Path("/bd23/login/data");
		Path path2 = new Path("/bd76/total/result");
		hdfsFileSystem.delete(path2,true);
		//keyvalueinputformat map的输入会把文本文件读取成kv对,按照分隔符把一行分成两部分
		//前面key,后面value,如果分隔符不存在正行都是key,value为空,默认分隔符\t,
		//手动指定分隔符参数:mapreduce.input.keyvalueinerecordreader.key.value.separator
		//
		job.setInputFormatClass(SequenceFileInputFormat.class);
		//设置输出文件为sequencefile
		//job.setOutputFormatClass(SequenceFileOutputFormat.class);
		FileInputFormat.addInputPath(job, path);
		FileOutputFormat.setOutputPath(job, path2);
		
		//将随机抽样写入文件,在job启动之前启动抽样程序,并将抽样排序取出的中值写入文件,分布式抽样
		InputSampler.writePartitionFile(job, sampler);
		
		//启动job
		System.exit(job.waitForCompletion(true)?0:1);
		
	}
```
>全倒序排序,job.setSortComparatorClass(WritableDesx.class);自定义的比较器,实现intwriable.comparator,是按照key倒序,只重写compare方法即可

``` stylus
//设置key的比较规则,是相反的即是倒序排序,默认是从小到大
	public static class WritableDesx extends IntWritable.Comparator{

		@Override
		public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {
			return -super.compare(b1, s1, l1, b2, s2, l2);
		}
		
	}
```
### 二次排序的功能实现:1. 首先自定义封装对象的类型和自定义排序规则(重写compareTo方法,比较第一个字段再比较第二个字段)  2. 自定义分区规则,默认是hash,继承一下Partitioner类,重写getPartition方法,即可自行分区,返回的是reduce的标号   3.定义map把两个字段加到自定义封装对象中区,作为key,使用自定义的排序规则    4.定义reduce,把kv的值转换一下,需要遍历,因为可能是45 44 lisan,wangwu 需要写进去两次,不遍历会漏掉    5.创建job,set条件运行,不同的是 设置inpuformat方式为keyvalueinputformat,因为读取的源文件正好符合这种方式就分为两部分,所以使用这个     另外设置分区规则setPartitionerClass即自定义的规则
>1.
``` stylus
//自定义封装对象的类型,封装二次排序的第一个地段和第二个字段
	//自定义排序规则,第一个字段不同按照第一个排序,第一个相同按照第二个排序
	public static class TwoFiles implements WritableComparable<TwoFiles>{
        private String firstFiled;
        private int secondfiled;
        
		
		public String getFirstFiled() {
			return firstFiled;
		}
		public void setFirstFiled(String firstFiled) {
			this.firstFiled = firstFiled;
		}
		public int getSecondfiled() {
			return secondfiled;
		}
		public void setSecondfiled(int secondfiled) {
			this.secondfiled = secondfiled;
		}
		//序列化
		@Override
		public void write(DataOutput out) throws IOException {
			out.writeUTF(firstFiled);
			out.writeInt(secondfiled);
		}
        //反序列化
		@Override
		public void readFields(DataInput in) throws IOException {
			this.firstFiled = in.readUTF();
			this.secondfiled = in.readInt();
		}
        //比较方法
		//先比较第一个字段,若相等,再比较第二个字段
		@Override
		public int compareTo(TwoFiles o) {
			if(this.firstFiled.equals(o.firstFiled)){
				return this.secondfiled-o.secondfiled;
			}else{
				return this.firstFiled.compareTo(o.firstFiled);
			}
		}
	}
```
>2.

``` stylus
//自定义分区,用来将第一个字段相同的key值分区到同一个reduce节点上
	public static class TwoFilesPartitioner extends Partitioner<TwoFiles, NullWritable>{

		//返回值是一个int数字,这个数组是reduce的标号,numPartitions是reduce的个数
		@Override
		public int getPartition(TwoFiles key, NullWritable value, int numPartitions) {
		// (key.firstFiled.hashCode()&Integer.MAX_VALUE)得到正值
			int reduceNo = (key.firstFiled.hashCode()&Integer.MAX_VALUE)%numPartitions;
			return reduceNo;
		}
		
	}
```
![][10]

>3.

``` stylus
//定义map
	public static class SecondSortMap extends Mapper<Text, Text, TwoFiles, NullWritable>{

		private final NullWritable oValue = NullWritable.get();
		@Override
		protected void map(Text key, Text value, Mapper<Text, Text, TwoFiles, NullWritable>.Context context)
				throws IOException, InterruptedException {
			TwoFiles twoFiles = new TwoFiles();
			//将两个字段中的内容封装到对象中,把对象作为key传递到reduce
			twoFiles.setFirstFiled(key.toString());
			twoFiles.setSecondfiled(Integer.valueOf(value.toString()));
		    context.write(twoFiles, oValue);
		}
		
	}
```
>4.

``` stylus
//定义reduce
	public static class SecondSorReduce extends Reducer<TwoFiles, NullWritable, Text, Text>{

		private Text oKey = new Text();
		private Text oValue = new Text();
		
		@Override
		protected void reduce(TwoFiles key, Iterable<NullWritable> values,
				Reducer<TwoFiles, NullWritable, Text, Text>.Context context) throws IOException, InterruptedException {
			//若两个key一样,value虽然为空,那么也是由数量的是2,那么for循环,会将两个都加进去,不循环,会少
			//NullWritable 也可以计算次数,不过自身为空
			for (NullWritable value : values) {
				oKey.set(key.firstFiled);
				oValue.set(String.valueOf(key.secondfiled));
				context.write(oKey, oValue);
			}
			oKey.set("-------");
			oValue.set("");
			context.write(oKey, oValue);
		}
   }
```
>5.

``` stylus
//定义job
	public static void main(String[] args) throws Exception {
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		job.setJarByClass(SecondSort.class);
		job.setMapperClass(SecondSortMap.class);
		job.setReducerClass(SecondSorReduce.class);
		job.setMapOutputKeyClass(TwoFiles.class);
		job.setMapOutputValueClass(NullWritable.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		job.setJobName("二次排序");
		//job.setNumReduceTasks(2);
		Path path = new Path("/secondaryorder");
		Path path2 = new Path("/bd76/secondTwo");
		FileSystem.get(configuration).delete(path2, true);
		FileInputFormat.addInputPath(job, path);
		FileOutputFormat.setOutputPath(job, path2);
		//把文件内容以kv的形式读取出来发送给map
		job.setInputFormatClass(KeyValueTextInputFormat.class);
		job.setPartitionerClass(TwoFilesPartitioner.class);
		
		//设置分组比较器
		job.setGroupingComparatorClass(GroupToReduceCompare.class);
		
		System.exit(job.waitForCompletion(true)?0:1);
	}
```
#### 相同的key执行完后统一加"---",即保证相同的key,处理一套逻辑,那么需要让相同的key调用一个reduce方法,那么需要自定义分组比较器,继承wriableCompare,然后再job.setGroupingComparatorClass(GroupToReduceCompare.class);,设置自己的分组比较器

``` stylus
//定义分组比较器,不同的key,只要第一个字段相同,就调用一个reduce方法,目的是相同的key,
	//执行相同的代码或是对相同的key进行相同的操作或是统一 的操作,如key完后加"---"
	public static class GroupToReduceCompare extends WritableComparator{
        //构造方法里面要先向父类传递比较器要比较的数据类型
		public GroupToReduceCompare(){
			super(TwoFiles.class);
		}
		//重写compare方法自定义排序规则
		@Override
		public int compare(WritableComparable a, WritableComparable b) {
			TwoFiles tf1 = (TwoFiles)a;
			TwoFiles tf2 = (TwoFiles)b;
			return tf1.getFirstFiled().compareTo(tf2.getFirstFiled());
		}
		
	}
```
### Combiner,实质是一个reduce,不过是在本地,即只计算这个map上的数据,为了减少Reduce的压力,所有能聚合的先实现combiner聚合,再传值给Reduce进行最后聚合,它是在map阶段分区之后,在进行聚合,肯定只会各个区之间聚合,不然不同区聚合,reduce拿数据就会出现问题,现排序(Comparator)再分区(Partitioner)最后聚合(Combinner)

#### 设置分割符,例如keyvalueinputformat默认是"\t",修改这个有两种方式

![][11]

#### combiner的理解

![][12]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905726665.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905793135.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905811418.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905830980.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905863586.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905896186.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905936208.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905953753.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507905977282.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508167450126.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508168337754.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508168383892.jpg