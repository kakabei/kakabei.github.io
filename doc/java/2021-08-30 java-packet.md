---
layout: post
title: java 包
date: 2021-07-15 16:12:15
categories: 学习笔记
tags: java
excerpt: java 
---

## 包
Java允许使用包（package）将类组织起来。借助于包可以方便地组织自己的代码，并将自己的代码与别人提供的代码库分开管理。



使用包的主要原因是确保类名的唯一性。

假如两个程序员不约而同地建立了Employee类。只要将这些类放置在不同的包中，就不会产生冲突。

事实上，为了保证包名的绝对唯一性，Sun公司建议将公司的因特网域名（这显然是独一无二的）以逆序的形式作为包名，并且对于不同的项目使用不同的子包。例如，horstmann.com是本书作者之一注册的域名。逆序形式为com.horstmann。这个包还可以被进一步地划分成子包，如com.horstmann. corejava。

## 类的导入 



一个类可以使所属包的中的所有类，以及其他包中公有类，有两种方法 ，一种是在每一个类名之前添加完整的包名：

```java 
java.time.LocatDate today = java.time.LocaDate.now(); 
```

另一种方法是使用 `import`。 可以导入一个特定的类或整个包。 `import`语句应该位于源文件的顶部（但位于package语句的后面）如： 

```java
import java.time.LocalDate;
import java.util.*; 
```

加星号的这种方式，可能导入冲突。 

```java
import java.util.*; 
import java.sql.*; 

Date today; // Error java.util.Date or java.sql.Date ? 
```

编译器无法确定使用哪一个。 解决的方法是： 在类前面加上完整的包名。

`import`语句的好处是简捷，不用去写出完整的包名。

## 静态导入 

`import`语句不仅可以导入类，还增加了导入静态方法和静态域的功能。如：

```java 
import static java.lang.System.*; 

out.println("Hello ShenZhen China.")  //system.out
```

`out`不用加类名前缀。

```java
sqrt(pow(x, 2) + pow(y,2)); // 没有Math 
Maht,sqrt(Math.pow(x,2) + Math,pow(y, 2)) ; // 
```

## 创建包

包声明应该在源文件的第一行，每个源文件只能有一个包声明，这个文件中的每个类型都应用于它。 通常使用小写的字母来命名避免与类、接口名字的冲突。

如： 

```java 
package animals; 

interface Animal {
    public void eat(); 
    public void travel(); 
}

```

将包中的文件放到与完整的包名匹配的子目录中。

如果一个源文件中没有使用包声明，那么其中的类，函数，枚举，注释等将被放在一个无名的包（unnamed package）中。



## 包作用域 

如果一个源文件中没有使用包声明，那么其中的类，函数，枚举，注释等将被放在一个无名的包（unnamed package）中。



## 类路径

类存储在文件系统的子目录中。类的路径必须与包名匹配。

类文件也可以存储在JAR(Java归档)文件中。在一个JAR文件中，可以包含多个压缩形式的类文件和子目录，这样既可以节省又可以改善性能。在程序中用到第三方（third-party）的库文件时，通常会给出一个或多个需要包含的JAR文件。

JAR文件使用ZIP格式组织文件和子目录。可以使用所有ZIP实用程序查看内部的rt.jar以及其他的JAR文件。

采用-classpath（或-cp）选项指定类路径：

```shell 
java -classpath /home/user/classdri:.:/home/user/archives/archive.jar Myprog
```

整个指令应该书写在一行中。将这样一个长的命令行放在一个shell脚本或一个批处理文件中。 

利用-classpath选项设置类路径是首选的方法，也可以通过设置CLASSPATH环境变量完成这个操作。

```shell 
export CLASSPATH=/home/user/classdir:.:/home/user/archives/archive.jar
```

直到退出shell为止，类路径设置均有效。

不建议CLASSPATH设置成环境变量。

## 文档注释

JDK包含一个很有用的工具，叫做javadoc，它可以由源文件生成一个HTML文档。

如果在源代码中添加以专用的定界符/**开始的注释，那么可以很容易地生成一个看上去具有专业水准的文档。

每个`/** . . . */`文档注释在标记之后紧跟着自由格式文本（free-form text）。标记由@开始，如`@autho`或`@param`。





## 类设计技巧

我们不会面面俱到，也不希望过于沉闷，所以这一章结束之前，简单地介绍几点技巧。应用这些技巧可以使得设计出来的类更具有OOP的专业水准。

1．一定要保证数据私有

2．一定要对数据初始化

3．不要在类中使用过多的基本类型

4．不是所有的域都需要独立的域访问器和域更改器

5．将职责过多的类进行分解

6．类名和方法名要能够体现它们的职责

7．优先使用不可变的类

## 关键字 final

使用final关键字的好处：

- final关键字提高了性能。JVM和Java应用都会缓存final变量。
- final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。
- 使用final关键字，JVM会对方法、变量及类进行优化。



**不可变类：**

创建不可变类要使用final关键字。不可变类是指它的对象一旦被创建了就不能被更改了。String是不可变类的代表。不可变类有很多好处，譬如它们的对象是只读的，可以在多线程环境下安全的共享，不用额外的同步开销等等。



**重要知识点：**

- final关键字可以用于成员变量、本地变量、方法以及类。
- final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。
-  你不能够对final变量再次赋值。
- 本地变量必须在声明时赋值。
- 在匿名类中所有变量都必须是final变量。
- final方法不能被重写。
- final类不能被继承。
- final关键字不同于finally关键字，后者用于异常处理。
- final关键字容易与finalize()方法搞混，后者是在Object类中定义的方法，是在垃圾回收之前被JVM调用的方法。
- 接口中声明的所有变量本身是final的。
- final和abstract这两个关键字是反相关的，final类就不可能是abstract的。
- final方法在编译阶段绑定，称为静态绑定(static binding)。
- 没有在声明时初始化final变量的称为空白final变量(blank final variable)，它们必须在构造器中初始化，或者调用this()初始化。不这么做的话，编译器会报错“final变量(变量名)需要进行初始化”。
- 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化。
- 按照Java代码惯例，final变量就是常量，而且通常常量名要大写。
  
  

