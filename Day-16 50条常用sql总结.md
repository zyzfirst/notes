---
title: Day-16 50条常用sql总结
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 数据准备
--1.学生表
Student(S,Sname,Sage,Ssex) --S 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别
--2.课程表 
Course(C,Cname,T) --C --课程编号,Cname 课程名称,T 教师编号
--3.教师表 
Teacher(T,Tname) --T 教师编号,Tname 教师姓名
--4.成绩表 
SC(S,C,score) --S 学生编号,C 课程编号,score 分数

--创建测试数据

``` stylus
create table Student(S varchar(10),Sname varchar(10),Sage datetime,Ssex nvarchar(10))
insert into Student values('01' , '赵雷' , '1990-01-01' , '男')
insert into Student values('02' , '钱电' , '1990-12-21' , '男')
insert into Student values('03' , '孙风' , '1990-05-20' , '男')
insert into Student values('04' , '李云' , '1990-08-06' , '男')
insert into Student values('05' , '周梅' , '1991-12-01' , '女')
insert into Student values('06' , '吴兰' , '1992-03-01' , '女')
insert into Student values('07' , '郑竹' , '1989-07-01' , '女')
insert into Student values('08' , '王菊' , '1990-01-20' , '女')
create table Course(C varchar(10),Cname,varchar(10),T varchar(10))
insert into Course values('01' , '语文' , '02')
insert into Course values('02' , '数学' , '01')
insert into Course values('03' , '英语' , '03')
create table Teacher(T varchar(10),Tname,varchar(10))
insert into Teacher values('01' , '张三')
insert into Teacher values('02' , '李四')
insert into Teacher values('03' , '王五')
create table SC(S varchar(10),C varchar(10),score decimal(18,1))
insert into SC values('01' , '01' , 80)
insert into SC values('01' , '02' , 90)
insert into SC values('01' , '03' , 99)
insert into SC values('02' , '01' , 70)
insert into SC values('02' , '02' , 60)
insert into SC values('02' , '03' , 80)
insert into SC values('03' , '01' , 80)
insert into SC values('03' , '02' , 80)
insert into SC values('03' , '03' , 80)
insert into SC values('04' , '01' , 50)
insert into SC values('04' , '02' , 30)
insert into SC values('04' , '03' , 20)
insert into SC values('05' , '01' , 76)
insert into SC values('05' , '02' , 87)
insert into SC values('06' , '01' , 31)
insert into SC values('06' , '03' , 34)
insert into SC values('07' , '02' , 89)
insert into SC values('07' , '03' , 98)
```

# 1.查询"01"课程比"02"课程成绩高的学生的信息及课程分数
SELECT a.* ,b.score AS '01分数' ,c.score AS '02分数'
FROM student a 
INNER JOIN sc b 
ON a.S=b.S AND b.C='01'
INNER JOIN sc c 
ON a.S=c.S AND c.C='02'
WHERE b.score>c.score

# 2.查询"01"课程比"02"课程成绩低的学生的信息及课程分数
SELECT a.*,b.score AS '01分数',c.score AS '02分数'
FROM student a
INNER JOIN sc b
ON a.S=b.S AND b.C='01'
INNER JOIN sc c 
ON a.S=c.S AND c.C='02'
WHERE b.score<c.score

#  3.查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
SELECT b.S,a.Sname,AVG(score) AS avg_sc
FROM student a
INNER JOIN sc b
ON a.S=b.S 
GROUP BY b.S 
HAVING AVG(score)>=60

# 4.查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
SELECT b.S,a.Sname,AVG(score) AS avg_sc
FROM student a
INNER JOIN sc b
ON a.S=b.S
GROUP BY b.S
HAVING AVG(score)<60

# 5.查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
SELECT b.S,a.Sname,COUNT(C) AS sum_c,SUM(score) sum_s
FROM student a
INNER JOIN sc b
ON a.S=b.S
GROUP BY b.S

# 6.查询"李"姓老师的数量
SELECT COUNT(1) AS '李姓老师'
FROM teacher
WHERE Tname LIKE '%' '李' '%'
# 6.查询"李"姓老师的数量
SELECT COUNT(1) AS '李姓老师'
FROM teacher
WHERE Tname LIKE '%李%'

# 7.查询学过"张三"老师授课的同学的信息 
SELECT c.*
FROM student c
INNER JOIN sc d
ON c.S=d.S AND d.C =
  (
  SELECT b.C FROM course b WHERE b.C =
      (
         SELECT a.T FROM teacher a WHERE Tname = '张三'
      )
  )


# 8.查询没学过"张三"老师授课的同学的信息
SELECT * FROM student s
WHERE s.S NOT IN (
   SELECT ss.S FROM sc ss
   WHERE ss.C IN (
   SELECT b.C FROM course b WHERE b.C =
      (
         SELECT a.T FROM teacher a WHERE Tname = '张三'
      )
   )
)
 
# 9.查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
SELECT a.*
FROM student a
INNER JOIN sc b
ON a.S=b.S AND b.C='01'
INNER JOIN sc c
ON c.S=a.S AND c.C='02'

# 10.查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
SELECT a.* 
FROM student a
INNER JOIN sc b
ON a.S=b.S AND b.C='01'
LEFT JOIN sc c
ON c.S=a.S AND c.C='02'
WHERE c.C IS NULL AND b.C='01'

# 11.查询没有学全所有课程的同学的信息 
SELECT a.* 
FROM student a
LEFT JOIN sc b
ON b.S=a.S
GROUP BY b.S
HAVING COUNT(b.C)<(SELECT COUNT(1) FROM course)

# 12.查询至少有一门课与学号为"01"的同学所学相同的同学的信息 
SELECT s.* FROM student s WHERE s.S IN(
  SELECT S FROM sc WHERE C IN (
      SELECT C FROM sc WHERE S='01'  
  )AND s.S<>'01'
)
# 12.查询至少有一门课与学号为"01"的同学所学相同的同学的信息
SELECT DISTINCT a.* FROM student a JOIN sc s ON s.S=a.S AND s.C IN(
  SELECT C FROM sc WHERE S='01'
)AND s.S <> '01'




# 13.查询和"01"号的同学学习的课程完全相同的其他同学的信息 
SELECT a.* 
FROM(
  SELECT s.S ,GROUP_CONCAT(s.C ORDER BY s.C) AS c_one FROM sc s  GROUP BY s.S HAVING s.S<>'01'
)s
,(
 SELECT GROUP_CONCAT(ss.C ORDER BY ss.C) AS c_one FROM sc ss WHERE ss.S='01'
) ss
,student a
WHERE s.c_one=ss.c_one AND a.S=s.S


# 14.查询没学过"张三"老师讲授的任一门课程的学生姓名 
SELECT a.Sname
FROM student a
LEFT JOIN sc b
ON b.S=a.S
WHERE b.S NOT IN (
SELECT c.S FROM sc c WHERE c.C IN(
  SELECT a.C FROM course a
  JOIN teacher b
  ON b.T=a.T AND b.Tname='张三')

)OR b.C IS NULL
GROUP BY 1

# 15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 
SELECT a.S,a.Sname,AVG(score)
FROM student a
JOIN sc b
ON b.S=a.S
WHERE b.S IN (
 
 SELECT a.S FROM sc a
 WHERE a.score<60 
 GROUP BY a.S
 HAVING COUNT(a.S)>1
 
)
GROUP BY a.S

# 16、检索"01"课程分数小于60，按分数降序排列的学生信息
SELECT a.*
FROM student a
JOIN sc b
ON b.S=a.S AND b.C='01'
WHERE b.score<60
ORDER BY b.score DESC

# 17、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
SELECT S,AVG(score),
       MAX(CASE C WHEN '01' THEN score ELSE 0 END)AS c_01,
       MAX(CASE C WHEN '02' THEN score ELSE 0 END)AS c_02,
       MAX(CASE C WHEN '03' THEN score ELSE 0 END)AS c_03
FROM sc 
GROUP BY S
ORDER BY AVG(score) DESC

# 18、查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
# 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
SELECT a.C AS '课程ID'
       ,b.Cname AS '课程name'
       ,MAX(score) AS '最高分'
       ,MIN(score) AS '最低分'
       ,AVG(score) AS '平均分'
       ,SUM(CASE WHEN a.score>=60 THEN 1 ELSE 0 END)/COUNT(*) AS '及格率'
       ,SUM(CASE WHEN a.score<80 AND a.score>=70  THEN 1 ELSE 0 END)/COUNT(*) AS '中等率'
       ,SUM(CASE WHEN a.score<90 AND a.score>=80 THEN 1 ELSE 0 END)/COUNT(*) AS '优良率'
       ,SUM(CASE WHEN a.score>=90 THEN 1 ELSE 0 END)/COUNT(*) AS '优秀率'
FROM course b
JOIN sc a
ON a.C=b.C
GROUP BY a.C

# 19、按各科成绩进行排序，并显示排名
SELECT t.* 
       ,(SELECT COUNT(1) FROM SC WHERE C = t.C AND score > t.score) + 1 AS px
FROM sc t 
ORDER BY t.C,px


# 20、查询学生的总成绩并进行排名
SET @rank=0;
SELECT c.* ,@rank :=@rank+1 AS rank
FROM(
SELECT a.S,b.Sname
     ,SUM(a.score) AS sum_s
FROM sc a
JOIN student b
ON b.S=a.S
GROUP BY a.S
ORDER BY SUM(score) DESC)c


# 21、查询不同老师所教不同课程平均分从高到低显示 
SELECT t.Tname,c.Cname,AVG(score) AS '平均分'
FROM course c
JOIN teacher t
ON t.T=c.T
JOIN sc s
ON s.C=c.C
GROUP BY s.C
ORDER BY AVG(score) DESC

# 22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩
SELECT a.*,b.score
FROM student a
JOIN sc b
ON b.S=a.S
ORDER BY b.score DESC
LIMIT 1,2


# 23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比 
SELECT a.C,b.Cname
       ,SUM(CASE WHEN a.score<100 AND a.score>=85 THEN 1 ELSE 0 END) AS '[100-85]'
       ,SUM(CASE WHEN a.score<100 AND a.score>=85 THEN 1 ELSE 0 END)/COUNT(*) AS '[100-85]百分比'
       ,SUM(CASE WHEN a.score<85 AND a.score>=70 THEN 1 ELSE 0 END) AS '[85-70]'
       ,SUM(CASE WHEN a.score<85 AND a.score>=70 THEN 1 ELSE 0 END)/COUNT(*) AS '[85-70]百分比'
       ,SUM(CASE WHEN a.score<70 AND a.score>=60 THEN 1 ELSE 0 END) AS '[70-60]'
       ,SUM(CASE WHEN a.score<70 AND a.score>=60 THEN 1 ELSE 0 END)/COUNT(*) AS '[70-60]百分比'
       ,SUM(CASE WHEN a.score<60 AND a.score>=0 THEN 1 ELSE 0 END) AS '[0-60]'
       ,SUM(CASE WHEN a.score<60 AND a.score>=0 THEN 1 ELSE 0 END)/COUNT(*) AS '[0-60]百分比'
FROM sc a
JOIN course b
ON b.C=a.C
GROUP BY a.C

# 24、查询学生平均成绩及其名次 
SET @rank=0;
SELECT c.*,@rank :=@rank+1 AS rank
FROM(
SELECT a.S,b.Sname,AVG(score) AS avg_s
FROM sc a
JOIN student b
ON b.S=a.S
GROUP BY a.S
ORDER BY AVG(score) DESC)c

# 25、查询各科成绩前三名的记录
(SELECT a.S,a.C,a.score FROM sc a WHERE C='01' ORDER BY score DESC LIMIT 0,3)
UNION 
(SELECT b.S,b.C,b.score  FROM sc b WHERE C='02' ORDER BY score DESC LIMIT 0,3)
UNION
(SELECT c.S,c.C,c.score FROM sc c WHERE C='03' ORDER BY score DESC LIMIT 0,3)




# 26、查询每门课程被选修的学生数 
SELECT C ,COUNT(1) FROM sc GROUP BY C

# 27、查询出只有两门课程的全部学生的学号和姓名 
SELECT a.S,b.Sname
FROM student b
JOIN sc a
ON a.S=b.S
GROUP BY a.S
HAVING COUNT(1)=2
 
# 28、查询男生、女生人数 
SELECT SUM(CASE WHEN Ssex='男' THEN 1 ELSE 0 END) AS man_s,SUM(CASE WHEN Ssex='女' THEN 1 ELSE 0 END) AS woman_s
FROM student

SELECT a.ssex
	,COUNT(a.Ssex)
FROM student a
GROUP BY Ssex
# 29、查询名字中含有"风"字的学生信息
SELECT * FROM student WHERE Sname LIKE '%风%'

# 30、查询同名同性学生名单，并统计同名人数 
SELECT a.* ,COUNT(1) AS peo
FROM student a
JOIN student b ON a.S<>b.S AND a.Sname=b.Sname

# 31、查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime) 
SELECT * FROM student WHERE YEAR(Sage)=1990

# 32、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号
SELECT C,AVG(score) FROM sc GROUP BY C ORDER BY AVG(score) DESC,C DESC

# 33、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩
SELECT a.S,b.Sname,AVG(score)
FROM sc a
JOIN student b
ON a.S=b.S
GROUP BY a.S
HAVING AVG(score)>=85
 
# 34、查询课程名称为"数学"，且分数低于60的学生姓名和分数 
SELECT b.Sname,a.score
FROM sc a
JOIN student b
ON a.S=b.S
JOIN course c
ON c.C=a.C AND c.Cname='数学'
WHERE a.score<60


# 35、查询所有学生的课程及分数情况；
SELECT *
FROM sc a
INNER JOIN student b
ON a.s=b.s
INNER JOIN course c
ON a.c=c.c
 
# 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数；
SELECT b.Sname,c.Cname,a.score
FROM sc a
JOIN student b ON b.S=a.S
JOIN course c ON c.C=a.C
WHERE a.score>70
 
# 37、查询不及格的课程
SELECT s.Sname,a.C,c.Cname
FROM sc a
JOIN course c ON c.C=a.C
JOIN student s ON s.S=a.S
WHERE a.score<60


# 38、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名；
SELECT a.S,s.Sname
FROM sc a 
JOIN student s
ON s.S=a.S
WHERE a.C='01' AND a.score>=80
 
# 39、求每门课程的学生人数 
SELECT a.C,c.Cname,COUNT(a.C)
FROM sc a
JOIN course c
ON c.C=a.C
GROUP BY a.C

# 40、查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩
SELECT a.*,MAX(b.score)
FROM sc b
JOIN student a ON a.S=b.S
JOIN course c ON c.C=b.C
JOIN teacher t ON t.T=c.T AND t.Tname='张三'
 

# 41、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 
SELECT DISTINCT a.S,a.C,a.score FROM sc a
JOIN sc b
ON b.score=a.score AND a.C<>b.C
ORDER BY a.S

# 42、查询每门功成绩最好的前两名 
(SELECT * FROM sc WHERE C='01' ORDER BY score DESC LIMIT 0,2)
UNION
(SELECT * FROM sc WHERE C='02' ORDER BY score DESC LIMIT 0,2)
UNION
(SELECT * FROM sc WHERE C='03' ORDER BY score DESC LIMIT 0,2)



# 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列 
SELECT a.C,COUNT(1)
FROM sc a
GROUP BY a.C HAVING COUNT(1)>5
ORDER BY COUNT(1) DESC,a.C ASC
 
# 44、检索至少选修两门课程的学生学号
SELECT a.S,b.Sname,COUNT(1)
FROM sc a 
JOIN student b 
ON a.S=b.S
GROUP BY a.S
HAVING COUNT(1)>=2
 
# 45、查询选修了全部课程的学生信息 
SELECT b.*
FROM sc a 
JOIN student b 
ON a.S=b.S
GROUP BY a.S
HAVING COUNT(1)=(SELECT COUNT(1) FROM course)

# 46、查询各学生的年龄
SELECT s.*,YEAR(CURRENT_DATE)-YEAR(Sage) AS age
FROM student s

# 47、查询本周过生日的学生
SELECT a.Sname
FROM student a
WHERE MONTH(DATE(a.Sage)) BETWEEN MONTH(DATE_SUB(CURRENT_DATE(),INTERVAL WEEKDAY(CURRENT_DATE())+1 DAY)) AND MONTH(DATE_ADD(CURRENT_DATE(),INTERVAL 6 - WEEKDAY(CURRENT_DATE()) DAY))
AND DAY(a.Sage) BETWEEN DAY(DATE_SUB(CURRENT_DATE(),INTERVAL WEEKDAY(CURRENT_DATE())+1 DAY)) AND DAY(DATE_ADD(CURRENT_DATE(),INTERVAL 6 - WEEKDAY(CURRENT_DATE()) DAY))

# 47、查询本周过生日的学生
SELECT b.*
FROM(
	SELECT *
		,DATE_SUB(CURDATE(),INTERVAL WEEKDAY(CURDATE()) DAY) starttime 
		,DATE_SUB(CURDATE(),INTERVAL WEEKDAY(CURDATE())-6 DAY) endtime 
		
	FROM student a
) b
WHERE DATE_FORMAT(b.Sage,'%Y-%m-%d') >= starttime AND DATE_FORMAT(b.Sage,'%Y-%m-%d') <= endtime






# 48、查询下周过生日的学生
SELECT a.* 
FROM (
  SELECT *
         ,DATE_ADD(CURRENT_DATE,INTERVAL 7-WEEKDAY(CURRENT_DATE)DAY) starttime
         ,DATE_ADD(CURRENT_DATE,INTERVAL 7+6-WEEKDAY(CURRENT_DATE)DAY) endtime
  FROM student b
)a
WHERE DATE_FORMAT(a.Sage,'%Y-%m-%d')>=starttime AND DATE_FORMAT(a.Sage,'%Y-%m-%d')<=endtime


# 49、查询本月过生日的学生
SELECT s.* FROM student s WHERE MONTH(Sage)=MONTH(CURRENT_DATE)

# 50、查询下月过生日的学生
SELECT s.* FROM student s WHERE MONTH(Sage)=MONTH(CURRENT_DATE)+1

# 总结
1.内链接 外连接的on条件后只能跟原始条件,不能跟查询后的条件,要想跟查询后的条件,需要在having后加,group分组后的条件写在group后
2.子查询的结果只能查出来一个,多于一行报错Subquery returns more than 1 row
3.in和exists的区别(大表小表的效率问题)  not in和not exists尽量使用后者
4.行转列  MAX(CASE Subject WHEN '语文' THEN Score ELSE 0 END) AS '语文',
  
5.distinct去重,可以去除多个字段DISTINCT a.S,a.C,a.score 会按照三个字段都不相同的记录去重
6.set @rank=0;可以定义变量 用@表示
7.左连接大于子查询,not exists,exists代替 not in,用in替代 or(or的效率极低),用>=代替>,
8.某一组内,记录完全相同,用group_concat
  group_concat()会计算哪些行属于同一组，将属于同一组的列显示出来。要返回哪些列由函数参数(就是字段名)决定。分组必须有个标准，就是根据group by指定的列进行分组。

9.日期时间的总结:
 date_sub 减去时间可以使day,week,month等
 current_date 获取当前时间
 year,month,day等获取时间的 特定格式
 date_format(date,'%Y-%m-%d') 转换时间格式,返回值还是date类型
 date_add 加时间 同date_sub
 weekday(date) 返回当前时间的星期索引,0是星期一
 date_sub的用法: date_sub(current_date,interval 数值 数值类型(day,week,month等))

