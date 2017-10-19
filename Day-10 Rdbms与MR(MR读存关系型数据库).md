---
title: Day-10 Rdbms与MR(MR读存关系型数据库)
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
# 维度数据和事实数据
>维度数据是用来描述事物,描述的事物真实存在,名词,某个事物,并且相对稳定,不经常变化,且数量较少

>事实数据是用来描述事件的下订单(订单数据)动词,做某件事,并且相对更新较快,数据量比较大

# 维度数据的导入策略
>1.每日导入覆盖2.定期导入3.每日导增量4.直接读取关系型数据库

# 为什么读取关系型数据库rdbms
>因为每日导入或者定期导入比较麻烦,所以直接读取数据库,读取的是维度数据

>事实数据导出文本的格式,传到hdfs上,读取hdfs来做数据的分析

![][1]

![][2]

# 连接数据库的读写流程



  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508415734726.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1508415746343.jpg