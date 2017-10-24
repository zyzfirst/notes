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

``` stylus
create table department(
  dep_id string,
  department string,
  address string

)row format delimited fields terminated by '\t' stored as textfile;
```


#### 创建外部表

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
>只会克隆表结构,不会克隆数据

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






  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508854268069.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508858163433.jpg