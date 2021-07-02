---
title: 三个案例展示ANTLR的不同特性
date: 2021-05-13 11:30:45
tags:
- ANTLR
- JAVA
---

# 三个案例展示ANTLR的不同特性

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

**注意：ANTLR4.0目前只支持JAVA7，如果使用高版本的JAVA，会报异常。**

该文章所有源代码点击 https://media.pragprog.com/titles/tpantlr2/code/tpantlr2-code.zip下载

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

![image-20210511204440048.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210511204440048.png)



最终，我们需要把ANTLR为我们自动生成的语法分析器集成到程序中。

```java
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.*;
import java.io.FileInputStream;
import java.io.InputStream;

public class ExprJoyRide {
    public static void main(String[] args) throws Exception {
        String inputFile =null;
        if(args.length>0) inputFile = args[0];
        InputStream is = System.in;
        if(inputFile!=Null) is = new FileInputStream(inputFile);

        // 新建一个CharStream，从标准输入读取数据
        ANTLRInputStream input = new ANTLRInputStream(is);
		
        // 新建一个词法分析器，处理输入的CharStream
        ExprLexer lexer = new ArrayInitLexer(input);

        // 新建一个词法符号的缓冲区，用于存储词法分析器生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器，处理词法符号缓冲区的内容
        ExprParser parser = new ArrayInitParser(tokens);

        // 针对init规则，开始进行语法分析
        ParseTree tree = parser.prog();

        // 用LISP风格打印出生成的树
        System.out.println(tree.toStringTree(parser));
    }
}
```

编译执行

```bash
$ javac ExprJoyRide.java Expr*.java
$ java ExprJoyRide t.expr
```

### 3.1 语法导入

语法拆分成两个逻辑单元，分别是语法分析器和词法分析器的语法。

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

例如，可以把这个拆分成两个。

```apl
## CommonLexerRules

lexer grammar CommonLexerRules; // 注意是lexer grammar

ID  :   [a-zA-Z]+ ;      // match identifiers <label id="code.tour.expr.3"/>
INT :   [0-9]+ ;         // match integers
NEWLINE:'\r'? '\n' ;     // return newlines to parser (is end-statement signal)
WS  :   [ \t]+ -> skip ; // toss out whitespace
```
```apl
## LibExpr.g4

grammar LibExpr;
import CommonLexerRules;

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
```

利用import导入词法规则。

**在执行anltr时不需要对被导入的语法执行antlr。**

### 3.2 错误输入

ANTLR能自动报告语法错误并从错误中恢复。

![image-20210512204014029.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210512204014029.png)

语法分析器遇到语法错误能够正确向下分析。

![image-20210512204216966.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210512204216966.png)

![image-20210512204235718.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210512204235718.png)

## 4. 利用访问器构建计算器

语法文件：

```apl
grammar LabeledExpr; // rename to distinguish from Expr.g4

prog:   stat+ ;

stat:   expr NEWLINE                # printExpr
    |   ID '=' expr NEWLINE         # assign
    |   NEWLINE                     # blank
    ;

expr:   expr op=('*'|'/') expr      # MulDiv
    |   expr op=('+'|'-') expr      # AddSub
    |   INT                         # int
    |   ID                          # id
    |   '(' expr ')'                # parens
    ;

MUL :   '*' ; // assigns token name to '*' used above in grammar
DIV :   '/' ;
ADD :   '+' ;
SUB :   '-' ;
ID  :   [a-zA-Z]+ ;      // match identifiers
INT :   [0-9]+ ;         // match integers
NEWLINE:'\r'? '\n' ;     // return newlines to parser (is end-statement signal)
WS  :   [ \t]+ -> skip ; // toss out whitespace
```

计算器：

```java
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.ParseTree;

import java.io.FileInputStream;
import java.io.InputStream;

public class Calc {
    public static void main(String[] args) throws Exception {
        String inputFile = null;
        if ( args.length>0 ) inputFile = args[0];
        InputStream is = System.in;
        if ( inputFile!=null ) is = new FileInputStream(inputFile);
        ANTLRInputStream input = new ANTLRInputStream(is);
        LabeledExprLexer lexer = new LabeledExprLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        LabeledExprParser parser = new LabeledExprParser(tokens);
        ParseTree tree = parser.prog(); // parse

        EvalVisitor eval = new EvalVisitor();
        eval.visit(tree);
    }
}
```

开始：

```bash
$ antlr4 -no-listener -visitor LabeledExpr.g4
```

ANTLR自动生成了访问器接口，并且为每个带标签的备选分支生成了一个方法。

![image-20210512210734768.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210512210734768.png)

因为使用了泛型，所以我们的实现类可以自定义返回类型。

同时，ANTLR生成了该访问器的默认实现类`LabeledExprBaseVisitor`。因为我们的计算结果都是整数，所以我们的`EvalVistor`类应该继承`LabeledExprBaseVisitor<Integer>`类。

```java
package labelExpr;
 
import java.util.HashMap;
import java.util.Map;
 
public class MyEvalVisitor extends LabelExprBaseVisitor<Integer> {
 
    /**
     * 保存运行中的结果
     */
    Map<String, Integer> memory = new HashMap<>();
 
    @Override
    public Integer visitAssign(LabelExprParser.AssignContext ctx) {
        String id = ctx.ID().getText();
        int value = visit(ctx.expr());
        memory.put(id, value);
        return value;
    }
 
    @Override
    public Integer visitPrintExpr(LabelExprParser.PrintExprContext ctx) {
        Integer value = visit(ctx.expr());
        System.out.println(value);
        return 0;
    }
 
    @Override
    public Integer visitInt(LabelExprParser.IntContext ctx) {
        return Integer.valueOf(ctx.INT().getText());
    }
 
    @Override
    public Integer visitId(LabelExprParser.IdContext ctx) {
        String id = ctx.ID().getText();
        if (memory.containsKey(id)) {
            return memory.get(id);
        }
        return 0;
    }
 
    @Override
    public Integer visitMulDiv(LabelExprParser.MulDivContext ctx) {
        int left = visit(ctx.expr(0));
        int right = visit(ctx.expr(1));
        if (ctx.op.getType() == LabelExprParser.MUL) {
            return left * right;
        } else {
            return left / right;
        }
    }
 
    @Override
    public Integer visitAddSub(LabelExprParser.AddSubContext ctx) {
        int left = visit(ctx.expr(0));
        int right = visit(ctx.expr(1));
        if (ctx.op.getType() == LabelExprParser.Add) {
            return left + right;
        } else {
            return left - right;
        }
    }
 
    @Override
    public Integer visitParens(LabelExprParser.ParensContext ctx) {
        return visit(ctx.expr());
    }
}
```

```bash
$ antlr4 -no-listener -visitor LabeledExpr.g4
$ ls LabeledExpr*.java
$ javac Calc.java LabeledExpr*.java EvalVisitor.java
$ java Calc t.expr
```

![image-20210513100528473.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210513100528473.png)

如果有任何一步报错或者异常，请检查JAVA版本和ANTLR版本。

## 5. 利用监听器构建翻译程序

目标：解析一个类，用其中的方法签名生成接口，保留全部的空白字符和注释。

我们可以通过ANTLR的监听器来完成，监听JAVA语法分析树遍历器触发的事件。ANTLR会自动生成接口：包含遍历器进入和离开类定义时，遍历器遇到方法时的抽象方法。

**访问器和监听器的区别：**

访问器的方法中，必须显式调用visit方法来访问子节点。不调用visit方法，对应的子树就不会被访问。

监听器的方法会被ANTLR提供的遍历器对象自动调用。

我们不需要实现接口中的全部方法，ANTLR提供了一个默认的`JAVABaseListener`的实现，我们可以继承该类，然后覆写我们需要的方法。

```java
// ExtractInterfaceListener继承BaseListener类
import org.antlr.v4.runtime.TokenStream;
import org.antlr.v4.runtime.misc.Interval;

public class ExtractInterfaceListener extends JavaBaseListener {
    JavaParser parser;
    public ExtractInterfaceListener(JavaParser parser) {this.parser = parser;}
    /** Listen to matches of classDeclaration */
    @Override
    public void enterClassDeclaration(JavaParser.ClassDeclarationContext ctx){
        System.out.println("interface I"+ctx.Identifier()+" {");
    }
    @Override
    public void exitClassDeclaration(JavaParser.ClassDeclarationContext ctx) {
        System.out.println("}");
    }
    
    /** Listen to matches of methodDeclaration */
    @Override
    public void enterMethodDeclaration(
        JavaParser.MethodDeclarationContext ctx
    )
    {
        // need parser to get tokens
        TokenStream tokens = parser.getTokenStream();
        String type = "void";
        if ( ctx.type()!=null ) {
            type = tokens.getText(ctx.type());
        }
        String args = tokens.getText(ctx.formalParameters());
        System.out.println("\t"+type+" "+ctx.Identifier()+args+";");
    }
}
```

```java
// 提供main方法
import org.antlr.v4.runtime.ANTLRInputStream;
import org.antlr.v4.runtime.CommonTokenStream;
import org.antlr.v4.runtime.ParserRuleContext;
import org.antlr.v4.runtime.Token;
import org.antlr.v4.runtime.tree.*;

import java.io.FileInputStream;
import java.io.InputStream;

public class ExtractInterfaceTool {
    public static void main(String[] args) throws Exception {
        String inputFile = null;
        if ( args.length>0 ) inputFile = args[0];
        InputStream is = System.in;
        if ( inputFile!=null ) {
            is = new FileInputStream(inputFile);
        }
        ANTLRInputStream input = new ANTLRInputStream(is);

        JavaLexer lexer = new JavaLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        JavaParser parser = new JavaParser(tokens);
        ParseTree tree = parser.compilationUnit(); // parse

        ParseTreeWalker walker = new ParseTreeWalker(); // create standard walker
        ExtractInterfaceListener extractor = new ExtractInterfaceListener(parser);
        walker.walk(extractor, tree); // initiate walk of tree with listener
    }
}
```

```bash
$ antlr4 Java.g4
$ javac Java*.java Extract*.java
$ java ExtractInterfaceTool Demo.java
```

原Demo.java

```java
import java.util.List;
import java.util.Map;
public class Demo {
	void f(int x, String y) { }
	int[ ] g(/*no args*/) { return null; }
	List<Map<String, Integer>>[] h() { return null; }
}
```

生成的结果：

```java
interface IDemo {
        void f(int x, String y);
        int[ ] g(/*no args*/);
        List<Map<String, Integer>>[] h();
}
```



