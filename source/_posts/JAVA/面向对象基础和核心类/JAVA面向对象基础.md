---
title: JAVA - 面向对象基础
date: 2021-04-2 19:44:25
tags: JAVA
---

# JAVA面向对象基础

感谢廖雪峰老师的教程！

[JAVA教程](https://www.liaoxuefeng.com/wiki/1252599548343744)

## 1. 方法class

`class`由`field`和`method`组成。

```java
class Person{
    private String name;
    private int age;
    
    public String getName(String name){
        this.name = name;
 
    }
}
```

`private`修饰的`field`无法直接外部访问，但是可以被`method`来访问以及修改。

在方法内部，可以使用一个隐含的变量`this`，它始终指向当前实例。因此，通过`this.field`就可以访问当前实例的字段。

可变参数用`类型...`定义，可变参数相当于数组类型：

```java
class Group {
    private String[] names;

    public void setNames(String... names) {
        this.names = names;
    }
}
```

```java
Group g = new Group();
g.setNames("Xiao Ming", "Xiao Hong", "Xiao Jun"); // 传入3个String
g.setNames("Xiao Ming", "Xiao Hong"); // 传入2个String
g.setNames("Xiao Ming"); // 传入1个String
g.setNames(); // 传入0个String
```

## 2. 构造方法

创建实例的时候，实际上是通过构造方法来初始化实例的。我们先来定义一个构造方法，能在创建`Person`实例的时候，一次性传入`name`和`age`，完成初始化：

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return this.name;
    }

    public int getAge() {
        return this.age;
    }
}

```
通过`Person p = new Person("Xiao Ming", 15);`使用构造函数完成初始化。

如果没有构造方法，编译器会生成默认的构造方法。

可以在一个类中定义多个构造方法，同时不同构造方法之间也可以相互调用。

## 3.方法重载（overload）

定义同名方法，但是参数不同，叫做方法重载。

例如：

```java
class Hello {
    public void hello() {
        System.out.println("Hello, world!");
    }

    public void hello(String name) {
        System.out.println("Hello, " + name + "!");
    }

    public void hello(String name, int age) {
        if (age < 18) {
            System.out.println("Hi, " + name + "!");
        } else {
            System.out.println("Hello, " + name + "!");
        }
    }
}
```

## 4. 继承

继承是面向对象编程中非常强大的一种机制，它首先可以复用代码。当我们让`Student`从`Person`继承时，`Student`就获得了`Person`的所有功能，我们只需要为`Student`编写新增的功能。

**extends**

Java使用`extends`关键字来实现继承：

```java
class Person {
    private String name;
    private int age;

    public String getName() {...}
    public void setName(String name) {...}
    public int getAge() {...}
    public void setAge(int age) {...}
}

class Student extends Person {
    // 不要重复name和age字段/方法,
    // 只需要定义新增score字段/方法:
    private int score;

    public int getScore() { … }
    public void setScore(int score) { … }
}
```

**protected**
继承有个特点，就是子类无法访问父类的`private`字段或者`private`方法。例如，`Student`类就无法访问`Person`类的`name`和`age`字段：

我们需要把`private`改为`protected`。用`protected`修饰的字段可以被子类访问

```java
class Person {
    protected String name;
    protected int age;
}

class Student extends Person {
    public String hello() {
        return "Hello, " + name; // OK!
    }
}
```
**super**
super关键字表示父类（超类）。子类引用父类的字段时，可以用`super.fieldName`。
这是因为在Java中，任何class的构造方法，第一行语句必须是调用父类的构造方法。如果没有明确地调用父类的构造方法，编译器会帮我们自动加一句super();，所以，Student类的构造方法实际上是这样：

```java
class Student extends Person {
    protected int score;

    public Student(String name, int age, int score) {
        super(name, age); // 调用父类的构造方法Person(String, int)
        this.score = score;
    }
}
```
**阻止继承（JAVA15）**

正常情况下，只要某个class没有`final`修饰符，那么任何类都可以从该class继承。

从Java 15开始，允许使用`sealed`修饰class，并通过`permits`明确写出能够从该class继承的子类名称。

例如

```java
public sealed class Shape permits Rect, Circle, Triangle {
    ...
}
```

表示`Shape`只能被`Rect`、`Circle`、`Triangle`继承。

**向下转型**

```java
Person p1 = new Student(); // upcasting, ok
Person p2 = new Person();
Student s1 = (Student) p1; // ok
Student s2 = (Student) p2; // runtime error! ClassCastException!
```

为了避免Error，可以使用`instanceof`操作符来判断某个实例究竟是不是某种类。

```java
Person p = new Person();
System.out.println(p instanceof Person); // true
System.out.println(p instanceof Student); // false

Student s = new Student();
System.out.println(s instanceof Person); // true
System.out.println(s instanceof Student); // true

Student n = null;
System.out.println(n instanceof Student); // false
```

在**java14**后，判断后可以直接转型成指定变量。

```java
public class Main {
    public static void main(String[] args) {
        Object obj = "hello";
        if (obj instanceof String s) {
            // 可以直接使用变量s:
            System.out.println(s.toUpperCase());
        }
    }
}
```

这样更简洁。

## 5. 多态

在继承关系中，子类如果定义了一个与父类方法签名完全相同的方法，被称为**覆写（Override）**。

```java
class Person {
    public void run() {
        System.out.println("Person.run");
    }
}
```

```java
class Student extends Person {
    @Override
    public void run() {
        System.out.println("Student.run");
    }
}
```

> 注意：方法名相同，方法参数相同，但方法返回值不同，也是不同的方法。在Java程序中，出现这种情况，编译器会报错。

**多态**是指，针对某个类型的方法调用，其真正执行的方法取决于运行时期实际类型的方法。例如：

```java
public class Main {
    public static void main(String[] args) {
        // 给一个有普通收入、工资收入和享受国务院特殊津贴的小伙伴算税:
        Income[] incomes = new Income[] {
            new Income(3000),
            new Salary(7500),
            new StateCouncilSpecialAllowance(15000)
        };
        System.out.println(totalTax(incomes));
    }

    public static double totalTax(Income... incomes) {
        double total = 0;
        for (Income income: incomes) {
            total = total + income.getTax();
        }
        return total;
    }
}

class Income {
    protected double income;

    public Income(double income) {
        this.income = income;
    }
    
    public double getTax() {
        return income * 0.1; // 税率10%
    }
}

class Salary extends Income {
    public Salary(double income) {
        super(income);
    }

    @Override
    public double getTax() {
        if (income <= 5000) {
            return 0;
        }
        return (income - 5000) * 0.2;
    }
}

class StateCouncilSpecialAllowance extends Income {
    public StateCouncilSpecialAllowance(double income) {
        super(income);
    }

    @Override
    public double getTax() {
        return 0;
    }
}
```

具体实例调用时选择的方法是实例自己的`getTax()`

**覆写Object方法**

因为所有的`class`最终都继承自`Object`，而`Object`定义了几个重要的方法：

- `toString()`：把instance输出为`String`；
- `equals()`：判断两个instance是否逻辑相等；
- `hashCode()`：计算一个instance的哈希值。

在必要的情况下，我们可以覆写`Object`的这几个方法。

**调用super**

在子类的覆写方法中，如果要调用父类的被覆写的方法，可以通过`super`来调用。

```java
class Person {
    protected String name;
    public String hello() {
        return "Hello, " + name;
    }
}

Student extends Person {
    @Override
    public String hello() {
        // 调用父类的hello()方法:
        return super.hello() + "!";
    }
}
```

**final**

继承可以允许子类覆写父类的方法。如果一个父类**不允许子类对它的某个方法进行覆写**，可以把该方法标记为`final`。用`final`修饰的方法不能被`Override`

如果一个类**不希望任何其他类继承**自它，那么可以把这个类本身标记为`final`。用`final`修饰的类不能被继承：

对于一个类的**实例字段**，同样可以用`final`修饰。用`final`修饰的字段在**初始化后不能被修改**。

## 6.抽象类

如果父类的方法本身不需要实现任何功能，仅仅是为了定义方法签名，目的是让子类去覆写它，那么，可以把父类的方法声明为抽象方法：

```java
abstract class Person {
    public abstract void run();
}
```

- **类和方法都必须标注出`abstract`**

- **表明抽象方法和抽象类后，该类无法实例化**。

- **抽象类的子类必须重写抽象方法**

- **引用类时尽量引用高层类型，避免使用实际子类型。**

## 7. 接口

如果一个抽象类没有字段，所有方法全部都是抽象方法，就可以把该抽象类改写为接口：`interface`。

```java
interface Person {
    void run();
    String getName();
}
```

因为接口定义的所有方法默认都是`public abstract`的，所以这两个修饰符不需要写出来。

当一个具体的`class`去实现一个`interface`时，需要使用`implements`关键字。

```java
class Student implements Person {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(this.name + " run");
    }

    @Override
    public String getName() {
        return this.name;
    }
}
```

一个类可以实现多个`interface`

```java
class Student implements Person, Hello { // 实现了两个interface
    ...
}
```

|            | abstract class      | interface                   |
| :--------: | ------------------- | --------------------------- |
|    继承    | 只能extend一个class | 可以implements多个interface |
|    字段    | 可以定义实例字段    | 不能定义实例字段            |
|  抽象方法  | 可以定义抽象方法    | 可以定义抽象方法            |
| 非抽象方法 | 可以定义非抽象方法  | 可以定义`default`方法       |

 **`default`方法:**

 ```java
 interface Person {
     String getName();
     default void run() {
         System.out.println(getName() + " run");
     }
 }
 ```

 实现类可以不用重写default方法。

**继承关系：**

![1425423424754.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/1425423424754.png)

## 8.静态字段和静态方法

用`static`修饰的字段，称为静态字段：`static field`。

静态字段通常使用 `类名.静态字段`访问静态对象。

![5567512578452.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/5567512578452.png)

所有实例和类共享同一个静态字段。

静态方法也是相同的。用`static`修饰的方法称为静态方法。通过类名可以调用静态方法。

静态方法经常用于工具类，例如`Arrays.sort()`,`Math.random()`

`interface`  是抽象类，不能定义实例字段，但是可以定义静态字段。

```java
public interface Person {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
}
```

`public static final`可以不用写，默认为`public static final`。

## 9. 包

Java定义了一种名字空间，称之为包：`package`。一个类总是属于某个包，类名（比如`Person`）只是一个简写，真正的完整类名是`包名.类名`。

```java
package ming; // 申明包名ming

public class Person {
}
```

包名不同，类就不同。例如`ming.Person`和`hong.Person`是两个不同的类。

包可以是多层结构，用`.`隔开。例如：`java.util`。

包没有父子关系。`java.util`和`java.util.zip`是不同的包，两者没有任何继承关系。

还需要按照包结构把上面的Java文件组织起来。

![86765436434898.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/86765436434898.png)

位于同一个包的类，可以访问包作用域的字段和方法。

**import**

在一个`class`中，我们总会引用其他的`class`。

有**三种写法**：

1. 直接写出完整类名。`mr.jun.Arrays arrays = new mr.jun.Arrays();`

2. 使用import语句。`import mr.jun.Arrays;Arrays arrays = new Arrays();`同时可以直接*将包下面的所有`class`都导入进来。
3. `import static`导入一个类的静态字段和静态方法。

```java
package main;

// 导入System类的所有静态字段和静态方法:
import static java.lang.System.*;

public class Main {
    public static void main(String[] args) {
        // 相当于调用System.out.println(…)
        out.println("Hello, world!");
    }
}
```

**查找顺序：**

- 如果是完整类名，就直接根据完整类名查找这个`class`；
- 如果是简单类名，按下面的顺序依次查找：
  - 查找当前`package`是否存在这个`class`；
  - 查找`import`的包是否包含这个`class`；
  - 查找`java.lang`包是否包含这个`class`。

**最佳实践**

最好使用倒置的域名来确保唯一性。例如

- `org.apache`
- `org.apache.commons.log`
- `com.liaoxuefeng.sample`

## 10. 作用域

**private**

定义为`public`的`class`、`interface`可以被其他任何类访问：

```java
package abc;

public class Hello {
    public void hi() {
    }
}
```

```java
package xyz;

class Main {
    void foo() {
        // Main可以访问Hello
        Hello h = new Hello();
    }
}
```

定义为`public`的`field`、`method`可以被其他类访问，前提是首先有访问`class`的权限：

```java
package abc;

public class Hello {
    public void hi() {
    }
}
```

```java
package xyz;

class Main {
    void foo() {
        Hello h = new Hello();
        h.hi();
    }
}
```

**private**

定义为`private`的`field`、`method`无法被其他类访问：

如果一个类内部还定义了嵌套类，那么，嵌套类拥有访问`private`的权限

```java
package abc;

public class Hello {
    // 不能被其他类调用:
    private void hi() {
    }

    public void hello() {
        this.hi();
    }
}
```

**protected**

`protected`作用于继承关系。定义为`protected`的字段和方法可以被子类访问，以及子类的子类：

```java
package abc;

public class Hello {
    // protected方法:
    protected void hi() {
    }
}
```

```java
package xyz;

class Main extends Hello {
    void foo() {
        // 可以访问protected方法:
        hi();
    }
}
```

**package**

包作用域是指一个类允许访问同一个`package`的没有`public`、`private`修饰的`class`，以及没有`public`、`protected`、`private`修饰的字段和方法。

```java
package abc;
// package权限的类:
class Hello {
    // package权限的方法:
    void hi() {
    }
}
```

```java
package abc;

class Main {
    void foo() {
        // 可以访问package权限的类:
        Hello h = new Hello();
        // 可以调用package权限的方法:
        h.hi();
    }
}
```

**局部变量**

在方法内部定义的变量称为局部变量，局部变量作用域从变量声明处开始到对应的块结束。方法参数也是局部变量。

**final**

用`final`修饰`class`可以阻止被继承。

用`final`修饰`method`可以阻止被子类覆写。

用`final`修饰`field`可以阻止被重新赋值。

用`final`修饰局部变量可以阻止被重新赋值。

**总结**

如果不确定是否需要`public`，就不声明为`public`, 尽可能少暴露对外的字段和方法。

把方法定义为`package`权限有助于测试，因为测试类和被测试类只要位于同一个`package`，测试代码就可以访问被测试类的`package`权限方法。

一个`.java`文件只能包含一个`public`类，但可以包含多个非`public`类。如果有`public`类，文件名必须和`public`类的名字相同。

**`pubilc > package > protected > private`**

## 11. 内部类

有一种类，它被定义在另一个类的内部，所以称为**内部类（Nested Class）**。

```java
class Outer {
    class Inner {
        // 定义了一个Inner Class
    }
}
```

**`Inner`**是一个Inner Class，它与普通类有个最大的不同，就是Inner Class的实例不能单独存在，必须依附于一个Outer Class的实例。

先实例化外部类，再调用外部类的new来创建内部类的实例。

```java
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested"); // 实例化一个Outer
        Outer.Inner inner = outer.new Inner(); // 实例化一个Inner
        inner.hello();
    }
}

class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }
    
    class Inner {
        void hello() {
            System.out.println("Hello, " + Outer.this.name);
        }
    }
}
```



**匿名类Anonymous Class**



```java
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested");
        outer.asyncHello();
    }
}

class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }
    
    void asyncHello() {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello, " + Outer.this.name);
            }
        };
        new Thread(r).start();
    }
}
```

在方法内部实例化了一个`Runnable`。`Runnable`本身是接口，接口是不能实例化的，所以这里实际上是定义了一个实现了`Runnable`接口的匿名类，并且通过`new`实例化该匿名类，然后转型为`Runnable`。在定义匿名类的时候就必须实例化它，定义匿名类的写法如下：

`Runnable`和`Thread`是多线程编程部分的内容。

```java
Runnable r = new Runnable() {
    // 实现必要的抽象方法...
};
```



**静态内部类**

静态内部类和Inner Class类似，但是使用`static`修饰。

```java
public class Main {
    public static void main(String[] args) {
        Outer.StaticNested sn = new Outer.StaticNested();
        sn.hello();
    }
}

class Outer {
    private static String NAME = "OUTER";

    private String name;
    
    Outer(String name) {
        this.name = name;
    }
    
    static class StaticNested {
        void hello() {
            System.out.println("Hello, " + Outer.NAME);
        }
    }

}
```

用`static`修饰的内部类和Inner Class有很大的不同，它不再依附于`Outer`的实例，而是一个完全独立的类，因此无法引用`Outer.this`，但它可以访问`Outer`的`private`静态字段和静态方法。如果把`StaticNested`移到`Outer`之外，就失去了访问`private`的权限。

## 12. `classpath`和`jar`

JVM通过环境变量`classpath`决定搜索`class`的路径和顺序；

```java
java -cp .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

当JVM在加载`abc.xyz.Hello`这个类时，会依次查找：

- `<当前目录>\abc\xyz\Hello.class`
- `C:\work\project1\bin\abc\xyz\Hello.class`
- `C:\shared\abc\xyz\Hello.class`

不推荐设置系统环境变量`classpath`，始终建议通过`-cp`命令传入；

**jar包**相当于目录，可以包含很多`.class`文件，方便下载和使用；

```java
java -cp ./hello.jar abc.xyz.Hello
```

这样JVM会自动在`hello.jar`文件里去搜索某个类。

jar包的格式如图所示：

![89656879456456456456.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/89656879456456456456.png)

`MANIFEST.MF`文件可以提供jar包的信息，如`Main-Class`，这样可以直接运行jar包。`MANIFEST.MF`是纯文本，可以指定`Main-Class`和其它信息。JVM会自动读取这个`MANIFEST.MF`文件，如果存在`Main-Class`，我们就不必在命令行指定启动的类名，而是用更方便的命令：

```
java -jar hello.jar
```

在大型项目中，不可能手动编写`MANIFEST.MF`文件，再手动创建zip包。Java社区提供了大量的开源构建工具，例如[Maven](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945359327200)，可以非常方便地创建jar包。

## 13. 模块

从Java 9开始，原有的Java标准库已经由一个单一巨大的`rt.jar`分拆成了几十个模块，这些模块以`.jmod`扩展名标识，可以在`$JAVA_HOME/jmods`目录下找到它们：

- `java.base.jmod`
- `ava.compiler.jmod`
- `java.datatransfer.jmod`
- `java.desktop.jmod`
- ...

这些`.jmod`文件每一个都是一个模块，模块名就是文件名。例如：模块`java.base`对应的文件就是`java.base.jmod`。模块之间的依赖关系已经被写入到模块内的`module-info.class`文件了。所有的模块都依赖`java.base`模块，只有`java.base`模块不依赖任何模块，它可以被看作是“根模块”，好比所有的类都是从`Object`直接或间接继承而来。

把class封装成包不仅要打包，而且要写入依赖关系。

![678678123489796.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/678678123489796.png)

包的机构与Java项目类似，module-info.java如下所示：

```java
module hello.world {
	requires java.base; // 可不写，任何模块都会自动引入java.base
	requires java.xml;
}
```

`module`是关键字，后面的`hello.world`是模块的名称，它的命名规范与包一致。花括号的`requires xxx;`表示这个模块需要引用的其他模块名。除了`java.base`可以被自动引入外，这里我们引入了一个`java.xml`的模块。

当我们使用模块声明了依赖关系后，才能使用引入的模块。

```java
package com.itranswarp.sample;

// 必须引入java.xml模块后才能使用其中的类:
import javax.xml.XMLConstants;

public class Main {
	public static void main(String[] args) {
		Greeting g = new Greeting();
		System.out.println(g.hello(XMLConstants.XML_NS_PREFIX));
	}
}
```

对`oop-module`项目进行编译

```bash
$ javac -d bin src/module-info.java src/com/itranswarp/sample/*.java
```

![978645678789789.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/978645678789789.png)

下一步将bin目录下的class打包成jar。传入`--main-class`参数，让这个jar包能自己定位`main`方法所在的类：

```bash
$ jar --create --file hello.jar --main-class com.itranswarp.sample.Main -C bin .
```

继续使用JDK自带的`jmod`命令把一个jar包转换成模块：

```bash
$ jmod create --class-path hello.jar hello.jmod
```

**运行模块**

```bash
$ java --module-path hello.jar --module hello.world
Hello, xml!
```

**打包JRE**

```bash
$ jlink --module-path hello.jmod --add-modules java.base,java.xml,hello.world --output jre/
```

```bash
$ jre/bin/java --module hello.world
Hello, xml!
```
