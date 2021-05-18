---
title: ANTLR安装和初次尝试
date: 2021-04-26 10:56:33
tags: 
- JAVA
- 语法分析
- ANTLR
---

> 本文是antlr4权威指南的阅读笔记。
>
> 书籍链接：链接: https://pan.baidu.com/s/1OLBC46BL8V_3qZQm7bgzdw 提取码: ckdf

# 1. 初识ANTLR

## 1.1 安装ANTLR

**第一步：下载jar包**

> 该书籍使用4.0版本，但是当前最新版已经到了4.9.2版本了，本文仍然采用4.0版本。

jar包包含了ANTLR工具、运行库、树形结构生成库、SrtingTemplate。

- **ANTLR工具**：将语法文件转化成可以识别该语法文件所描述的语言的程序。例如：给定一个识别JSON的语法，ANTLR工具会根据该语法生成程序。

- **运行库**：利用ANTLR工具生成的程序通过ANTLR运行库来识别输入的JSON。

- **树形结构生成库**：生成树形结构。

- **StringTemplate**：用于生成源代码、网页、电子邮件或其他格式化的输出文本。

**UNIX系统**

0. 安装JAVA7

   **注意：ANTLR4.0目前只支持JAVA7，如果使用高版本的JAVA，会报异常`java.lang.IndexOutOfBoundsException`。**

1. 下载

```bash
$ cd /usr/local/lib
$ curl -O https://www.antlr.org/download/antlr-4.0-complete.jar
```

2. 添加`antlr-4.9-complete.jar`到`CLASSPATH`：

```java
$ export CLASSPATH=".:/usr/local/lib/antlr-4.0-complete.jar:$CLASSPATH"
```

将其放入`.bash_profile`中（用户根目录）。

3. 为ANTLR工具创建别名，然后为`TestRig`。

```bash
$ alias antlr4='java -Xmx500M -cp "/usr/local/lib/antlr-4.0-complete.jar:$CLASSPATH" org.antlr.v4.Tool'
$ alias grun='java -Xmx500M -cp "/usr/local/lib/antlr-4.0-complete.jar:$CLASSPATH" org.antlr.v4.gui.TestRig'
```

**Windows系统**

0. 安装JAVA

1. 从https://www.antlr.org/download/下载antlr-4.0-complete.jar（或任何版本），然后 保存到第三方Java库的目录中`C:\Javalib`

2. 添加`antlr-4.0-complete.jar`到CLASSPATH，可以：
   
   - 永久：高级系统设置>环境变量>创建或附加到`CLASSPATH`变量
   
3. 使用批处理文件或doskey命令为ANTLR工具和TestRig创建简短的便捷命令：

   - 使用doskey命令：

   ```bash
   doskey antlr4=java org.antlr.v4.Tool $*
   doskey grun =java org.antlr.v4.runtime.misc.TestRig  $*
   ```

   每次打开命令行都需要重新输入这段命令，可以通过修改注册表来使得每次打开自动输入这两条命令。

   具体操作参照：https://www.zhihu.com/question/51962577/answer/128317488

## 1.2 测试&初次尝试

将以下语法放入文件Hello.g4中：Hello.g4

```java
// Define a grammar called Hello
grammar Hello;
r  : 'hello' ID ;         // match keyword hello followed by an identifier
ID : [a-z]+ ;             // match lower-case identifiers
WS : [ \t\r\n]+ -> skip ; // skip spaces, tabs, newlines
```

![image-20210429165441964](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210429165441964.png)

在其上运行ANTLR工具：

```bash
$ cd /tmp
$ antlr4 Hello.g4
$ javac Hello*.java
```

执行`antlr4 Hello.g4`后，会自动生成.java文件以及.tokens文件。

![image-20210429165536160](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210429165536160.png)



执行`javac Hello*.java`会把所有的.java文件编译成.class文件。

![image-20210429165657920](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210429165657920.png)

测试：

```bash
grun Hello r -tokens
=>hello parrt
=>(Ctrl+Z)
#输出：
[@0,0:4='hello',<1>,1:0]
[@1,6:10='parrt',<2>,1:6]
[@2,13:12='<EOF>',<-1>,2:0]
```

每行代表一个词法符号，表明了全部信息，例如`[@1,6:10='parrt',<2>,1:6]`表示，这个词法符号位于第二个位置，由输入文本的第6-10个位置之间的字符组成。

```bash
grun Hello r -gui
=>hello parrt
=>(Ctrl+Z)
```

利用gui可以展示语法分析树：

![image-20210426173304589](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210426173304589.png)