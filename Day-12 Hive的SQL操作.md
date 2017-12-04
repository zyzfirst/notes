---
title: Day-12 Hive的SQL操作
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# Hive的数据类型
>**Numeric Types 基本数据类型**
Tinyint 1 字节的有符号整数 -128~127
SmallInt 2 个字节的有符号整数 -32768~32767
Int 4 个字节的有符号整数 -2147483648 ~ 2147483647
BigInt 8 个字节的有符号整数 9223372036854775808 ~ 9223372036854775807
Float 单精度浮点数
Double 双精度浮点数

>**Date/Time Types日期类型**
TimeStamp 整数 支持Unix timestamp，可以达到纳秒精度
Date 日期 0000-01-01 ~ 9999-12-31，常用String 代替

>**Misc Types 微型类型**
Binary 字节数组
Boolean 布尔类型，true 或者false true、false

>**String Types字符串**
string 
char
varchar

>**Complex Types 复杂类型**
array array 是一组具有相同类型和名称的变量的集合，这些变量称为数据的元素，每 个数组元素编号都从0 开始。
map map 是一组键值对元素集合。可通过 字段名[‘key’]来访问值。
struct 值类似于对象，有属性和值，可以用.来访问值。

``` stylus
create table employees (
name string,
salary float,
subname array<string>,
deductions map<string,float>,
address struct<provice:string,city:string,zip:int>
);
```
# SQL语言分类
>数据定义语言：简称DDL(Data Definition Language)，用来定义数据库对象：数据库，表，
列等。关键字：create，alter，drop等
数据操作语言：简称DML(Data Manipulation Language)，用来对数据库中表的记录进行更
新。关键字：insert，delete，update等
数据控制语言：简称DCL(Data Control Language)，用来定义数据库的访问权限和安全级
别，及创建用户。
数据查询语言：简称DQL(Data Query Language)，用来查询数据库中表的记录。关键字：
select，from，where等

![][1]

## DDL的使用

### 概念
>hive本身不提供存储,它是存储在hdfs上,元数据存储在关系型的数据库上

>导入数据是,内部表和外部表都会将hdfs上的文件剪切到响应的hive目录,如果是在本地,那么会复制一份储存在hive目录下,

>外部表且使用了location ,那么就会做一个映射,并不会将hdfs上的文件剪切到hive目录下,而是对此目录下的文件做了一个映射,元数据同样存储在关系型数据库中,那么此时删除表,只会删除元数据,即删除表的结构,数据还是会在hdfs上,而内部表不是映射,是复制到hive目录下,那么数据和元数据都由hive管理,那么在删除的时候就都删除了

>导入数据应该注意的是:尽量用外部表的location,避免无意数据的丢失,另外数据的格式尽量和元数据一样,保证数据的准确性,之后再利用cast( '' as 数据类型) 转换,操作生成的这张表

### 具体的操作

#### 创建内部表
>row format delimited fields terminated by '\t' stored as textfile 是因为需要到入数据 
>会将hdfs上的/input/data目录下的数据转移到/input/table_data目录下。删除test表后，会将test表的数据和元数据信息全部删除，即最后/input/table_data下无数据，当然/input/data下再上一步已经没有了数据！

     如果创建内部表时没有指定location，就会在/user/Hive/warehouse/下新建一个表目录，其余情况同上。

``` stylus
create table department(
  dep_id string,
  department string,
  address string

)row format delimited fields terminated by '\t' stored as textfile;
```
#### 内部表和外部表的区别
> 1、在导入数据到外部表，数据并没有移动到自己的数据仓库目录下(如果指定了location的话)，也就是说外部表中的数据并不是由它自己来管理的！而内部表则不一样；
  >  2、在删除内部表的时候，Hive将会把属于表的元数据和数据全部删掉；而删除外部表的时候，Hive仅仅删除外部表的元数据，数据是不会删除的！
 >3. 在创建内部表或外部表时加上location 的效果是一样的，只不过表目录的位置不同而已，加上partition用法也一样，只不过表目录下会有分区目录而已，load data local inpath直接把本地文件系统的数据上传到hdfs上，有location上传到location指定的位置上，没有的话上传到hive默认配置的数据仓库中。
>外部表相对来说更加安全些，数据组织也更加灵活，方便共享源数据。 

#### 创建外部表
>把hdfs上/input/edata/下的数据转到/user/hive/warehouse/et下，删除这个外部表后，/user/hive/warehouse/et下的数据不会删除，但是/input/edata/下的数据在上一步load后已经没有了！数据的位置发生了变化！

``` stylus
create external table ext_employee(
   emp_id string,
   emp_name string,
   status string,
   salary string,
   status_salary string,
   in_work_date string,
   leader_id string,
   dep_id string
   )
   row format delimited fields terminated by '\t' stored as textfile;
```


#### 复制表
>把一个的查询结果直接作为新建表的输入结果create table a as select * from b;就将b的结果和内容复制给了a,可以运用cast(字段名 as 数据类型)别名 来修改字段类型和字段名,即修改元数据

``` stylus
create table de_employee as select cast (emp_id as int) emp_id
   ,emp_name,status,cast(salary as double)salary,cast(status_salary as double)status_salary,cast(in_work_date as date)in_work_date,cast(leader_id as int)
  leader_id ,cast(dep_id as int) dep_id from employee;
```


#### 克隆表
>只会克隆表结构,不会克隆数据,然后可以使用insert into select语法进行数据的导入

``` stylus
create table employee_clone like employee
```


#### 导入数据到内部表或外部表
>从hdfs上会剪切到hive目录,从本地会赋值到hive目录

``` stylus
load data inpath '/dep.txt' overwrite into table department;
 load data local inpath '/bigdata/hiveData/emp_dep/employee.txt' overwrite into table ext_employee;
```


#### 建立映射导入数据到外部表
>会将hdfs上的文件与hive做一个映射,,并不移动源数据

``` stylus
create external table txt_department02(
dep_id string,
dep_name string,
address string
) r
ow format delimited
fields terminated by '\t'
location '/emp_dep'
```
#### 查看表结构
>三种方式formatted 最详细,包括创建时期,内部表等信息,extended也包括这些信息,不过是一行内容不清楚

``` stylus
describe employee;
describe formatted department;
describe extended employee;
```
#### 修改表

>(1) 修改表名
--将表名从dealerinfo 改为dealer_info
alter table dealerinfo rename to dealer_info;

>(2) 添加字段
--在dealer_info 表添加一个字段provinceid，int 类型
alter table dealer_info add columns (provinceid int );

>(3) 修改字段
alter table dealer_info replace columns (dealerid int,dealername string,cityid int,joindate
date,provinceid int);
修改字段，只是修改了Hive 表的元数据信息（元数据信息一般是存储在MySql 中），并不
对存在于HDFS 中的表数据做修改。修改需要字段是序列化反序列化类型

#### 删除表
>drop table if exists dealer_info;

## DQL的使用

### 判断语句
>case when 条件 then (true)___else___end
>nvl 如果第一个字段为空显示0,如果不为空显示字段

``` stylus
select emp_id
         ,emp_name
         ,salary + case when status_salary is null then 0 else status_salary end as zong 
  from de_employee
  select emp_id
         ,emp_name
         ,salary + nvl(status_salary,0) 
  
  from de_employee
```
### 去重

``` stylus
select distinct status
  from de_employee;
  select status 
  from de_employee
  group by status;
```
>去重可以去多个字段,从查询出每个部门中不重复的职位(distinct)

``` stylus
select distinct dep_id
         ,status       
  from de_employee;
```
![][2]

### 字段的大小写
>相当于一个函数,在需要用到大小写的时候调用函数,参数是字段名称

``` stylus
--以小写的形式展示职位(lower())upper()
  select emp_id
         ,emp_name
         ,lower(status)
         ,upper(status)
  from de_employee
  --忽略大小写匹配职位等于‘ANALYST’的记录
select *
from de_employee
where upper(status)='ANAYLST'
```
### year函数,返回年份是int类型,参数是date类型的字段名

``` stylus
--查询出2016年入职的员工,year(字段)返回年份直接判断
select *
from de_employee
where year(in_work_date)=2014

```
### 字段语法
>is null    is not null    字段为空不为空
>between a and b   not between a and b 字段的范围在区别不在区间
>like  模糊查询'%描述%'
>**hive中不支持非等于的子查询和join(非等于连接,or连接,><连接说的是条件)**

### insert select语法

``` stylus
-- insert select 语法,overwrite会先把表清空,然后再插进去数据,insert into是直接插入
create table de_employee_leader like de_employee;
insert into de_employee_leader
select * 
from de_employee where leader_id is null;
select * from de_employee_leader;
insert overwrite table de_employee_leader
select * 
from de_employee where leader_id is null;
```
### 子查询
>**设置笛卡尔积:**set hive.strict.checks.cartesian.product=false;

``` stylus
-- 工资高于平均薪水的人的信息 hive中不支持非等于的子查询判断
-- 这种不成立,
select *
from de_employee a
where exists(
      select avgsalary from (select avg(salary) avgsalary from de_employee )b
      where a.salary > b.avgsalary
)
-- 笛卡尔积 先set设置一下,允许笛卡尔积改成false.然后再执行
set hive.strict.checks.cartesian.product=false;
select a.*
       ,b.*
from de_employee a,(select avg(salary) avgsalary from de_employee) b
where a.salary>b.avgsalary
-- 查询出最高薪水的人的信息,不支持=值操作用in替代

select *
from de_employee
where salary in (select max(salary) from de_employee)
-- in在后面天骄子查询结果作为判断条件时,如果子查询中有索引的话,in是用不到
-- 这个索引的,一次in对子查询的效率比较低
```
### Union
>将两个或多个查询结果集组合为单个结果集,是result的纵向合并,需要注意的是结果集的字段个数和类型必须保持一致
>UNION和UNION ALL的区别：
union 检查重复
union all 不做检查
比如 select 'a' union select 'a' 输出就是一行 a
比如 select 'a' union all select 'a' 输出就是两行 a

### join
>JOIN用于按照ON条件联接两个表，主要有四种：
INNER JOIN：内部联接两个表中的记录，仅当至少有一个同属于两表的行符合联接条件
时，内联接才返回行。我理解的是只要记录不符合ON条件，就不会显示在结果集内。
LEFT JOIN / LEFT OUTER JOIN：外部联接两个表中的记录，并包含左表中的全部记录。
如果左表的某记录在右表中没有匹配记录，则在相关联的结果集中右表的所有选择列表列均
为空值。理解为即使不符合ON条件，左表中的记录也全部显示出来，且结果集中该类记录
的右表字段为空值。
RIGHT JOIN / RIGHT OUTER JOIN：外部联接两个表中的记录，并包含右表中的全部记
录。简单说就是和LEFT JOIN反过来。
FULL JOIN / FULL OUTER JOIN：完整外部联接返回左表和右表中的所有行。就是LEFT
JOIN和RIGHT JOIN和合并，左右两表的数据都全部显示。

>内连接是两张表的互相过滤,全外连接是两张表的互相补充

### 优化和一些设置
>1.开启本地服务 local    
//true为开启
hive.exec.mode.local.auto=false
//本地处理的最大文件个数,超过4个map就不在本地处理
hive.exec.mode.local.auto.input.files.max=4
//本地处理的最大字节数
hive.exec.mode.local.auto.inputbytes.max=134217728

>2.设置reduce个数
//默认的reduce的个数的参数
hive.exec.reducers.bytes.per.reducer=256000000
hive.exec.reducers.max=1009
//自己设置reduce的个数
mapreduce.job.reduces=2


>3.map端关联
------前提是处理多表关联问题-----------
//是否自动开启map端join,默认为true开启
hive.auto.convert.join=true
//决定是否使用map端join,如果关联表有一个小于这个参数的配置就自动开启map端join
//因为满足了小表的条件,系统认为有一张小表
hive.mapjoin.smalltable.filesize=25000000
//手动开启map端join,原来的老版本
select /*+MAPJON(department)*/ 正常查询语法


>4.order by
//默认不设置limit也可以运行   
set hive.mapred.mode=nonstrict; (default value / 默认值)
//可以设置为strict,这样在运行order by的时候必须制定limit(每个map取limit个传到reduce)
set hive.mapred.mode=strict;

### 复制并修改表的结构
>外部表和源文件格式保持一致,为了保证数据的完整与不失真,但是我们做数据分析可能需要将格式转换成自己计算的格式,方便进行计算,利用cast(.. as ..)别名

``` stylus
-- 重新建表
create table dep as
select cast(dep_id as int)dep_id
       ,cast(department as string)dep_name
       ,cast(address as string)dep_address
from department
select * from dep
```

### left right join的条件限制
>left 和right  对地位低的加限制条件,那么只能加在on后边,因为它会过滤地位高得表,对地位高的可以加在where后边
>count(1)是几行记录,如果部分人数为null,那么算一行记录,可视会显示部门人数为1,所以count(一个唯一的字段)

``` stylus
create table  dep_sort as
select * from(
select a.dep_id
       ,a.dep_name
       ,a.dep_address
       ,count(b.emp_id) p_num
       ,sum(nvl(salary,0))a_salary
from dep a
left join de_employee b
on a.dep_id=b.dep_id
and b.status='Coder'
group by a.dep_id
         ,a.dep_name
         ,a.dep_address
)c
sort by p_num
```
### group by,order by ,sort by,distribute by,cluster by
>**group by :** 分组,按照某个字段分组,相同的数据记录会合并
>**order by :** 全排序,是全局的排序,在hive中,处理此类的机制是只有一个reduce保证是全局的排序,,因此效率低,一般不用,使用的话需要指定limit减少工作量  **set hive.mapred.mode=strict;**  在这个模式下,必须指定limit,否则报错,select * from test order by id limit 2;
>**sort by  :**如果设置了mapred.reduce.tasks=n,那么一个表中会有n个文件,每个reduce一个文件,并且这个排序是在reducer之前,因此能保证单个reduce有序,并不能保证全局排序
>**distribute by :** 按照指定字段对数据进行分区,即发布到不同的reduce,可以自定义方法,保证有序的进入不同的reduce,默认是hash值,因此使用默认也会随机的分配,只不过hash值一样的会分到同一个reduce
**正常情况 distribute by 字段和sort by字段 可以实现全局的排序**
>**cluster by ;** cluster by 除了具有 distribute by 的功能外还兼具 sort by 的功能。 但是排序只能是顺序排序，不能指定排序规则为asc 或者desc。cluster by status 相当于 distribute by status sort by status

``` stylus
create table emp_distrubute4 as 
select * from de_employee
distribute by status sort by status,salary desc
```
![][3]

>只有分区有问题,因为按照hash分的reduce,是有两个,reduce和reduce之间没有排序,所以不是全局的排序

### 复杂的数据类型
>需要致命复杂类型的导入数据的集合之间或是map之间的分隔符

``` stylus
-- 数据类型
create table test_serializer(
  string1 string
  ,int1 int
  ,tinnyint1 tinyint
  ,smallint1 smallint
  ,bigint1 bigint
  ,boolean1 boolean
  ,float1 float
  ,double1 double
  ,list1 array<string>
  ,map1 map<string,int>
  ,struct1 struct<sint:int,sboolean:boolean,sstring:string>
  ,union1 uniontype<float,boolean,string>
  ,enum1 string
  ,nullableint int
  ,bytes1 binary
  ,fixed1 binary
)
row format delimited
fields terminated by ','
collection items terminated by ':'
map keys terminated by '#'
lines terminated by '\n'
NULL DEFINED AS 'NULL'
stored as textfile
show tables
load data local inpath '/bigdata/hiveData/testdatatype.csv' overwrite into table test_serializer

```
##### array
>可以查单个字段,可以按照索引查询,可以将list展开,可以查询包含什么的

![][4]

![][5]

![][6]

##### map
>查询整个map和所有的key,value等 也可以根据key来查询value ,map1[key]

![][7]

![][8]

![][9]

##### struct
>查询对象,查询属性用对象打点调

![][10]

### 文件格式

![][11]

>parquet是Impala使用比较广泛的,ORC是hive使用比较广泛的
>hive 默认⽀持的⽂件格式有很多， 其中arvo、 orc、 Parquet、 Compressed Data Storage、 LZO Compression等等，

#### 设置文件储存格式

``` stylus
-- 文件格式
create table avro_employee
stored as avro
as 
select * from de_employee
dfs -cat /user/hive/warehouse/bd14.db/avro_employee/000000_0

create table orc_employee
stored as orc
as 
select * from de_employee
```
#### orc格式支持ACID原子操作delete,update

``` stylus
-- 支持事务型操作 update ,delete
-- 1.参数设置
-- client
set hive.support.concurrency=true;
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
-- server
set hive.compactor.initiator.on=true;
set hive.compactor.worker.threads=1;

-- 2.表定义:表必须是分桶表,文件存储格式必须是orc ,表参数添加TBLPROPERTLES("transactional"="true")
create table acid_employee(
 emp_id int
 ,emp_name string
 ,salary double
)
clustered by (emp_id) into 2 buckets
stored as orc
tblproperties("transactional"="true")
-- 插入数据
insert into table acid_employee
select emp_id
       ,emp_name
       ,salary
from de_employee

select * from acid_employee
describe formatted acid_employee

delete from acid_employee where emp_id=1008
update acid_employee set salary=10000 where emp_id=1006
```
### 分区表
>一般数据量非常大,而用户只关心一个月的数据,那么进行全盘扫面的话太耗费时间,因此按照月增量疆场查询的数据进行分区,每次查询只在响应的分区,这样提高速度
>最常用的分区条件:1.时间:年 月 日 2.行政区划 : 省 地市区县 3.具体的业务类型(不常用): 商品品类 服饰 数码等
>分区表需要指定partitioned by (字段,字段类型)  在insert into 数据的时候,只要字段有值就能自动导入响应的分区
>分区的本质其实就是在表的目录下建子目录,它是一个文件夹,不过他也可以当做表的虚拟字段也可以查询,在文件夹下储存文件(没有二级分区的话)

``` stylus
- 分区表的创建
create table p_orders(
  order_id int
  ,order_date string
  ,customer_id int
  ,order_status string
)
partitioned by(date_month string)
row format delimited
fields terminated by '|'
-- 新增分区
alter table p_orders add partition(date_month='201709');
alter table p_orders add partition(date_month='201708');
alter table p_orders add partition(date_month='201707');
--删除分区
alter table p_orders drop partition(date_month='201709')
-- 静态导入分区数据,若没有分区自动创建
-- 当使用loaddata往分区表中加载数据的时候,hive是不会对数据做任何的转换的,他只是单纯的把数据源赋值到分区的目录下
load data local inpath '/bigdata/hiveData/orderdata/orders' overwrite into table p_orders partition(date_month='201709')
-- 动态导入分区数据
-- 设置配置,默认
--hive.exec.dynamic.partition=true
--hive.exec.dynamic.partition.mode=strict
set hive.exec.dynamic.partition=true; 
set hive.exec.dynamic.partition.mode=nonstrict;
-- 使用insert into select语句来完成数据的动态导入
-- 1.创建临时表
create temporary table tmp_orsers(
   order_id int
  ,order_date string
  ,customer_id int
  ,order_status string
)
row format delimited
fields terminated by '|'
stored as textfile;
-- 2.把数据加载到临时表
load data local inpath '/bigdata/hiveData/orderdata/orders' overwrite into table tmp_orsers;
-- 3.用insert into select从临时表中取数据并且转换分区字段导入到分区表中
-- 两个函数 to_date把string转换成date,date_format是date类型的格式转换
insert into table p_orders partition(date_month)
select order_id 
  ,order_date
  ,customer_id 
  ,order_status
  ,date_format(to_date(order_date),'yyyyMM')as date_month
from tmp_orsers;
```
### 二级分区

``` stylus
create table p_test(
 	test1 string 
	,test2 string
)
partitioned by (date_day string,date_hour string)

show tables;

alter table p_test add partition(date_day='20171025',date_hour='01');
alter table p_test add partition(date_day='20171025',date_hour='02');
alter table p_test add partition(date_day='20171025',date_hour='03');
alter table p_test add partition(date_day='20171026',date_hour='01');
alter table p_test add partition(date_day='20171026',date_hour='02');
alter table p_test add partition(date_day='20171026',date_hour='03');
```
>创建表的时候用partitioned by(字段,类型)  加载数据的时候 用partition(字段="")

``` stylus
-- 分区字段可以当做一个字段来查询
select * from p_orders
where date_month>'201306'
describe formatted p_orders
```
>可以查询分区字段的数据

### 分桶表 分桶和文件对应,分区和目录对应
>对于每一个表（table）或者分区， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织。Hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。
把表（或者分区）组织成桶（Bucket）有两个理由：
（1）获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。
（2）使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。

**当从桶表中进行查询时，hive会根据分桶的字段进行计算分析出数据存放的桶中，然后直接到对应的桶中去取数据，这样做就很好的提高了效率。**
**分桶表需要先开启配置分桶,然后指定分几个桶,会自动计算从哪个桶取数据,效率更高**

``` stylus
-- 分桶,数据量不大,但是经常查询
create table pb_orders(
  order_id int 
  ,order_date string 
  ,customer_id int
  ,order_status string
)
partitioned by (date_month string)
clustered by(customer_id) sorted by(customer_id) into 2 buckets
stored as textfile
-- 动态导入分桶表数据
insert into table pb_orders partition(date_month)
select order_id 
  ,order_date
  ,customer_id 
  ,order_status
  ,date_format(to_date(order_date),'yyyyMM')as date_month
from tmp_orders
-- 查询测试
select * from pb_orders where date_month='201307' and customer_id=5125
select * from tmp_orders where customer_id=5125 and date_format(to_date(order_date),'yyyyMM')='201307'
```
### 文件压缩
>需要设置开启压缩与设置压缩格式

``` stylus
-- 文件压缩
-- 开启压缩
set hive.exec.compress.output=true
-- 自己设置压缩格式
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.GzipCodec
drop table compress_order
create table compress_order as 
select * from tmp_orders
describe formatted compress_order
```
### 阿里云镜像

>- 添加如下代码在正确的位置,在conf的settting.xml下

![][12]

## 出错

![job in use][13]

错误信息,job in use   提示job一致在占用,无法删除,因为没有启动jobhistoryserver服务
在运行job时,applicationmaster会将mr的job根据mapered-site.xml中的配置,将jobhistory的信息保存到hdfs上,当父进程检索到hdfs上的信息时,会将内存释放,job任务完成,若是检索不到信息,那么job的进程就会一直在后天运行


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508854268069.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508858163433.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939353674.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939689707.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939708385.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939736217.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939774355.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939785941.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939803750.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508939850153.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508940892389.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509024996374.jpg
  [13]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512141725172.jpg