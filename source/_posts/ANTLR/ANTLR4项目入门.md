---
title: ANTLR4项目入门
date: 2021-05-11 11:27:30
tags:
- ANTLR
- 语法
---

# 入门的ANTLR项目

# 1. 项目需求

**需求：**

- 识别包含在花括号内或者嵌套花括号的整数，例如{1，2，3}或{1，{2，3}，4}。这种括号常见于初始化语句中。
- 如果语句中的所有整数都可以用一个字节表示，将该整数数组转化为字节数组。
- 将short数组转化为字符串。

```java
static short[] data = {1,2,3};
```

转化为

```java
static String data = "\u000u\u0002\u0003";
```

**为什么提出这个需求：**

Java的.class文件在处理时，将初始化语句等同于`data[0] = 1; data[1] = 2`。相比之下，JAVA的class文件把字符串存储成连续的short序列。将数组的初始化转化为字符串而可以得到更紧凑的class文件。避免了JAVA对初始化方法的长度限制。

## 2. ANTLR工具、运行库、自动生成的代码。

### 2.1 ANTLR工具

ANTLR工具中的两个关键部分：

- ANTLR工具
- ANTLR运行库

当提到对一个语法运行ANTLR时，指的是利用`org.antlr.v4.Tool`类生成一些代码（语法分析器和词法分析器）。

运行库是由若干类和方法组成的库，这些类和方法是自动生成的代码（如Parser，Lexer和Token）运行所必需的。

故**使用ANTLR的步骤：**

- 对一个语法运行ANTLR，将生成的代码和jar包中的运行库一起编译。
- 将编译好的代码和运行库一起运行。

> **关于如何编写语法文件？**
>
> 链接：**暂时留白！**

### 2.2 开始编写

1. 语法文件`ArrayInit.g4`：

   ```yaml
   grammar ArrayInit;
   
   init : '{' value ( ',' value)* '}';
   
   value :init
       | INT 
       ;
   INT : [0-9]+ ;
   WS : [\t\r\n]+ -> skip;
   ```


2. 生成代码

   ```shell
   $ antlr4 ArrayInit.g4
   ```

   **注意**：直接`antlr4`命令要提前运行`antlr4=java org.antlr.v4.Tool $*`

   > 参考：https://ghj1998.github.io/2021/04/26/ANTLR4%E6%8C%87%E5%8D%97/

   生成文件如下所示：

   ![image-20210510153723801](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210510153723801.png)

   生成文件介绍

   - `ArrayInitParser.java`

     包含一个语法分析器的类的定义，用来识别语法`ArrayInit`。每条规则都有对应的方法。

   - `ArrayInitLexer.java`

     包含词法分析器的类定义，词法分析器的作用是将输入字符序列分解为词汇符号。

   - `ArrayInit.tokens`

     ANTLR会给每个词法符号指定一个数字形式的类型，同时存储关系在这个文件中。

   - `ArrayInitListener.java`

     定义遍历语法分析树的回调方法。

3. 编译

   需要提前设置好环境变量：

   > 参考：https://ghj1998.github.io/2021/04/26/ANTLR4%E6%8C%87%E5%8D%97/

   ```shell
   $ javac *.java
   ```

4. 运行

   同样要提前设置好才能使用`grun`，参考链接同上。

   ```shell
   $ grun ArrayInit init -tokens
   {99, 3, 451}
   ^Z
   
   // output:
   [@0,0:0='{',<1>,1:0]
   [@1,1:2='99',<4>,1:1]
   [@2,3:3=',',<2>,1:3]
   [@3,5:5='3',<4>,1:5]
   [@4,6:6=',',<2>,1:6]
   [@5,8:10='451',<4>,1:8]
   [@6,11:11='}',<3>,1:11]
   [@7,14:13='<EOF>',<-1>,2:0]
   ```

   ```bash
   $ grun ArrayInit init -gui
   {1,{2,3},4}
   ^Z
   
   ```

   output: 

   ![image-20210510162450230](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210510162450230.png)

### 2.3 从其他JAVA程序中调用生成的语法分析器

```java
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.*;


public class Test {
    public static void main(String[] args) throws Exception {
        // 新建一个CharStream，从标准输入读取数据
        ANTLRInputStream input = new ANTLRInputStream(System.in);
		
        // 新建一个词法分析器，处理输入的CharStream
        ArrayInitLexer lexer = new ArrayInitLexer(input);

        // 新建一个词法符号的缓冲区，用于存储词法分析器生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器，处理词法符号缓冲区的内容
        ArrayInitParser parser = new ArrayInitParser(tokens);

        // 针对init规则，开始进行语法分析
        ParseTree tree = parser.init();

        // 用LISP风格打印出生成的树
        System.out.println(tree.toStringTree(parser));
    }
}
```

验证：

```shell
$ java Test
{1,{2,3},4}
^Z

(init { (value 1) , (value (init { (value 2) , (value 3) })) , (value 4) })
```

同时，ANTLR还能自动报告语法错误。

```shell
$ java Test
{1,3
^Z

line 2:0 missing '}' at '<EOF>'
(init { (value 1) , (value 3) <missing '}'>)
```

> 上面提到的类，可以参考: **链接（此处留白）**

### 2.4 完成项目需求

程序要从语法分析树中提取数据，然后在触发的回调函数中进行适当操作。如果要通过编写程序操纵输入数据，需要继承`ArrayInitBaseListener`类，覆盖其中必要的方法。

我们的思想是：在遍历器进行语法分析树的遍历时，让每个监听器方法翻译输入数据的一部分并将结果打印出来。

**监听器的优点**在于：我们不需要写遍历语法分析树的代码。

对于这个项目需求，我们可以找出通用的翻译逻辑。

![image-20210510174143835](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210510174143835.png)

1. 把{翻译成"；
2. 把}翻译成"；
3. 把整数翻译成四位的十六进制形式，然后加前缀\u

第一步：编写方法，继承`BaseListener`类。

```java
public class ShortToUnicodeString extends ArrayInitBaseListener{

    @Override
    public void enterInit(ArrayInitParser.InitContext ctx) {
        System.out.print('"');
    }

    @Override
    public void exitInit(ArrayInitParser.InitContext ctx) {
        System.out.print('"');
    }

    @Override
    public void enterValue(ArrayInitParser.ValueContext ctx) {
        int value = Integer.valueOf(ctx.INT().getText());
        System.out.printf("\\u%04x",value);
    }
}
```

第二部，扩展main函数。

```java
import org.antlr.v4.runtime.*;
import org.antlr.v4.runtime.tree.*;


public class Test {
    public static void main(String[] args) throws Exception {
        // 新建一个CharStream，从标准输入读取数据
        ANTLRInputStream input = new ANTLRInputStream(System.in);
		
        // 新建一个词法分析器，处理输入的CharStream
        ArrayInitLexer lexer = new ArrayInitLexer(input);

        // 新建一个词法符号的缓冲区，用于存储词法分析器生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器，处理词法符号缓冲区的内容
        ArrayInitParser parser = new ArrayInitParser(tokens);

        // 遍历语法分析过程中生成的语法分析树，触发回调
      	ParseTreeWalker walker = new ParseTreeWalker();
        walker.walk(new ShortToUnicodeString(),tree);
        System.out.println();
    }
}
```

执行结果：

![image-20210511112729050](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511112729050.png)

