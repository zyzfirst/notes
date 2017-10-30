---
title: hdfs_zyz
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# hdfs
## Hdfs概述

>1. Hdfs是Hadoop Distributed File System 的简称，它是Hadoop实现的一个分布式文件系统。
   2. Hdfs有高容错性的特点，并且设计用来部署在低廉的硬件上；而且它提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集的应用程序。
   3. Hdfs放宽了POSIX的要求，可以以流的形式访问文件系统中的数据。
   4. Hdfs总体上采用了master/slave 架构，主要由以下几个组件组成：Client 、NameNode 、Secondary 和DataNode

## Hdfs特性

>1. 保存多个副本，且提供容错机制，副本丢失或宕机自动恢复。默认存3份。
  2. 运行在廉价的机器上。
  3. 适合大数据的处理。多大？多小？HDFS默认会将文件分割成block，128M为1个block。然后将block按键值对存储在Hdfs上，并将键值对的映射存到内存中。如果小文件太多，那内存的负担会很重。

## Hdfs的写操作

![][1]

![][2]

![][3]

![][4]

>hdfs的启动:首先启动namenode,在通过RPC通信启动子节点,在启动namenode的时候会加载fsimage和edits,把元数据信息读取到内存中,fsimage中的数据和edits中的操作记录也会重新操作一遍,保证内存中是最新的数据,当有请求来向hdfs中写入一个文件时,元数据会以事务的形式把元数据信息修改到两个地方,namenode的内存中和edits中,edits就记录着数据变化的操作,当关闭hdfs时,fsimage和edits中的数据不会改变,当再次开启的时候,内存会读fsimage和edits,并且执行edits中的操作,从而保证内存中的元数据的完整性,读取完成后会重新创建fsimage和edits,以便记录下次的写操作,当新建完成前,此时namenode是安全模式只支持读操作.
block块的位置是通过本身没有储存在namenode的fsimage或者edits中,而是在每次启动由datanode自动检查data.dir目录获取所有的block信息,然后通过rpc通讯,传递给namenode的内存,hdfs的操作都是在内存中的
另一种机制,防止服务器的edits和fsimage过大,在再次启动的时候加载时间过长,所以会有secondarynamenode来监控edits,当达到某种容量时,就会新建一个空的edits,然后拷贝原来达到最大容量的edits,然后复制fsimage,在它的内存里进行加载操作,完成后,将数据更新到fsimage中,偷吃然后复制给namenode,并且也会进行备份,防止丢失.当下次操作时,就会拿来edits和本地的fsimage进行加载然后重建.

>进入safemode,就会进入只读不写的模式,如果自己系统某个文件某个块丢失,进会强制进去安全模式. hdfs dfsadmin -safemode leave 强制退出即可进行读写操作 hdfs dfsadmin -safemode enter 会手动进去安全模式.

>namenode的储存位置:在name/current中,元数据的操作日志(创建文件,删除修改文件)操作记录都会以日志的形式记录保存在edits中,元数据的具体内容(文件的名称,文件的大小,路径,创建者和时间)保存在fsimage中,每一个block的位置:在集群中的每个节点上它的具体位置不在fsimage中,位置在datanode中,当启动的时候datanode会自我检查所有的block信息以及id存放位置,然后回报给namenode,加载在内存 中,每次启动都会检查汇报,所以namenode就知道block的存放位置

### 各组件的作用

>.namenode:接受请求经过分析,指派datanode,分配任务转发请求(心跳,均衡(新加的节点,执行命令手动均衡,防止新加的倾斜),副本,元数据的处理)
>datanode:数据的读写请求执行和数据的保存操作
>secondaryNamenode:备份namenode

### 元数据

>元数据就是描述数据的数据,如记录block的id,大小,归属哪个文件,以及块存放的位置.同样数据库也有元数据,用来描述表的信息,如表名创建者和时间,字段(类型长度约束信息)数据库的元数据等都需要存储,用来存储元数据的表是数据字典表(mysql,orecal的数据字典表会有自短信息)(sql数据库模式(字段信息)和元祖(储存的数据),no-sql减少了模式,只有元组来储存数据)元数据的功能就是数据分析,质量控制,通过元数据的分析,来得出数据的质量

### 日志的命名规则

>log日志的命名规则:4段,第一是集群名称,第二是启动进程的名称,第三是进程或是角色的名称,第四是节点的hostname  .log是日志,.out是输出内容 

![][5]

![][6]

## hdfs 读操作

>会从namenode出得到block的地址 block host1 host2 host3    会按照顺序依次读取,以保证文件的完整

## Hdfs的代码实现

### 创建一个类,定义config属性和gethdfs

``` stylus
public class HdfsWork {
    //在classpath下配置core-site.xml 会默认去加载这个文件
	public static final Configuration conf = new Configuration();
	public static FileSystem hdfs;
	static {
		try {
			//filesystem不能直接实例化,只能通过静态方法得到对象
			hdfs = FileSystem.get(conf);
		} catch (IOException e) {
			System.out.println("无法连接");
			e.printStackTrace();
		}
	}
	}
```
>也可以用另一种方法

``` stylus
URI uri = new URI("hdfs://master:9000");
			hdfs = FileSystem.get(uri, null);
```


### 在类中写方法

#### 创建文件并写入内容

``` stylus
public static void creatDir(String fileName, String content) throws Exception {
		Path path = new Path(fileName);
		if (hdfs.exists(path)) {
			System.out.println("已经存在");
		} else {
			FSDataOutputStream outputStream = hdfs.create(path);
			//FSDataOutputStream提供的方法writeUTF能直接写入字符串,但是这种方式写入,必须用readUTF读
			outputStream.writeUTF(content);
			outputStream.flush();
			outputStream.close();
		}
    }
```
#### 读取已存在文件

``` stylus
public static void getContent(String fileName) throws Exception {
		Path path = new Path(fileName);
		FSDataInputStream inputStream = hdfs.open(path);
		//判断路径是存在并且是文件
		if (hdfs.exists(path) && !hdfs.isDirectory(path)) {
			String string = inputStream.readUTF();
			System.out.println(string);
			inputStream.close();
		} else {
			System.out.println("目录不存在或为文件夹");
		}

	}
```
#### 删除已存在文件

``` stylus
public static void deleteFile(String fileName) throws Exception {
		Path path = new Path(fileName);
		if (hdfs.exists(path)) {
			hdfs.delete(path, true);
		}
	}
```
#### 查看所有文件状态

``` stylus
public static void getStatus(String fileName) throws Exception {
		Path path = new Path(fileName);
		if (hdfs.exists(path)) {
			//会查看此路径下所有文件的状态,若为true的话,会递归操作子文件夹下的所有子文件
			RemoteIterator<LocatedFileStatus> iterator = hdfs.listFiles(path, true);
			while (iterator.hasNext()) {
				LocatedFileStatus status = iterator.next();
				System.out.println(status);
			}
		}
	}
```
#### 上传下载文件

``` stylus
// 上传文件
	public static void uploadFile(String src, String dst) throws Exception {
		hdfs.copyFromLocalFile(new Path(src), new Path(dst));
	}

	// 下载文件
	public static void downloadFile(String src, String dst) throws Exception {
		hdfs.copyToLocalFile(new Path(src), new Path(dst));
	}

```
#### 用流上传下载文件

``` stylus
// 用流上传文件
	public static void uploadFileByStream(String src, String dst) throws Exception {
		FileInputStream inputStream = new FileInputStream(new File(src));
		FSDataOutputStream outputStream = hdfs.create(new Path(dst));
		IOUtils.copy(inputStream, outputStream);
	}

	// 用流下载文件
	public static void downloadFileByStream(String src, String dst) throws Exception {
		FileOutputStream outputStream = new FileOutputStream(new File(dst));
		FSDataInputStream inputStream = hdfs.open(new Path(src));
		IOUtils.copy(inputStream, outputStream);
	}
```
>也可以用原来的方法

``` stylus
// 用fileinputstream上传
	public static void uploadFileByStream(String src, String dst) throws Exception {

		FileInputStream input = new FileInputStream(new File(src));
		FSDataOutputStream outputStream = hdfs.create(new Path(dst));
		int len = 0;
		byte[] b = new byte[1024];
		while((len = input.read(b)) != -1){
			outputStream.write(b, 0, len);
		}
		//IOUtils.copy(input, outputStream);
		input.close();
		outputStream.close();
	}
```
#### 查看文件状态,当前文件夹下的内容

``` stylus
// 查看文件状态,当前文件夹下的内容,只会查看当前文件夹下的文件或是文件夹
	public static void getFileStatus(String fileName) throws Exception {
		Path path = new Path(fileName);
		if (hdfs.exists(path)) {
			FileStatus[] status = hdfs.listStatus(path);
			for (FileStatus fileStatus : status) {
				System.out.println(fileStatus);
			}
		}
	}
```
#### 查看当前文件夹下的所有文件状态,递归操作

``` stylus
// 查看当前文件夹下的所有文件状态,
	public static void getAllFileStatus(Path path) throws Exception {
		if (hdfs.exists(path)) {
			FileStatus[] status = hdfs.listStatus(path);
			for (FileStatus fileStatus : status) {
				if (fileStatus.isDirectory()) {
					getAllFileStatus(fileStatus.getPath());
				} else {
					System.out.println(fileStatus);
				}
			}
		}
	}

```

#### 得到家目录

![][7]

#### getfilestatus 和listfilestatus

![][8]

#### newinstaence和get方法得到filesystem的区别

![][9]

#### 剪切文件

![][10]

#### get权限状态

![][11]

## hdfs的安全模式
>安全模式下,指启动了namenode还没有启动datanode,namenode等待datanode向他发送block数量的数据,在这种模式下,只能查看文件的系统上的文件个数,不能查看内容,因为内容储存在datanode上,而datanode还没有开启服务,并且不能创建新文件夹,上传文件,删除文件等操作

>手动开启或关闭命令

![][12]

![][13]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507908698738.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507908721447.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507908758354.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507908779275.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507909142797.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507909160374.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507910971239.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507910943527.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507911039298.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507911090876.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507911153197.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509366204455.jpg
  [13]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509366527943.jpg