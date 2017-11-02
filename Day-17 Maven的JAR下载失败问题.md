---
title: Day-17 Maven的JAR下载失败问题
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# buildpath
修改jre,改为workspave的jre

![][1]

# 把相应目录删除再重新下载,update一下

# 用cmd让maven下载jar
打开properties ,进入项目的根目录,复制地址,打开cmd,然后进入到根目录,运行mvn install命令即可

![][2]

![][3]

![][4]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509638219199.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509638388461.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509638426396.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1509638491034.jpg