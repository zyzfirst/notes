---
title: Day-18 Phoenix
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 安装
## 客户端的安装
解压配置环境变量

>拷贝phoenix-4.7.0-HBase-1.1-bin.tar.gz文件到linux的/usr/tools目录下面
解压：
tar –zxvf phoenix-4.7.0-HBase-1.1-bin.tar.gz
生成新的目录phoenix-4.7.0-HBase-1.1-bin
将其配置到环境变量中去
#phoenix
export PHOENIX_HOME=/usr/tools/phoenix-4.7.0-HBase-1.1-bin
export PHOENIX_CLASSPATH=$PHOENIX_HOME/lib
export PATH=$PATH:$PHOENIX_HOME/bin
配置完以后是环境变量生效
source /etc/profile

## 服务端的安装
拷贝jar包到hbase的lib下,因为它是嵌套在hbase里的

>将/usr/tools/phoenix-4.7.0-HBase-1.1-bin目录下面的phoenix-4.7.0-HBase-1.1-server.jar文件拷贝到每一台HRegionServer的hbase安装目录的lib目录下面去
cp phoenix-4.7.0-HBase-1.1-server.jar /usr/tools/hbase-1.2.0/lib/
scp phoenix-4.7.0-HBase-1.1-server.jar root@jokeros2:/usr/tools/hbase-1.2.0/lib/
scp phoenix-4.7.0-HBase-1.1-server.jar root@jokeros3:/usr/tools/hbase-1.2.0/lib/
重新启动hbase
stop-hbase.sh
start-hbase.sh

## 配置python环境

yum install python-argparse

否则报错

![][1]

## 启动测试
sqlline.py jokeros1,jokeros2,jokeros3:2181
sqlline是phoenix的客户端
关闭服务命令: !quit

![][2]

## 使用SQuirrel来连接phoenix

- 把需要的两个jar包拷贝到SQurriel的扩展目录下

![][3]
- 配置driver

![][4]
- 测试连接

![][5]

### 在SQuirrel中进行sql操作

``` stylus
select PK_NAME from system.catalog where table_schem='SYSTEM'and table_name='STATS'and column_name='PHYSICAL_NAME';
create table phoenix_user(
  user_id integer not null primary key
  ,user_name varchar(20)
  ,age integer
  ,birthday varchar(20)
)
select * from phoenix_user;

--插入数据
upsert into phoenix_user(user_id,user_name,age,birthday) values(1,'张三',12,'2012-4-5');
upsert into phoenix_user(user_id,user_name,age,birthday) values(2,'张4',142,'2012-7-5');
upsert into phoenix_user(user_id,user_name,age,birthday) values(3,'张5',122,'2017-4-12');

--修改数据,必须跟随主键
upsert into phoenix_user(user_id,age) values(1,15);

-- 年龄大于20岁的删除
delete from phoenix_user where user_id=1;
delete from phoenix_user where user_id>5;

create table phoenix_us(
  user_id integer not null primary key
  ,user_name varchar(20)
  ,age integer
  ,birthday varchar(20)
)
```
其实phoenix提供一种映射,把非关系型的hbase转换成关系型的表结构,table_schema表示hbase中的库的名字,

### 基本操作

【分类】
SELECT
UPSERT VALUES
UPSERT SELECT
DELETE
CREATE
DROP
ALTER TABLE
CREATE INDEX 


【CRUD】
查询操作
SELECT * FROM TEST;
SELECT a.* FROM TEST;
SELECT DISTINCT NAME FROM TEST;
SELECT ID, COUNT(1) FROM TEST GROUP BY ID;
SELECT NAME, SUM(VAL) FROM TEST GROUP BY NAME HAVING COUNT(1) > 2;
SELECT 'ID' COL, MAX(ID) AS MAX FROM TEST;
SELECT * FROM TEST LIMIT 1000; 

插入操作
UPSERT INTO TEST VALUES('foo','bar',3);
UPSERT INTO TEST(NAME,ID) VALUES('foo',123); 
UPSERT INTO test.targetTable(col1, col2) SELECT col3, col4 FROM test.sourceTable WHERE col5 < 100
UPSERT INTO foo SELECT * FROM bar; 

删除操作
DELETE FROM TEST;
DELETE FROM TEST WHERE ID=123;
DELETE FROM TEST WHERE NAME LIKE 'foo%'; 



【其他操作】
CREATE TABLE my_schema.my_table ( id BIGINT not null primary key, date DATE not null)
CREATE TABLE my_table ( id INTEGER not null primary key desc, date DATE not null,
    m.db_utilization DECIMAL, i.db_utilization)
    m.DATA_BLOCK_ENCODING='DIFF'
CREATE TABLE stats.prod_metrics ( host char(50) not null, created_date date not null,
    txn_count bigint CONSTRAINT pk PRIMARY KEY (host, created_date) )
CREATE TABLE IF NOT EXISTS my_table ( id char(10) not null primary key, value integer)
    DATA_BLOCK_ENCODING='NONE',VERSIONS=?,MAX_FILESIZE=2000000 split on (?, ?, ?) 
DROP TABLE my_schema.my_table
DROP VIEW my_view 
ALTER TABLE my_schema.my_table ADD d.dept_id char(10) VERSIONS=10
ALTER TABLE my_table ADD dept_name char(50)
ALTER TABLE my_table ADD parent_id char(15) null primary key
ALTER TABLE my_table DROP COLUMN d.dept_id
ALTER TABLE my_table DROP COLUMN dept_name
ALTER TABLE my_table DROP COLUMN parent_id
ALTER TABLE my_table SET IMMUTABLE_ROWS=true 
CREATE INDEX my_idx ON sales.opportunity(last_updated_date DESC)
CREATE INDEX my_idx ON log.event(created_date DESC) INCLUDE (name, payload) SALT_BUCKETS=10
CREATE INDEX IF NOT EXISTS my_comp_idx ON server_metrics ( gc_time DESC, created_date DESC )
    DATA_BLOCK_ENCODING='NONE',VERSIONS=?,MAX_FILESIZE=2000000 split on (?, ?, ?) 



# view的作用
在数据库管理系统当中， view是描述数据库中信息的⼀种⽅式。 若要将数据项按某种特定的序列排列、 突出某些数据项，
或者只显⽰特定的数据项， 这些都可以通过view(视图)来实现。 对于任何数据库来说， 可能有⼀些视图需要定义。 与拥有
少量数据项的数据库相⽐， 拥有很多数据项的数据库可能有更多的视图。 就像虚拟表⼀样， 视图本⾝并不真正的存储信
息， 但仅仅只是从⼀个或多个已经存在的表中将数据取出。 虽然很⽆常， ⼀个视图能通过存储其查询标准， ⽽被重复的访
问。

view不存储数据,存储sql,它是实时的sql的查询,所以原表增加数据不影响view,而当原表删除数据的时候,就会影响view,因为它的数据查询不到了,并没有存储数据

应用场景:1. 表关联4个,每次查询比较麻烦,所以可以先建一个view 直接查询view,可以理解为对查询做了封装
2. view 可以做一些数据限制,因为数据库的限制只到表级别,要想限制某些行记录不允许查看,那么可以先添加限制条件,然后建一个view ,那么就可以在这个表中查询

# JavaAPI操作phoenix的表
- 1.创建maven项目,添加phoenix的core依赖
- 2.需要注意它的upsert的事务默认为false,可以手动提交,可以设置为true,由connection.setAutoCommit

``` stylus
package com.zhiyou.phoenix.jdbctest;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Random;


/**  
* @ClassName: PhoenicJdbcTest  
* @Description: TODO  
* @author zyz  
* @date 2017年11月4日 上午11:07:55  
*   
*/
public class PhoenicJdbcTest {

	public static final String URL="jdbc:phoenix:master,slaver1,slaver2:2181";
	//driver_class 使用的是PhoenixDriver,全包名可以ctrl+shift+t  然后复制org.apache.phoenix.jdbc.PhoenixDriver
	public static final String DRIVER_CLASS = "org.apache.phoenix.jdbc.PhoenixDriver";
    public static final String USER_NAME = "root";
    public static final String PASSWORD = "";
    private Connection connection;
    public PhoenicJdbcTest() throws Exception{
    	Class.forName(DRIVER_CLASS);
    	this.connection = DriverManager.getConnection(URL, USER_NAME, PASSWORD);
    }
    //查询数据
    public void findData() throws Exception{
    	Statement statement = connection.createStatement();
    	String sql = "select * from phoenix_user";
    	ResultSet resultSet = statement.executeQuery(sql);
    	while(resultSet.next()){
    		System.out.println(resultSet.getInt("user_id")
    				+resultSet.getString("user_name")
    				);
    	}
    }
    //插入数据
    public void insertData() throws Exception{
    	String sql ="upsert into phoenix_user (user_id,user_name,age,birthday) values(?,?,?,?)";
    	PreparedStatement statement = connection.prepareStatement(sql);
         Random random = new Random();
        for(int i = 0; i<10;i++){
        	statement.setInt(1, 14+i);
        	statement.setString(2, "user"+i);
        	statement.setInt(3, random.nextInt(30)+1);
        	statement.setString(4, "201"+random.nextInt(10)+random.nextInt(12)+random.nextInt(28));
           //添加批量
        	statement.addBatch();
        	//System.out.println(statement.execute());
       }
    	//设置事务自动提交
    	connection.setAutoCommit(true);
    	//提交批量
        statement.executeBatch();
        //phoenix默认的connection 是false,可以手动提交,也可以设置自动提交
        //statement.getConnection().commit();
   }
    public void cleanUp(){
    	if(connection!=null){
    		try {
				connection.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
    	}
    }
    public static void main(String[] args) throws Exception {
		PhoenicJdbcTest jdbcTest = new PhoenicJdbcTest();
		//jdbcTest.findData();
		jdbcTest.insertData();
		jdbcTest.cleanUp();
	}
}

```






  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509806793574.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509807676991.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509806990694.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509807031279.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509807048162.jpg