---
title: JAVA基础知识面试题（记录一些自己不会的知识点）
date: 2021-05-14 22:24:06
tags:
- JAVA
- 面试
---

### 1. **什么是Java程序的主类？应用程序和小程序的主类有何不同？**

一个程序中可以有多个类，但只能有一个类是主类。在Java应用程序中，这个主类是指包含main()方法的类。

而在Java小程序中，这个主类是一个继承自系统类JApplet或Applet的子类。应用程序的主类不一定要求是public类，但小程序的主类要求必须是public类。主类是Java程序执行的入口点。

### 2. static的作用
static的主要意义是在于创建独立于具体对象的域变量或者方法。以致于即使没有创建对象，也能使用属性和调用方法！
static关键字还有一个比较关键的作用就是用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。
static应用场景有：
1、修饰成员变量 2、修饰成员方法 3、静态代码块 4、修饰类【只能修饰内部类也就是静态内部类】 5、静态导包

### 3. &和&&的区别

&&运算符是短路与运算。是因为如果&&左边的表达式的值是 false，右边的表达式会被直接短路掉，不会进行运算。

&则左右的表达式都会运算。

### 4. final finally finalize区别
final可以修饰类、变量、方法，修饰类表示该类不能被继承、修饰方法表示该方法不能被重写、修饰变量表示该变量是一个常量不能被重新赋值。
finally一般作用在try-catch代码块中，在处理异常的时候，通常我们将一定要执行的代码方法finally代码块中，表示不管是否出现异常，该代码块都会执行，一般用来存放一些关闭资源的代码。
finalize是一个方法，属于Object类的一个方法，而Object类是所有类的父类，该方法一般由垃圾回收器来调用，当我们调用System.gc() 方法的时候，由垃圾回收器调用finalize()，回收垃圾，一个对象是否可回收的最后判断。

### 5. 在 Java 中，如何跳出当前的多重嵌套循环

在Java中，要想跳出多重循环，可以在外面的循环语句前定义一个标号，然后在里层循环体的代码中使用带有标号的break 语句，即可跳出外层循环。

```java
public static void main(String[] args) {
    ok:
    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            System.out.println("i=" + i + ",j=" + j);
            if (j == 5) {
                break ok;
            }
        }
    }
}
```

### 6. 面向对象五大基本原则是什么

单一职责原则SRP：类的功能要单一，不能包罗万象。

开放封闭原则OCP：一个模块对于拓展是开放的，对于修改是封闭的。

里式替换原则LSP：子类可以扩展父类的功能,但不能改变父类原有的功能。子类可以替换父类出现在父类能够出现的任何地方。

依赖倒置原则DIP：程序要依赖于抽象接口，不要依赖于具体实现。高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。

接口分离原则ISP：设计时采用多个与特定客户类有关的接口比采用一个通用的接口要好。

### 7. 存储位置

成员变量：随着对象的创建而存在，随着对象的消失而消失，存储在堆内存中。
局部变量：在方法被调用，或者语句被执行的时候存在，存储在栈内存中。当方法调用完，或者语句结束后，就自动释放。

### 8. 在Java中定义一个不做事且没有参数的构造方法的作用

Java程序在执行子类的构造方法之前，如果没有用super()来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用super()来调用父类中特定的构造方法，则编译时将发生错误，因为Java程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。

### 9.静态变量和实例变量区别

静态变量：静态变量由于不属于任何实例对象，属于类的，所以在内存中只会有一份，在类的加载过程中，JVM只为静态变量分配一次内存空间。

实例变量：每次创建对象，都会为每个对象分配成员变量内存空间，实例变量是属于实例对象的，在内存中，创建几次对象，就有几份成员变量。

### 10.内部类的分类

内部类可以分为四种：**成员内部类、局部内部类、匿名内部类和静态内部类**。

定义在类内部的静态类，就是静态内部类。

```java
public class Outer {
    private static int radius = 1;
    static class StaticInner {
        public void visit() {
            System.out.println("visit outer static variable:" + radius);
        }
    }
}
```

```java
Outer.StaticInner inner = new Outer.StaticInner();
inner.visit();
```

定义在类内部，成员位置上的非静态类，就是成员内部类。

```java
public class Outer {
    private static  int radius = 1;
    private int count =2;
     class Inner {
        public void visit() {
            System.out.println("visit outer static variable:" + radius);
            System.out.println("visit outer variable:" + count);
        }
    }
}
```

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
inner.visit();
```

定义在方法中的内部类，就是局部内部类。

```java
public class Outer {
    private  int out_a = 1;
    private static int STATIC_b = 2;
    public void testFunctionClass(){
        int inner_c =3;
        class Inner {
            private void fun(){
                System.out.println(out_a);
                System.out.println(STATIC_b);
                System.out.println(inner_c);
            }
        }
        Inner inner = new Inner();
        inner.fun();
    }
    public static void testStaticFunctionClass(){
        int d =3;
        class Inner {
            private void fun(){
                // System.out.println(out_a); 编译错误，定义在静态方法中的局部类不可以访问外部类的实例变量
                System.out.println(STATIC_b);
                System.out.println(d);
            }
        }
        Inner inner = new Inner();
        inner.fun();
    }
}
```

匿名内部类就是没有名字的内部类，日常开发中使用的比较多。

```java
public class Outer {
    private void test(final int i) {
        new Service() {
            public void method() {
                for (int j = 0; j < i; j++) {
                    System.out.println("匿名内部类" );
                }
            }
        }.method();
    }
 }
 //匿名内部类必须继承或实现一个已有的接口
 interface Service{
    void method();
}
```

匿名内部类还有以下特点：

- 匿名内部类必须继承一个抽象类或者实现一个接口。
- 匿名内部类不能定义任何静态成员和静态方法。
- 当所在的方法的形参需要被匿名内部类使用时，必须声明为 final。
- 匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的所有抽象方法。

### 11. 内部类的优点

- 一个内部类对象可以访问创建它的外部类对象的内容，包括私有数据！
- 内部类不为同一包的其他类所见，具有很好的封装性；
- 内部类有效实现了“多重继承”，优化 java 单继承的缺陷。
- 匿名内部类可以很方便的定义回调。

