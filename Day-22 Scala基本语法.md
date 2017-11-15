---
title: Day-22 Scala基本语法
tags: scala,OO,函数
grammar_cjkRuby: true
---


# Scala概述

scala是一种可扩展的语言,是一种混合功能编程语言,它是面向对象和面向函数的结合,它由scalac编译成.class文件即字节码文件,运行在JVM虚拟机

## 特性
- **面向对象语言:** Scala是一种纯粹的面向对象语言，每一个值都是一个对象。 对象的类型和行为由类和特征描述
- **函数编程语言:** Scala也是一种函数式语言，每个函数都是一个值，每个值都是一个对象，所以每个函数都是一个对象。
Scala提供了一个轻量级的语法来定义匿名函数，它支持高阶函数，它允许函数嵌套，并支持currying
- **运行在JVM上:** Scala代码被编译成由Java虚拟机(JVM)执行的Java字节代码，这意味着Scala和Java具有通用的运行时平台。因此，可以轻松地从Java迁移到Scala。
- **静态类型的语言:** Scala与其他静态类型语言(C，Pascal，Rust等)不同，它不提供冗余类型的信息。 在大多数情况下，您不需要指定类型，当然减少了不必的重复。
- **可执行java代码:** Scala能够使用Java SDK的所有类以及自定义Java类，或您最喜欢的Java开源项目。
- **可以支持并发和同步** Scala允许您以有效的方式表达一般的编程模式。它减少了线路数量，并帮助程序员以类型安全的方式进行编码。它允许您以不变的方式编写代码，这使得应用并发和并行性(Synchronize)变得容易。

## 与java的不同之处
- 所有类型都是对象
- 函数是对象
- 闭包
- 性状
- 嵌套函数
- 类型推断
- 域特定语言DCL的支持

# 开发环境的安装与配置
## 1.安装JDK环境,配置环境变量

## 2.安装scala的windows版本,解压,next即可

## 3.基于intelJ的scala的插件安装

![安装流程][1]

采用在线安装,按照上图流程即可,新建一个scala的project,首次使用需要createSDK,选择工作空间,和jdk环境,create之后会出现已经安装的SDK,若没有的话,点击brower自己选择目录添加进来即可

![新建][2]

![createSDK][3]

# 代码执行的两种方式
## 交互式REPL

进入到scala的bin目录,使用scala进入交互式界面,直接在界面上执行代码
REPL:即交互式命令行,写一句执行一句,不适合编码,适合做数据分析

![交互式][4]

## 脚本模式

即在intellJ等一些开发软件编写代码,会直接先由scalac编译成.class文件,然后再由JVM运行.使用scala 文件名来编译执行,交互式的是每次执行都会编译,也可以scalac先编译再去执行,也可以直接编译执行

![交互式][5]

![脚本模式][6]


## 强,弱类型的定义

- 强类型: (java,scala)在定义的时候必须指定数据类型,并且一旦定义号数据类型,数据类型就不能在发生改变,而scala中var a = 123;a.toDouble  可以转换成double类型,但是它的变量名称已经变为res,即已经不是哪个变量了

![enter description here][7]

- 弱类型: 在定义变量的时候不指定数据类型,在程序运行中,可以改变变量的归属类型,

# scala基本语法
## 定义变量

两中方式,显示的不常用,隐式的,不是没有指定类型,而是在变量值中自行判断

``` stylus
var str = "ss"
var sr:String = "ss"
```
## 声明变量的两种修饰符

Var:变量可以被重新赋值,用于必须可变的变量,例如while中的变量
  Val:(value常量)不可被重新赋值
  Def :定义函数,定义方法
   注意:在编程过程中,能使用val的地方不要使用var

## 基础数据类型9种(String)

Java基础数据类型:它对应的变量,不是对象,不能通过”.”运算符来访问对象的方法
Scala对应的java的举出数据类型,对应的变量可以通过”.”来调用对象的方法toDouble方法转换类型
Double类型可以用科学计数法表示 var a = 1415e2 会变为double141500.0

## String类型的字面量
使用转义字符或者是三个引号原样输出,$中可以定义运算,可以输出变量,相当于java中的EL表达式

![enter description here][8]

## 基础数据类型转换

对象.to类型   虽然是调用方法,但是不加括号,并且是生成一个新对象

## 标识符

7.标识符 符合java的规范
类标识符 ,驼峰命名首字母大写
变量 方法标识符 ,驼峰命名,首字母小写
包 标识符,全小写,层级使用点分割
Val在scala中虽然定义常量,但是,一般都用变量的规则来命名标识符
注释规则和java一样

语句块:java语句块全部都是过程,没有返回值,只有方法语句块中用return才能有返回值
           Scala中大部分语句块都是有返回值的,而且不需要return{}
   Java中语句块的作用:主要用来划分作用域,scala中的语句块除了划分作用域之外还可以带返回值
  var str1 =”11”
 Var str2 ={
   Val str3 =s”${str1}dededede”
Str3
}
Println(str3)  访问不到
Scala中语句快中最后一句,就是返回值(for,while循环没有返回值)

## if...else在java中是没有返回值的,scala中是有返回值的,可以直接用变量接收

``` stylus
 val score = args(0).toInt
    //类java的用法
    if(score>90){
      println("优秀")
    }else if(score>80){
      println("良好")
    }else if(score>60){
      println("中等")
    }else{
      println("yiban")
    }
    //scala用法
    val result = if (score>90){
      "优秀"
    }else if (score>80){
      "良好"
    }else if(score>60){
      "中等"
    }else{
      "查"
    }
    println(result)
```
## while循环

没有返回值,在scala中也没有返回值,一般用来构建死循环,do()while先执行再判断

``` stylus
 //[]作为泛型符号,所以用()来获取元素
    var times = args(0).toInt
    while(times>0){
      println(s"第${times}次打印")
      times = times - 1
    }

    //do循环  ++ -- 运算符是无效的在scala
    var times2 = args(0).toInt
    do{
      println(s"第${times2}次打印")
      times2 = times2 - 1
    }while(times2>0)

    //while一般用来构建死循环
    while(true){

    }
```
## for循环

在scala中,本身scala是没有返回值的,只是为了表达一种执行过程,可以用yield让它有返回值,在语句块前加yield,另外可以用添加守卫的方式来增加限制条件

``` stylus
var times = args(0).toInt
    for(i<-1 to times){
      println(s"${i}打印")

    }
    //循环守卫添加
        for (i<-1 to times if  i%2==0){
          println(s"${i}打印")
        }
    for (i<-1 to times if i%2 ==0&& i<4) println(s"${i}打印")

    //for多层循环(嵌套循环)
    //for (i<-1 to times)

      //scala的嵌套循环
    for(i <- 1 to times ;j<-1 to times){
      println(s"${i},${j}")
    }
   //for嵌套循环加限定条件
    for(i <- 1 to times ;j<-1 to times if i%2==1&& j%2==0){
      println(s"${i},${j}")
    }
    //i 长,j 宽 面积大于25打印
    for(i <- 1 to times ;j<-1 to times if i>j && i*j>20){
      println(s"${i},${j}")
    }
    //对于条件复杂的for循环,可以把小括号写成大括号
    for {
      i<-1 to times
      j<-1 to times
      x = i*j
      if x>10
    }{
      println(s"two${i},${j}")
    }

    //打印乘法口诀,不要使用var
    for{
      i<-1 to times
      j<-1 to i
    }{
      if(i ==j){print(s"${i}*${j}=${i*j} \n")}
      else{
        print(s"${i}*${j}=${i*j} ")
      }
    }
   //一行代码
    for (i<-1 to 9;j<-1 to i)print(s"$j*$i=${i*j} ${if(i==j)"\n"else ""}")

    //把1-10之间的偶数以一个集合对象返回
    //for循环有返回值的能力
    //yield 后面的语句块的返回值就是for yield的返回值
    //yield后面一定有返回值,即便表达式没有返回值,它会以unit对象来作为返回值 即(),括号代表没有返回值
   val result = for(i<-1 to times)yield {
      if(i%2==0) i
    }
    println(result)


    val result2 = for(i<-1 to times if (i%2 ==0))yield i
    println(result2)

    //每个集合对象+5 返回
    val result3 = for(i<-1 to times)yield i+5
    println(result3)
```
## unit类

java中没有返回值的方法的类型是void,scala中没有返回值,使用的是unit,它对应的是无值

## break跳出循环(return 跳出所有循环)

``` stylus
val loop = new Breaks;
    //在它的方法中写for循环即可break
    loop.breakable(
      for (i <- 1 to times) {
        if (i == 5) loop.break() else println(i)
      }
    )
```
终止循环:在scala中没有break和continue,但是可以使用scala中提供的特殊类型Breaks来实现break    也可以使用return终止整个函数的方式终止循环

## 函数的声明和字面量

### val futest:(Int,Int)=>Int =(x,y)=>x+y

>把函数定义成变量的形式,可以作为一个参数或者对象,传到下一个方法或是函数中,这种定义方法是先定义函数类型,再定义函数的字面量   即 def 函数名:函数类型=字面量

>其中**函数类型是:输入=>输出**   **字面量:参数=>函数体**    函数类型的定义相当于变量的定义  变量名:类型(不同与java)

### val funcationTest = (x:Int,y:Int)=>x+y

>定义成变量的简写形式,把字面量和函数类型放在一块定义,所以变量名=(类型+量)=>函数体

>**val定义的不同是=号的位置**

### def fuctiontest1(x:Int,y:Int)={x+y}

>定义成函数,不是变量,此方法只能调用,即传参数获得计算结果,不能引用此对象(函数就是一个对象),类似java的方法的定义 只是多了=号,因为在scala中,每个函数都是对象,所以需要给对象初始化值

### def functiontest2(x:Int,y:Int){x+y}

>这样定义的是一个过程函数,没有返回值,或者可以说返回值是Unit类型,在函数体中写return也不会返回想要的数值,一般不怎么用

## 函数被当做参数传到其他函数

>参数位置写:参数名:类型         若是传函数   即参数名(随便写):Int=>Int(类型,输入类型=>输出类型)  如果输入类型只有一个,那么可以省略(),若有多个必须有()  函数可以有多个输入,但只有一个输出类型

``` stylus
def caculateArea(x:Int,f:(Int)=>Double) ={
    f(x)
  }
```
## 函数类型的参数的传参

- 如果定义了函数,那么直接把函数传递过来即可
- 如果没有定义,那么就直接定义,直接定义字面量就可以了

``` stylus
  //对外提供万能的计算公式,传递参数一个函数,函数可以作为一个参数传递过来
    caculateArea(4,x=>{
      println(s"等边三角形面颊:${x*x*scala.math.sqrt(3)/4}")
      x*x*scala.math.sqrt(3)/4
    })
```
## array的声明和字面量

array是元素可变,长度不可变的,一旦定义,长度不变

>定义方式有两种,一种是直接赋值,一种是指定array的类型和长度,以及array的三种遍历数组的方式,filter的过滤会留下返回值为true的值,取出返回为false的值,exites方法,只要存在一个就返回true

``` stylus
//数组的声明和字面量写法
    val array = Array(1,2,3,4)
	//数组的非字面量声明
    val array1 = new Array[String](5)
	 println(array1.mkString(","))
	 
	   //数组的遍历 until表示娶不到,to 可以取到
    for(i<-0 until array.length){
      println(array(i))
    }
    for (i<-array){
      println(i)
    }

    array.foreach(x=>{
      println(x)
    })
	//冒泡排序 顺序

   def bubbleSort(array:Array[Int])={
      for (i<-0 until array.length-1;j<-0 until array.length-i-1){
        if(array(j)>array(j+1)){
          val tmp = array(j)
          array(j)=array(j+1)
          array(j+1)=tmp
        }
      }
    }

    //选择排序算法,顺序排序
    def selectSort(array:Array[Int]) ={
      for (i<-0 until array.length-1;j<-1+i until array.length){
        if(array(i)>array(j)){
          val tmp = array(i)
          array(i) = array(j)
          array(j) = tmp
        }
      }
      array
    }
	//过滤元素字符串中包含a的所有元素
    val fruits = Array("sfaa","jieug","jfha","gfjs")
    val noAFruits = fruits.filter(x=>(!x.contains("a")))
    println(noAFruits.mkString(","))

    //判断fruits中是否包含长度为5 的水果
    val is5Length = fruits.exists(x=>x.length==5)
    println(is5Length)
```
## arraybuffer

>ArrayBuffer是不定长的数组,在定义的时候可以不指定长度,并且长度也可以在定义后随着元素的变化而随意增长和减小

``` stylus
 //ArrayBuffer 的声明和字面量
    val ab1 = new ArrayBuffer[Int](2)
    val ab2 = new ArrayBuffer[String]()
    val ab3 = ArrayBuffer(1,2,3,4)
	 //新增数据到arraybuffer中,+= 或者-= 才是在原来的arraybuffer上进行的操作,不带=号会生成新的
   //根据equals方法来判断是不是含有-的值
    ab2 += "sss"
    ab2 += "sssgg"
	//+:数组  会加在数组前边一个字符串
	println("cc" +:ab2)
	//插入数据到指定位置
    //如果写入的起始位置超出arraybuffer中的现有index值的话会报错
    ab2.insert(0,"fsdf","qq")
    ab2.insert(2,"fsdf","qq")
    println(ab2)
	
	 //删除某个位置 或者某个位置删除几个,不指定长度删除一个
    ab2.remove(2,2)
    ab2.remove(2)
    println(ab2)
	
	//drop方法也是删除arraybuffer中的数据,不过会返回新数组,不是修改元数组
    print(ab2.drop(1)) //从左往右删除
    print(ab2.dropRight(1)) //从右往左删除
    println(ab2)
	
	//更新某位置出的数据,ji 修改,原来的会替换成现在的
    ab2.update(0,"wwwww")
    println(ab2)
	
	//返回一个新的arraybuffer ,只留下偶数
    val ab4 = ArrayBuffer(1,2,3,4,5,6,7,8,9)
    val result1 = ab4.filter(x=>x%2==0)
    println(result1)
	
	//定义一个函数,接收一个arraybuffer,和一个函数,定义的函数对每一个arraybufer中的值进行
    //计算,返回true 则保留元素,否则删除元素
    def removeBy(ab:ArrayBuffer[Int],f:Int=>Boolean)={
      //对ab遍历
      for(item <-ab){
        if(!f(item))ab -= item
      }
    }
```
## List

>list是不可变的元素列表,元素不可变,长度也不可改变,如果添加元素或删除元素,会返回新的list

### list的声明,字面量,添加元素,与取值

``` stylus
//list的声明和字面量
    val list = List(1,2,3,4)
    val list1 = List()
    println(list)
    println(list1)
    println(list(0))
    //list元素对象不可更改,不能重新赋值
   //list(0)=100

    //添加元素,形成新的list,都是返回新的list对象
    println(list.::(22))
    println(33::list)
    println(33::44::list)
    //:+向后追加
    println(list.:+(99))
    //+; 向前追加
    println(88+:list)
    println(77+:88+:list)
    //两个list合并
    println(list:::list)
    println(list++list++list)
```
### list 常用的函数

``` stylus
//count函数的应用
    val list2 = List(1,2,3,4,5,6,7,8,11,55,88,55,66,77,2,445,55,5,4,45,8,5,52,85,5,5)
    val bt5Sum = list2.count(x=>x>5)
    println(bt5Sum)

    //判断list2是否以7,8 结尾
    val isEnd78 = list2.endsWith(List(6,7))
    println(s"list${if(isEnd78)"是"else "不是"}")

    //find: 返回list中第一个满足查询条件的元素,并用option封装
    //找出list中第一个大于4并且是偶数的  会返回一个some类型的值,如果没有返回none
    val result = list2.find(x=>x>4&&x%2==0)
    println(result)

    val list3 = List("hadoop is good","spark is better" , "sql is best")
    //把list3中的每一个元素各自拆分成另一个单词一个元素list
    //flatmap 是一个战平操作,类似hive中的explore 接受一个函数,函数输入是一个元素,输出是一个集合
    val flatList = list3.flatMap(x => x.split("\\s"))
    println(flatList)

    //reduce fold  aggreate用来做聚合  科利华(分组)颗粒化
    val resultR = list2.reduce((x1,x2)=>x1+x2)
    println(resultR)

    //reduce 可以做聚合,可以求最大值最小值,底层是迭代计算,可以是加和,也可以是单纯返回自己本身的元素
    val maxR= list2.reduce((x1,x2)=>if(x1>x2)x1 else x2)
    println(s"最大值为$maxR")

    //fold[A1 >: A](z: A1)(op: (A1, A1) ⇒ A1): A1
    //z  最好不要影响聚合计算,求和是0求积是1,字符串拼接是""  op是一个函数满足自己的需求
    val sumFold = list2.fold(0)((x1,x2)=>x1+x2)
    println(sumFold)

    //把list2中的元素聚合成字符串,字符串中包含每一个元素
    val addStr = list2.foldLeft("")((c,x)=>s"$c${if(c=="")"" else ","}$x")
    println(addStr)

    //(B)(f1,f2) 一个初始值,两个函数,一个定义规则,一个是聚合(分区规则不清楚)
    val strAggrate = list2.aggregate("")(
      (c,x)=>s"$c${if(c=="")"" else ","}$x",  //seqop 每个分区的聚合操作
      (c1,c2)=>s"$c1,$c2"   //每个区已经聚合好的,在次聚合
    )
println(strAggrate)

    val sumAggregate = list2.aggregate(0)(
      (c,x)=>c+x,
      (c1,c2)=>c2+c1
    )
    println(sumAggregate)

    //nil 代表一个空的list
    val list4 = "a"::"b"::"v"::Nil
    println(list4)

    //获取左边第一个元素,迭代操作会用到
    println(list4.head)
    //获取除去第一个元素
    println(list4.tail)
    //获取右边第一个
    println(list4.last)
    //获取除去右边第一个
    println(list4.init)

    //把list按照奇偶分组
    val groups = list2.groupBy(x=>if(x%2==0)"偶数" else "奇数")
    println(groups)

    //zip拉链操作,把两个list组合起来,当元素数量不一致时,会按照元素小的为准,少的元素没有了就不匹配了
    println(list2.zip(list4))

    //map 方法把list或是map 通过某种规则构建一个新的collection
    //把list的每一个元素转换成字符传返回一个新的list,没个元素都是string类型
    val list5 = list2.map(x=>x.toString)
    println(list5)
```
### java List 的定义

``` stylus
//定义一个java  list对象
    val list6 = new util.ArrayList[String]()
    list6.add("122")
    list6.add("123")
    println(list6)
```
### listBuffer 是可变长的

``` stylus
//listBuffer 声明,字面量,取值,改值
    //listBuffer 是可变长的类型
    val mList = ListBuffer(1,2,3,4)
    mList(0)=100
    println(mList)
    mList.insert(0,11)
    println(mList)
    mList.update(0,100)
    println(mList)
    mList.remove(3)
    println(mList)
```
## def val lazy 定义变量的区别
- def: def修饰符修饰的变量,声明赋值时不会马上执行,在每次调用变量的时候会重新计算
- val:  在赋值声明完成后,就会进行计算并完成赋值,不管声明 的变量有木有使用,并且不会重复计算
- lazy: lazy val 的形式,跟val类似,不会重复计算,不过是在调用变量的时候才会完成计算并赋值的过程

``` stylus
 //val 类型的变量,在声明的时候就会把右边表达式的结果计算出来,然后赋值给result
    //一旦赋值,右边的表达式就不再计算
    val result = sumInt(4,5)
    println("定义完成")
    println(result)
    println(s"第二次$result")
    //def 类型的变量,在声明赋值的时候,右边的表达式不会马上执行
    //def 变量每一次被调用的时候,等号右边的表达式都会被重新计算一次
    def d = sumInt(2,3)
    println(s"变量已经定义了")
    println(s"第一次打印$result")
    println(s"第二次打印$result")
    //lazy定义的变量,在声明赋值时,等号右边的表达式不会马上执行
    //在lazy对象第一次调用的时候会被计算一次,并赋值给lazy对象,后续再次调用,表达式不在计算
    lazy val l = sumInt(1,9)
    println("定义完成")
    println(s"第一个$l")
    println(s"第二次$l")
```
## Set  无序,不重复
>set是长度可变,并且set是无序,不重复的,在定义的时候如果有重复的元素,会自动筛选掉,无序指的是定义顺序和存储顺序是不固定的,一旦存储顺序也就固定了,他不能根据索引来取值
>**set名字是重复的,有可变的set,也有不可变的set,默认不指定包的话,是不可变的set,如果想用可变的呃set,需要指定是mutable包写的set,他们的声明方式一样,并且大部分方法也是一样的,个别有不同**

``` stylus
//set的声明和字面量,无序,不重复
    val set1 = Set(1,2,3,4,1,5,2)
    println(set1)
    //返回false,因此set无序不重复,不能用位置来获取元素
    println(set1(0)) //会返回false
    for(i<-set1)print(i)
    println("---")
    set1.foreach(x=>{print(x)})

    //head tail
    //init last
    //可变的set
    val mSet = scala.collection.mutable.Set(1,2,3,8,9,5)
    mSet.add(4554)
    println(mSet)
    mSet += 66
    println(mSet)
    //remove 根据对象来删除,不能根据位置
    mSet.remove(1)
    println(mSet)
```
## map 相当于只有两个元素的元组
### 默认是不可变的map
#### map的声明 字面量 与取值

``` stylus
 //map 声明,字面量 ,取值
    val map1 = Map(1->"a",2->"b",3->"c")
    println(map1)
    val map2 = Map((1,"a"),(2,"b"))
    println(map2)
    println(map1(2))
    val map3 = Map("a"->1,"b"->2)
    println(map3)

    //map遍历
    map1.foreach(
      x=>println(s"key:${x._1},value:${x._2}")
    )
    println("----")
    for(x<-map2){
      println(s"key:${x._1},value:${x._2}")
    }
    println("----")
    for((k,v)<-map3){
      println(s"key:${k},vlaue:${v}")
    }
    println("----")
    for(ks<-map1.keySet){
      println(s"key:${ks},vlaue:${map1(ks)}")
    }

    //不可变map  不可重新给元素赋值
    //map1("q")=55

    //get 方法,获取指定key的value值
    val map4 = Map("zyz"->18,"wxx"->99,"fff"->66)
    println(map4.get("zyz"))
    println(map4("zyz"))
    //用()来获取元素,若没有key,那么会抛异常
    //println(map4("zyzh"))
    //get方法健壮性更强
    println(map4.get("hzyz"))

    //map ++
    println(map1 ++ map2)
    println(map1 ++ Map("gg"->88))
```
#### 不可变map的常用方法

##### count(有条件的计算个数或这数量)

``` stylus
//count
    val map5 = Map("apple"->5,"pear"->4,"tomato"->6,"banana"->9,"att"->6)
    //计算出value道标key长读的map
    val count = map5.count(x=>x._1.length==x._2)
    println(count)
    //计算key的长度为5的个数
    val count1 = map5.count(x=>x._1.length==5)
```
##### filter 过滤操作把原集合根据条件过滤掉一部分

``` stylus
//filter 过滤(集合)
    val fil1 = map5.filter(x=>x._1.length>5)
    println(fil1)
    println(map5.filter(x=>x._1.contains("t")&&x._2==6))
    println(map5.filterKeys(x=>x.length<6))
```
##### flatMap  展平操作,只能一级展平(split->list)(value->map)

``` stylus
//flatMap 展平map
    val map6 = Map("a"->List(1,2,3,5),"b"->List(7,8,9))
    val flatMap1 = map6.flatMap(x=>x._2)
    println(flatMap1)
    val map7 = Map("a"->List("1"->2,"2"->List(11,22),"zy","ii5"),"b"->List("k7","lo","9p"))
    println(map7.flatMap(x=>x._2))
	
	 val list3 = List("hadoop is good","spark is better" , "sql is best")
    //把list3中的每一个元素各自拆分成另一个单词一个元素list
    //flatmap 是一个战平操作,类似hive中的explore 接受一个函数,函数输入是一个元素,输出是一个集合
    val flatList = list3.flatMap(x => x.split("\\s"))
    println(flatList)
```
##### groupBy 根据函数的返回值结果进行分组(true-false)(给定的任意返回值) 相同的合成一个map

``` stylus
//groupBy  map的嵌套
    println(map5.groupBy(x=>x._2))
    val group1 = map5.groupBy(x=>x._1.startsWith("a"))
    println(group1)
    //按照姓氏分组   按照返回的值进行分组,上边按照false和true分组
    val group2 = map5.groupBy(x=>x._1.split(" ")(0))
```

##### map 对原collection进行加工,返回新的目标collection(list,map取决于自行定义的function)

``` stylus
//map 方法,对kv对  进行映射转换,key 和value 必须都的返回,都是返回新的map
    val map8 = Map("校长"->3000,"小智"->700,"PDD"->1400)
    //每个人工资加500
    println(map8.map(x=>(x._1,x._2+500)))
    //若只对value处理可以使用mapvalues
    println(map8.mapValues(x=>x+1000))
    //工资大于1000 是高收入,为她们呢的姓名打上标签
    val result = map8.map(x=>(if(x._2>1000)s"[高收入]${x._1},${x._2}"else s"[低收入]${x._1},${x._2}"))
    //val result = map8.map(x=>((if(x._2>1000)s"[高收入]${x._1}"else s"[低收入]${x._1}"),x._2))
    //val reuslt = map8.map(x=>((s"${if(x._2>10000)"[高收入]" else "[低收入]"}${x._1}"),x._2))
```
![list][9]

![map][10]

##### 聚合 reduce,fold,foldLeft,Aggregate

``` stylus
//reduce 聚合
    //把map8中所有的value加起来
    val salaryMonth = map8.reduce((x1,x2)=>{
      ("月指出",x1._2+x2._2)
    })
    println(salaryMonth)
    //fold
    val salaryFold = map8.fold(("月指出",0))((c,x)=>(c._1,c._2+x._2))
    val salaryFoldLeft = map8.foldLeft(0)((c,x)=>c+x._2)
    val salaryAggregate = map8.aggregate(0)(
      (c,x)=>c+x._2,
      (c1,c2)=>c1+c2
    )
    println(salaryFold)
    println(salaryFoldLeft)
    println(salaryAggregate)
```
##### 最大值最小值等

``` stylus
//key 最大最小值
    println(s"map1的最大值:${map1.max}")
    println(s"map1的最小值:${map1.min}")
    //根据工资返回最大值最小值,指定根据key还是value
    println(s"map1的最大值:${map8.maxBy(x=>x._2)}")
    println(s"map1的最小值:${map8.minBy(x=>x._2)}")

    //先将kv变换位置在进行最大值判断
    val maxSalary = map8.map(x=>(x._2,x._1)).max
    println(maxSalary)
	
	//判断map4中,是否包含key:"zang",判断的仅仅是key
    println(map4.contains("zyz"))
	
	//drop删除  参数是删除的个数,并不是索引,不是指定位置删除,而是指定个数删除,不可变返回的是新map
    println(map5)
    println(map5.drop(2))
    println(map5.dropRight(2))
```
### 可变map的声明,字面量

``` stylus
/可变map的声明定义
    val mmap1 =scala.collection.mutable.Map(1->"a",2->"c")
    println(mmap1)
    println(mmap1(1))
    println(mmap1.get(1))

    //put(key,value)
    mmap1.put(3,"v")
    //指定key来删除
    mmap1.remove(3)
    //+= 一个或多个 逗号隔开
    mmap1 += (4->"p",5->"tt")
    mmap1 += ((4,"p"),7->"dd")
    mmap1.update(2,"update")
    println(mmap1)
```

## 元组Tuple ,元素的类型可以不一致,并且属于不可变的,没在collection包下,不属于集合,不能foreach遍历去元素,只能通过 下划线索引来取值
>功能就是可以在函数方法中使用元组,返回多个结果,再通过下划线取值

``` stylus
//元组的声明,字面量,取值
    val tuple = (1,2,"a",3.0,123.000,true)
    val tuple1 = (1,"王思聪","男",3.0,123.000,true)
    val pairTuple = ("a",1) //等同于map,map就是只有两个值的元祖
    println(tuple)
    println(tuple1)
    println(pairTuple)

    //取值
    println(tuple._6)
    //元组也是不可变的,定义之后不能发生改变
    //tuple._6=false

    //元组可以进行多个变量定义和赋值(变量和元组的元素个数要一致,否则报错)
    val list = List(1,2,3)
   // val (one,two,three)= (list(0),list(1),list(2))
    val tutu = (1,"d",true)
    val (one,two,three)= tutu
    println(one)
    println(two)
    println(three)

    //集合类型都有泛型,都是集合,可以foreach循环
    /*for(i<-tuple){
      println(i)
    }*/

    //变量定义,计算,封装类型(可以让方法返回多个值)
    val tupleResult = tupleTest1("p666")
    println(tupleResult)
  }
  def tupleTest1(a:String)={
    val value1 = s"return value1${a}"
    val value2 = s"return value2$a"
    val value3 = s"return value3$a"
    val value4 = s"return value4$a"
    (value1,value2,value3,value4)
  }
```
## Option类 是用来封装其他类型的对象(相当于容器Option[Int]),一般应用于方法的返回值上,以避免方法返回值为空带来的问题   它的子类可以有(None==代表没有返回值)(Some==封装了返回值)

``` stylus
def main(args: Array[String]): Unit = {
    //option some none
    val value1 = optionT(4)
    val value2 = optionT(-5)
    //返回some(4)
    println(value1)
    //返回none
    println(value2)
    //得到4
    println(value1.get)
    //如果为none 还是用get方法就会抛异常,.NoSuchElementException
    println(value2.get)

    //用途,用来初始化一个值,he match 一块使用  判断是否为null
    val value3 = optionT(-5)
    //先判断value3是否为空,若为空返回0,若不为空,返回自身的值
    println(value3.getOrElse(0))
    //some 可以封装任意类型,一般用于函数返回值上
    val some1 = Some("fff")
    val some2 = Some(4)
    println(some1)
    println(some2)
  }
  def optionT(x:Int):Option[Int]={
    if(x>0) Some(x) else  None
  }
```




  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577589086.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577762910.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577834117.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578912186.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578605315.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578725733.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510581290857.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510582295531.jpg
  [9]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510758621325.jpg
  [10]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510758726019.jpg