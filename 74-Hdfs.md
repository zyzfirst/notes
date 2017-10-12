HDFS

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


## Hdfs的API

>创建文件系统的配置文件`CONF = new Configuration();`
	- 默认加载`classpath`下`core-site.xml`文件,所以需要将hadoop的配置文件拷贝下来
	- 也可以通过`CONF`的`set`方法进行设置如:`conf.set("fs.defaultFS", "hdfs://hadoop:9000");`
- 创建文件系统`hdfs = FileSystem.get(CONF);`
	- `FileSystem`是抽象类,返回值为具体的子类,如果是`HDFS`返回值的类型是`DistributedFileSystem`
	- ![](./1507819690505.png)
- 创建路径`Path path = new Path(fileName);`
- 判断路径是否存在`boolean orExist = hdfs.exists(path);`
- 判断是否是文件夹`hdfs.isDirectory(path)`
- 判断是否是文件`hdfs.isFile(path)`
- 创建输入流`FSDataInputStream input = hdfs.open(path);`
- 创建输出流`FSDataOutputStream output = hdfs.create(path);`
- 上传文件`hdfs.copyFromLocalFile(srcPath, desPath);`
- 下载文件`hdfs.copyToLocalFile(srcPath, desPath);`
- 递归删除文件`hdfs.delete(path, true);`
- 创建文件夹`boolean orSuccess = hdfs.mkdirs(path)`
- 获取目录下所有FileSatus`RemoteIterator<LocatedFileStatus> listFiles = hdfs.listFiles(path, true);`或者`FileStatus[] status = hdfs.listStatus(path);`
- Path相关
	- 通过Path获取路径`path.toString()`
	- 通过Path获取文件名`path.getName()`
- FileStatus相关
	- 通过Path生成FileStatus`hdfs.getFileStatus(path)`
	- 通过FileSatus获取Path`fileStatus.getPath()`
