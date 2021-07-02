---
title: JAVA - 异常处理
date: 2021-04-10 17:06:25
tags: JAVA
---

# JAVA异常处理

感谢廖雪峰老师的教程！

[JAVA教程](https://www.liaoxuefeng.com/wiki/1252599548343744)

## 1. JAVA的异常

Java使用异常来表示错误，并通过`try ... catch`捕获异常；

Java的异常是`class`，并且从`Throwable`继承；

`Error`是无需捕获的严重错误，`Exception`是应该捕获的可处理的错误；

`RuntimeException`无需强制捕获，非`RuntimeException`（Checked Exception）需强制捕获，或者用`throws`声明；

**结构**如下：

```java
try {
    String s = processFile(“C:\\test.txt”);
    // ok:
} catch (FileNotFoundException e) {
    // file not found:
} catch (SecurityException e) {
    // no read permission:
} catch (IOException e) {
    // io error:
} catch (Exception e) {
    // other error:
}
```

异常的继承关系：

![image-20210325183327585.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210325183327585.png)

`Throwable`是异常体系的根，它继承自`Object`。

`Throwable`有两个体系：`Error`和`Exception`，`Error`表示严重的错误，程序对此一般无能为力，例如：

- `OutOfMemoryError`：内存耗尽
- `NoClassDefFoundError`：无法加载某个Class
- `StackOverflowError`：栈溢出

而`Exception`则是运行时的错误，它可以被捕获并处理。

某些异常是应用程序逻辑处理的一部分，应该捕获并处理。例如：

- `NumberFormatException`：数值类型的格式错误
- `FileNotFoundException`：未找到文件
- `SocketException`：读取网络失败

还有一些异常是程序逻辑编写不对造成的，应该修复程序本身。例如：

- `NullPointerException`：对某个`null`的对象调用方法或字段
- `IndexOutOfBoundsException`：数组索引越界

`Exception`又分为两大类：

1. `RuntimeException`以及它的子类；
2. 非`RuntimeException`（包括`IOException`、`ReflectiveOperationException`等等）

**强制捕获异常**

```java
public byte[] getBytes(String charsetName) throws UnsupportedEncodingException {
    ...
}
```

定义方法时，使用`throws Xxx`表示该方法可能抛出的异常类型。调用方在调用的时候，必须强制捕获这些异常，否则编译器会报错。

```java
static byte[] toGBK(String s) {
    try {
        // 用指定编码转换String为byte[]:
        return s.getBytes("GBK");
    } catch (UnsupportedEncodingException e) {
        // 如果系统不支持GBK编码，会捕获到UnsupportedEncodingException:
        System.out.println(e); // 打印异常信息
        return s.getBytes(); // 尝试使用用默认编码
    }
}
```
我们也可以不捕获它，而是在方法定义处用throws表示`toGBK()`方法可能会抛出`UnsupportedEncodingException`。

```java
static byte[] toGBK(String s) throws UnsupportedEncodingException {
    return s.getBytes("GBK");
}
```
此时调用`toGBK`的方法，必须捕获`UnsupportedEncodingException`异常，否则就会出错。

如下所示：

```java
public class Main {
    public static void main(String[] args) {
        try {
            byte[] bs = toGBK("中文");
            System.out.println(Arrays.toString(bs));
        } catch (UnsupportedEncodingException e) {
            System.out.println(e);
        }
    }

    static byte[] toGBK(String s) throws UnsupportedEncodingException {
        // 用指定编码转换String为byte[]:
        return s.getBytes("GBK");
    }
}
```

如果是测试代码，上面的写法就略显麻烦。如果不想写任何`try`代码，可以直接把`main()`方法定义为`throws Exception`：

`public static void main(String[] args) throws Exception`

捕获到异常后即使什么都做不了，也要把异常记录下来：

```java
static byte[] toGBK(String s) {
    try {
        return s.getBytes("GBK");
    } catch (UnsupportedEncodingException e) {
        // 先记下来再说:
        e.printStackTrace();
    }
    return null;
```

**`printStackTrace()`**

所有异常都可以调用`printStackTrace()`方法打印异常栈，这是一个简单有用的快速打印异常的方法。

## 2. 捕获异常

多个`catch`只会命中一个，因此应当把子类的`catch`写在前面。

**`finally`**

Java的`try ... catch`机制还提供了`finally`语句，`finally`语句块保证有无错误都会执行。

注意`finally`有几个特点：

1. `finally`语句不是必须的，可写可不写；
2. `finally`总是最后执行。

```java
public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (UnsupportedEncodingException e) {
        System.out.println("Bad encoding");
    } catch (IOException e) {
        System.out.println("IO error");
    } finally {
        System.out.println("END");
    }
}
```

无论是否捕获到异常都会输出“END”。

```java
catch (IOException | NumberFormatException e) { // IOException或NumberFormatException
        System.out.println("Bad input");
```

也可以把多个异常合并到一个`catch`内部。

捕获到异常尽量都使用`printStackTrace()`把调用路径输出出来。

## 3. 抛出异常

抛出异常分两步：

1. 创建某个`Exception`的实例；
2. 用`throw`语句抛出。

```java
void process2(String s) {
    if (s==null) {
        throw new NullPointerException();
    }
}
```

有时候在`catch`中会抛出新的异常，例如：

```java
static void process1() {
   try {
       process2();
   } catch (NullPointerException e) {
       throw new IllegalArgumentException(e);   // 将catch捕获的异常作为新异常的参数。
   }
}
```

为了能追踪到完整的异常栈，在构造异常的时候，把原始的`Exception`实例传进去，新的`Exception`就可以持有原始`Exception`信息。

```java
java.lang.IllegalArgumentException: java.lang.NullPointerException
    at Main.process1(Main.java:15)
    at Main.main(Main.java:5)
Caused by: java.lang.NullPointerException
    at Main.process2(Main.java:20)
    at Main.process1(Main.java:13)
```

能同时打印出两个异常类型。

**finally**中不要抛出异常。

## 4. 自定义异常

自定义一个`BaseException`作为“根异常”，然后，派生出各种业务类型的异常。

`BaseException`一般从`RuntimeException`派生：

```java
public class BaseException extends RuntimeException {
    public BaseException() {
        super();
    }

    public BaseException(String message, Throwable cause) {
        super(message, cause);
    }

    public BaseException(String message) {
        super(message);
    }

    public BaseException(Throwable cause) {
        super(cause);
    }
}
```

其他业务类型的异常就可以从`BaseException`派生：

```java
public class UserNotFoundException extends BaseException {
}

public class LoginFailedException extends BaseException {
}
```

## 5. NullPointerException

`NullPointerException`即空指针异常，俗称NPE。如果一个对象为`null`，调用其方法或访问其字段就会产生`NullPointerException`

好的编码习惯可以极大地降低`NullPointerException`的产生，例如：

成员变量在定义时初始化：

```java
public class Person {
    private String name = "";
}
```

从Java 14开始，如果产生了`NullPointerException`，JVM可以给出详细的信息告诉我们`null`对象到底是谁。

## 6. 断言

在Java中，使用`assert`关键字来实现断言。

```java
public static void main(String[] args) {
    double x = Math.abs(-123.45);
    assert x >= 0;
    System.out.println(x);
}
```

语句`assert x >= 0;`即为断言，断言条件`x >= 0`预期为`true`。如果计算结果为`false`，则断言失败，抛出`AssertionError`。

使用`assert`语句时，还可以添加一个可选的断言消息：

```java
assert x >= 0 : "x must >= 0";
```

要执行`assert`语句，必须给Java虚拟机传递`-enableassertions`（可简写为`-ea`）参数启用断言。

实际开发中，很少使用断言。

## 7. JDK Logging

在编写程序的过程中，发现程序运行结果与预期不符，经常用`System.out.println()`打印出执行过程中的某些变量，观察每一步的结果与代码逻辑是否符合，然后有针对性地修改代码。

Java标准库内置了日志包`java.util.logging`，这个包比`System.out.println()`好用很多。

```java
public class Hello {
    public static void main(String[] args) {
        Logger logger = Logger.getGlobal();
        logger.info("start process...");
        logger.warning("memory is running out...");
        logger.fine("ignored.");
        logger.severe("process will be terminated...");
    }
}
```

输出如下：

```java
Mar 02, 2019 6:32:13 PM Hello main
INFO: start process...
Mar 02, 2019 6:32:13 PM Hello main
WARNING: memory is running out...
Mar 02, 2019 6:32:13 PM Hello main
SEVERE: process will be terminated...
```

它自动打印了时间、调用类、调用方法等很多有用的信息。

JDK的Logging定义了7个日志级别，从严重到普通：

- SEVERE
- WARNING
- INFO
- CONFIG
- FINE
- FINER
- FINEST

因为默认级别是INFO，因此，INFO级别以下的日志，不会被打印出来。使用日志级别的好处在于，调整级别，就可以屏蔽掉很多调试相关的日志输出。

## 8. Commons Logging

`Commons Logging`是一个第三方日志库，它是由`Apache`创建的日志模块。

使用Commons Logging只需要和两个类打交道，并且只有两步：

第一步，通过`LogFactory`获取`Log`类的实例； 

第二步，使用`Log`实例的方法打日志。

```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
```

```java
public class Main {
    public static void main(String[] args) {
        Log log = LogFactory.getLog(Main.class);
        log.info("start...");
        log.warn("end.");
    }
}
```

报错：

`error: package org.apache.commons.logging does not exist`（找不到`org.apache.commons.logging`这个包）。

因为Commons Logging是一个第三方提供的库，所以，必须先把它[下载](https://commons.apache.org/proper/commons-logging/download_logging.cgi)下来。下载后，解压，找到`commons-logging-1.2.jar`这个文件，再把Java源码`Main.java`放到一个目录下。

编译：

```java
javac -cp commons-logging-1.2.jar Main.java
```

执行：

```java
java -cp .;commons-logging-1.2.jar Main
```

ommons Logging定义了6个日志级别：

- FATAL
- ERROR
- WARNING
- INFO
- DEBUG
- TRACE

默认级别是`INFO`。

使用Commons Logging时，如果在静态方法中引用`Log`，通常直接定义一个静态类型变量：

```java
// 在静态方法中引用Log:
public class Main {
    static final Log log = LogFactory.getLog(Main.class);

    static void foo() {
        log.info("foo");
    }
}
```

在实例方法中引用`Log`，通常定义一个实例变量：

```java
// 在实例方法中引用Log:
public class Person {
    protected final Log log = LogFactory.getLog(getClass());

    void foo() {
        log.info("foo");
    }
}
```

实例变量中使用`LogFactory.getLog(getClass())`，子类还可以使用该log实例。

## 9. Log4j

Log4j是一种非常流行的日志框架，最新版本是2.x。

使用时通过配置文件进行配置。

// 用到在学

## 10. SLF4J和Logback

SLF4J和Logback可以取代Commons Logging和Log4j；

始终使用SLF4J的接口写入日志，使用Logback只需要配置，不需要修改代码。

// 用到在学

