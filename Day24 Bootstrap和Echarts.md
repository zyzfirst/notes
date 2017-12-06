---
title: Day24 spring boot
tags: spring boot,bootstrap,echarts
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


# spring boot 的应用
项目的BI层,属于一个项目的子项目,属于数据展示层,bi 即商业智能,即企业报表分析展现

![bi][6]

## bi 层的技术架构

![前后端][7]

后端渲染就是服务端把数据渲染成html,发送给前端页面
前端渲染 主要做前后端分离,项目比较大用到

- 1.前端
后端模板渲染
Jsp(springboot不推荐使用jsp，支持jsp)
Freemarker(应用广泛)、 volicity 、theamleaf(官方实例中使用较多)
前端模板渲染(前后端分离)
Angular、Vue、React
- 2.后端
MVC(控制显示逻辑的) springmvc
服务集成spring 
ORM(数据映射) mybatis

Model(数据模型，实体类型)(三层框架都是通过model实体类来进行跳转,调用方法 返回数据的封装对象)

Controller(mvc) 类
Service(service) 接口----实现类
Dao(ORM)  接口---实现类(mapper.xml)

传统web项目
代码=====> war =====> 放入java web容器(tomcat,jetty.weblogic)
springboot
代码=====> jar =====>独立运行
代码=====> war =====> 放入java web容器(tomcat,jetty.weblogic)

jsp需要servlet的支持,并且比较麻烦,一般官方不推荐使用jsp

### spring boot 的helloword

![enter description here][8]

![enter description here][9]

可以选择继承的框架,比较便捷,不需要麻烦的配置

这种相仿与使用模板创建spring boot项目,我们不用这种方式,因为这种方式生成的pom文件中,会有parent的依赖,而在bi层,我们的项目是一个子项目,已经有自己的parent,所以我们用手动添加依赖的方式创建

### 项目创建,直接在pom中引入依赖

``` stylus
<!-- 变量 版本控制 spring boot start -->
    <properties>
        <spring.version>1.5.9.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <!-- Spring Boot 的依赖,不用模板来创建项目-->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- spring boot web项目的常规依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- 日志依赖 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <!-- spring boot 开发工具依赖,对springboot热部署,自动启动功能 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- 用jsp开发的依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <version>${spring.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>8.5.14</version>
        </dependency>
        <!-- 支持jsp上的jstl标签 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!-- mybatis的依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
        <!-- 连接驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
    </dependencies>
    <!-- spring boot 支持maven的插件 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

创建jsp,没有此项的提醒,创建不出来

![添加jsp][10]

![修改path为webapp][11]

### 需要有一个继承SpringBootServletInitializer的类指明去加载这个类
某个类找不到,或是配置文件加载不到,可以尝试使用这个类run一下,加载进来

``` stylus
@MapperScan("com.zhiyou.bi.dao")
@SpringBootApplication
public class BIApplication extends SpringBootServletInitializer{
    public static void main(String[] args){
        SpringApplication.run(BIApplication.class);
    }

}
```

### 与ssm框架开发的不同点
- mapper文件在resource文件目录下
- application.properties   一些配置信息在这个文件中,且文件名就是这个名字
- model的实体类,在domain包下,官方指定
- 使用的是动态代理技术,dao层的实现类不用自己写,而由class main 加载加来

### 在domain中实体类中 自动生成set,get,tostring方法

![enter description here][12]

配置连接数据库的jdbc  ,关键字是  datasource

#配置mapper文件的位置
mybatis.mapper-locations=classpath:/mapper/*Mapper.xml


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512552731855.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512555917431.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512555960658.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512557333621.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512557751139.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512561970755.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512563431466.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512563538118.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512563582142.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512564092376.jpg
  [11]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512564137485.jpg
  [12]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1512566219329.jpg