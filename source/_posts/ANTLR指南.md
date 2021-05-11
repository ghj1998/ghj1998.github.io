---
title: ANTLR指南
date: 2021-05-11 14:56:45
tags:
- ANTLR
- JAVA
---

# 四个案例展示ANTLR的不同特性

## 1. 简介

1. 分析一个简单的算术表达式语言的语法，
   - 深入学习语法分析器的启动过程。
   - 如何使用语法导入将语法文件切分为容易管理的小块。
   - 语法分析器处理非法输入。
2. 使用访问者模式构建计算器
   - 遍历算术表达式的语法分析树并获得计算结果
   - 自动生成访问器接口和空的实现类。
3. 构建一个翻译器
   - 读取JAVA类定义将其翻译成一个仅有方法声明的接口
   - ANTLR生成的监听器机制
4. 如何将代码嵌入语法文件
   - 实现极端的灵活性
5. ANTLR如何处理包含不止一种语言的输入文件
   - `TokenStreamRewriter`
   - 修改词法符号流不影响原来的输入流

## 2. 环境搭建

使用maven搭建环境，在配置文件`pom.xml`中增加依赖：

> maven的使用参考我的另一篇博客：[Maven](https://ghj1998.github.io/2021/05/07/Maven%E4%BD%BF%E7%94%A8/#more)

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.antlr/antlr4 -->
    <dependency>
        <groupId>org.antlr</groupId>
        <artifactId>antlr4</artifactId>
        <version>4.9.2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.antlr/antlr4-runtime -->
    <dependency>
        <groupId>org.antlr</groupId>
        <artifactId>antlr4-runtime</artifactId>
        <version>4.9.2</version>
    </dependency>
</dependencies>
```

## 3. 匹配算术表达式

要处理的表达式如下所示：

```java
193
a = 5
b = 6
a+b*2
(1+2)*3
```

解析该表达式的ANTLR语法：

```apl
grammar Expr;

/** The start rule; begin parsing here. */
prog:   stat+ ; 

stat:   expr NEWLINE                
    |   ID '=' expr NEWLINE        
    |   NEWLINE                   
    ;

expr:   expr ('*'|'/') expr   
    |   expr ('+'|'-') expr   
    |   INT                    
    |   ID                    
    |   '(' expr ')'         
    ;

ID  :   [a-zA-Z]+ ;      // match identifiers <label id="code.tour.expr.3"/>
INT :   [0-9]+ ;         // match integers
NEWLINE:'\r'? '\n' ;     // return newlines to parser (is end-statement signal)
WS  :   [ \t]+ -> skip ; // toss out whitespace
```

`stat` 和`expr`表示语法结构。`ID` `INT`是词法符号。

- 语法分析器的规则以小写字母开头
- 词法分析器的规则以大写字母开头
- 用| 分割同一个语言规则的若干备选分支，用圆括号把一些符号组合成子规则。例如（'*'|''/'）匹配一个乘法符号或者一个除法符号。

更多的内容在**留白** 进行讨论。

使用该语法分析器对上述的表达式进行分析。

```bash
$ antlr4 Expr.g4
$ javac Expr*.java
$ grun Expr prog -gui t.expr
```

*在执行第一部的时候抛出了`IndexOutOfBoundsException`*，是`antlr4.0`的问题，去官网下载最新版解决了这个问题。

注意修改时要同步修改`pom.xml` 中的版本号。

运行结果如下😁：

![image-20210511204440048](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511204440048.png)



最终，我们需要把ANTLR为我们自动生成的语法分析器集成到程序中。

