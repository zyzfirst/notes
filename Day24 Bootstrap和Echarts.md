---
title: Day24 Bootstrap和Echarts
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 下载
官网下载,完整版即可

![echarts][1]

![bootsrap][2]

把链接得到的保存一下

![jquery下载][3]

bootstrap 的面板,可以将页面分成上下或是左右,把内容写在div中即可

``` stylus
    <div class="container">
        <div class="col-md-6">
        <div class="panel panel-warning">
            <div class="panel-heading">Panel heading without title</div>
            <div class="panel-body">
                <table class="table table-bordered">
                    <tr>
                        <td>ID</td>
                        <td>日期</td>
                        <td>酒店ID</td>
                        <td>酒店名称</td>
                        <td>房型ID</td>
                        <td>房型名称</td>
                        <td>房间数</td>
                        <td>当月预定数</td>
                        <td>当月入住数</td>
                    </tr>
                    <c:forEach var="hotelInfo" items="${hotelrtInfos}">
                        <tr>
                            <td>${hotelInfo.rrId}</td>
                            <td>${hotelInfo.dateMonth}</td>
                            <td>${hotelInfo.hotelId}</td>
                            <td>${hotelInfo.hotelName}</td>
                            <td>${hotelInfo.roomTypeId}</td>
                            <td>${hotelInfo.roomtypeName}</td>
                            <td>${hotelInfo.roomNum}</td>
                            <td>${hotelInfo.bookNum}</td>
                            <td>${hotelInfo.checkinNum}</td>
                        </tr>
                    </c:forEach>
                </table>
                </div>
            </div>
        </div>
        <div class="col-md-6">
            <div class="panel panel-success">
                <div class="panel-heading">图表</div>
                <div class="panel-body">
                    <div id="main" style="width: 500px;height:400px;"></div>
                </div>
            </div>
        </div>
    </div>
```

# 使用流程
## 先把bootstrap解压,并把文件和文件下拷贝到新建的web中的static文件目录下

## 把css,js ,jquery等引入到jsp中,css用link引入

``` stylus
<script type="text/javascript" src="/static/jquery.min.js"></script>
    <script type="text/javascript" src="/static/echarts.min.js"></script>
    <script type="text/javascript" src="/static/bootstrap-3.3.7/js/bootstrap.js"></script>
    <link href="/static/bootstrap-3.3.7/css/bootstrap.css" rel="stylesheet" type="text/css">
```

## bootstrap的使用,去官网找样式,组件,其实就是使用各种class

## echarts的使用

- 定义一个div有大小的div,使用id起一个名字
- 定义一个图表对象,把预留的div 加载进来 var echartobj =echarts.init(doument.getElementById("main"));
- 定义图标的参数 var option = {};
- 对象把参数导入图表 echartobj.setoption(option); 把参数记载进来

![echart使用流程][4]

``` stylus
xAxis : [
        {
            type : 'category',
            data : ['周一','周二','周三','周四','周五','周六','周日']
        }
    ],
    yAxis : [
        {
            type : 'value'
        }
    ],
```
x,y是维度 代表x,y轴  series可以是多个,这样可以展示多个数据

![多个数据条形图][5]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512552731855.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512555917431.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512555960658.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512557333621.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512557751139.jpg