---
title:Day-11eclipse maven 导出项目依赖的jar包 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 一、导出到默认目录 targed/dependency
从Maven项目中导出项目依赖的jar包：进入工程pom.xml 所在的目录下，执行如下命令：
1
mvn dependency:copy-dependencies
或在eclipse中，选择项目的pom.xml文件，点击右键菜单中的Run As,见下图红框中，在弹出的Configuration窗口中，输入 dependency:copy-dependencies后，点击运行
maven项目所依赖的jar包会导出到targed/dependency目录中。
# 二、导出到自定义目录中
在maven项目下创建lib文件夹，输入以下命令：
1
mvn dependency:copy-dependencies -DoutputDirectory=lib
maven项目所依赖的jar包都会复制到项目目录下的lib目录下
# 三、设置依赖级别
同时可以设置依赖级别，通常使用compile级别
1
mvn dependency:copy-dependencies -DoutputDirectory=lib -DincludeScope=compile

![][1]

![][2]

![][3]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508407158024.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508407408861.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508407366773.jpg