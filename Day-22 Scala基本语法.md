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




  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577589086.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577762910.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510577834117.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578912186.jpg
  [5]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578605315.jpg
  [6]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510578725733.jpg
  [7]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510581290857.jpg
  [8]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1510582295531.jpg