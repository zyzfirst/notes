---
title: Day-23 idea创建maven项目
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 进入scala官网,寻找模板的坐标(或搜索关键字scala maven archetype)
>利用html源码,复制版本信息,运用箭头 锁定位置

![enter description here][1]

![enter description here][2]

# 新建maven项目,导入模板,运用模板创建

![enter description here][3]

# import changens,下载完成后修改pom和测试的代码

``` stylus
pom中修改
     <scala.tools.version>2.11</scala.tools.version>
     <scala.version>2.11.8</scala.version>
    改成自己的版本

    删除：
<dependency>
      <groupId>org.specs2</groupId>
      <artifactId>specs2_${scala.tools.version}</artifactId>
      <version>1.13</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.scalatest</groupId>
      <artifactId>scalatest_${scala.tools.version}</artifactId>
      <version>2.0.M6-SNAP8</version>
      <scope>test</scope>
    </dependency>
   
    删除：
<arg>-make:transitive</arg>

    删除 test.scala.samples下的所有代码
```
# 打开app,导入sdk环境,run一下,出现hellowordld即可



  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510836744736.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510836899257.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510836937208.jpg