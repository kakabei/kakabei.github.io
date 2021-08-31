---
layout: post
title: java 类相关知识
date: 2021-07-15 16:12:15
categories: 学习笔记
tags: java
excerpt: java 类和对应相关的知识点
---

## 对象

对象的三个主要特性：

- 对象的行为（behavior）
- 对象的状态（state）
- 对象标识（identity）

类之间的关系:

依赖（“uses-a”） : 如果一个类的方法操纵另一个类的对象，我们就说一个类依赖于另一个类。应该尽可能地将相互依赖的类减至最少。

聚合（“has-a”）：聚合关系意味着类A的对象包含类B的对象。更加喜欢使用“关联”这个术语。

继承（“is-a”）：是一种用于表示特殊与一般关系的

### 对象与对象变量

要想使用对象，就必须首先构造对象，并指定其初始状态。然后，对对象应用方法。

使用构造器（constructor）构造新实例。构造器是一种特殊的方法，用来构造并初始化对象。 类似c++中的构造函数。

以Date类为例：

```java
Date birthday = new Date(); 
String s = new Date().toString();
```

对象与对象变量之间存在着一个重要的区别:

 ```java 
 Date deadline; 			 // deadllne dosen't refer to any object 
 s = deadline.toString(); // not yet
 deadline = birthday; 	 // refer to same object 
 ```

定义了一个对象变量deadline，它可以引用Date类型的对象。但是，一定要认识到：变量deadline不是一个对象，实际上也没有引用对象。此时，不能将任何Date方法应用于这个变量上。这一点和C++ 有区别，c++定义了对象变量，即是一个可以用的对象，只是它在栈中，不在堆中。

![](../..	/assets/java/2021-08-10-java-learn-2024333.png)

**一个对象变量并没有实际包含一个对象，而仅仅引用一个对象。**

很多人错误地认为Java对象变量与C++的引用类似。然而，在C++中没有空引用，并且引用不能被赋值。**可以将Java的对象变量看作C++的对象指针**。



### 更改器方法与访问器方法

调用一个方法后，只是把一个对象赋给另一个对象变量。原来的对象不做任何改动。这种叫访问器方法（accessor method）



调用一个方法手， 对象的数据被更改，叫更改器方法。（mutator method）

```java
LocalData aThousandDaysLater = newYearsEve.plusDays(1000);  // accessor method

sameDay.add(Calender.DAY_OF_MONTH,100); // mutator method
```

### 封装的优点

一个类的实例在获得或设置实例域的值，应该提供下面三项内容：

-  一个私有的数据域；
- 一个公有的域访问器方法；
- 一个公有的域更改器方法。

注意不要编写返回引用可变对象的访问器方法。 如： 

```java
class Employee 
{
    private Date hireDay; 
    
    public Date getHireDay()
    {
        return hireDay; // 没有错，但返回了一个可变对象，不是好习惯
    }
    // .... 
}
```

应该用clone 对原对象进行克隆。

```java
class Employee
{
    public Date getHireDay()
    {
       return (Date) hirDay.clone(); // ok　拷贝了对象 	
	}
}
```

一个类的对象可以访问这个类的其他对象，如：

```java
class Employee
{
    public boolean equals(Emplyee other)
    {
       return name.equals(other.name);  // 直接的访问了name. 但还是不建议这么用，要规范
	}
}
```

**final 实例域**。在实例构造后，就被设置了值，不能再被修改，即没有setXXX()方法了。

 final修饰符大都应用于基本（primitive）类型域，或不可变（immutable）类的域（如果类中的每个方法都不会改变其对象，这种类就是不可变的类。例如，String类就是一个不可变的类）。

对于可变的类，使用final修饰符对象，比较不好理解。例如：

```java
private final StringBuilder evaluations; 

evaluations = new StringBuilder(); 
```



final关键字只是表示存储在evaluations变量中的对象引用不会再指示其他StringBuilder对象。不过这个对象可以更改：

```java
public void giveGoldStar()
{
    vevaluations.append(LocalDate.now() +"Gold start!\n")
}
```

### 静态域与静态方法

类的成员变量可以分为两种：静态变量（或称为类变量），指被 static 修饰的成员变量；实例变量，指没有被 static 修饰的成员变量。

静态变量与实例变量的区别如下：

- 对于静态变量，运行时，Java 虚拟机只为静态变量分配一次内存，在加载类的过程中完成静态变量的内存分配。在类的内部，可以在任何方法内直接访问静态变量；在其他类中，可以通过类名访问该类中的静态变量。
- 对于实例变量，每创建一个实例，Java 虚拟机就会为实例变量分配一次内存。在类的内部，可以在非静态方法中直接访问实例变量；在本类的静态方法或其他类中则需要通过类的实例对象进行访问。

静态变量在类中的作用如下：

- 静态变量可以被类的所有实例共享，因此静态变量可以作为实例之间的共享数据，增加实例之间的交互性。
- 如果类的所有实例都包含一个相同的常量属性，则可以把这个属性定义为静态常量类型，从而节省内存空间。例如，在类中定义一个静态常量 PI。

与成员变量类似，成员方法也可以分为两种：静态方法（或称为类方法），指被 static 修饰的成员方法；实例方法，指没有被 static 修饰的成员方法。

静态方法与实例方法的区别如下：

- 静态方法不需要通过它所属的类的任何实例就可以被调用，因此在静态方法中不能使用 this 关键字，也不能直接访问所属类的实例变量和实例方法，但是可以直接访问所属类的静态变量和静态方法。另外，和 this 关键字一样，super 关键字也与类的特定实例相关，所以在静态方法中也不能使用 super 关键字。
- 在实例方法中可以直接访问所属类的静态变量、静态方法、实例变量和实例方法。

**静态代码块**

静态代码块指 Java 类中的 static{} 代码块，主要用于初始化类，为类的静态变量赋初始值。静态代码块的特点如下：

- 静态代码块类似于一个方法，但它不可以存在于任何方法体中。
- Java 虚拟机在加载类时会执行静态代码块，如果类中包含多个静态代码块，则 Java 虚拟机将按它们在类中出现的顺序依次执行它们，每个静态代码块只会被执行一次。
- 静态代码块与静态方法一样，不能直接访问类的实例变量和实例方法，而需要通过类的实例对象来访问。

### main方法

每一个类可以有一个main方法。这是一个常用于对类进行单元测试的技巧。例如，可以在Employee类中添加一个main方法：

```java
class Employee
{
   private static int nextId = 1;

   private String name;
   private double salary;
   private int id;

   public Employee(String n, double s)
   {
      name 	 = n;
      salary = s;
      id     = 0;
   }

   public String getName()
   {
      return name;
   }

   public double getSalary()
   {
      return salary;
   }

   public int getId()
   {
      return id;
   }

   public void setId()
   {
      id = nextId; // set id to next available id
      nextId++;
   }

   public static int getNextId()
   {
      return nextId; // returns static field
   }

   public static void main(String[] args) // unit test
   {
      Employee e = new Employee("Harry", 50000);
      System.out.println(e.getName() + " " + e.getSalary());
   }
}
```

### 方法参数

Java程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，特别是，方法不能修改传递给它的任何参数变量的内容。

法参数共有两种类型：

- 基本数据类型（数字、布尔值） 

- 对象引用

个方法不可能修改一个基本数据类型的参数。而对象引用作为参数就不同了。 

```java
public static void triplieSalary(Employee x) // work
{
    x.raiseSalary(200); 
}
```

实现一个改变对象参数状态的方法并不是一件难事。理由很简单，方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象。

![](../../assets/java/2021-08-10-java-learn-2024342.png)

总结一下Java中方法参数的使用情况：

● 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）。

● 一个方法可以改变一个对象参数的状态。

● 一个方法不能让对象参数引用一个新的对象。

书的一个例子，很好的说明static的特性：

```java
/**
 * This program demonstrates parameter passing in Java.
 * @version 1.00 2000-01-27
 * @author Cay Horstmann
 */
public class ParamTest
{
   public static void main(String[] args)
   {
      /*
       * Test 1: Methods can't modify numeric parameters
       */
      System.out.println("Testing tripleValue:");
      double percent = 10;
      System.out.println("Before: percent=" + percent);
      tripleValue(percent);
      System.out.println("After: percent=" + percent);

      /*
       * Test 2: Methods can change the state of object parameters
       */
      System.out.println("\nTesting tripleSalary:");
      Employee harry = new Employee("Harry", 50000);
      System.out.println("Before: salary=" + harry.getSalary());
      tripleSalary(harry);
      System.out.println("After: salary=" + harry.getSalary());

      /*
       * Test 3: Methods can't attach new objects to object parameters
       */
      System.out.println("\nTesting swap:");
      Employee a = new Employee("Alice", 70000);
      Employee b = new Employee("Bob", 60000);
      System.out.println("Before: a=" + a.getName());
      System.out.println("Before: b=" + b.getName());
      swap(a, b);
      System.out.println("After: a=" + a.getName());
      System.out.println("After: b=" + b.getName());
   }

   public static void tripleValue(double x) // doesn't work
   {
      x = 3 * x;
      System.out.println("End of method: x=" + x);
   }

   public static void tripleSalary(Employee x) // works
   {
      x.raiseSalary(200);
      System.out.println("End of method: salary=" + x.getSalary());
   }

   public static void swap(Employee x, Employee y)
   {
      Employee temp = x;
      x = y;
      y = temp;
      System.out.println("End of method: x=" + x.getName());
      System.out.println("End of method: y=" + y.getName());
   }
}

class Employee // simplified Employee class
{
   private String name;
   private double salary;
    
   public Employee(String n, double s)
   {
      name = n;
      salary = s;
   }

 	public Employee()
    {
        name = ""; 
        salary = 0; 
	}
}

```

### 初始化块

初始化数据域的方法有三种：

- 在构造器中设置值
- 在声明中赋值，即显式域初始化
- 初始化块

```java
lass Employee // simplified Employee class
{
   private String name = ""; // 显式域初始化
   private double salary;

   
   //   初始化块
   {
        in = nextId; 
        nextID++; 
   }
    
   public Employee(String n, double s) // 构造器 即构造函数
   {
      name = n;
      salary = s;
   }

   public String getName()
   {
      return name;
   }

   public double getSalary()
   {
      return salary;
   }

   public void raiseSalary(double byPercent)
   {
      double raise = salary * byPercent / 100;
      salary += raise;
   }
}

```

无论使用哪个构造器构造对象，id域都在对象初始化块中被初始化。首先运行初始化块，然后才运行构造器的主体部分。

