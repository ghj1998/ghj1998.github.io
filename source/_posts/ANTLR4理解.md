---
title: ANTLR4理解
date: 2021-05-8 16:04:47
tags: 
- ANTLR
- 语法分析

---

# ANTLR背后在干什么？

## 1. 语法分析的过程

**语法分析**由两个阶段：

- **词法分析(lexical analysis)/词法符号化(tokenizing)**：将字符聚集为单词或者符号(**词法符号，token**)的过程，将**输入文本**转化为**词法符号**的过程称为**词法分析器(lexer)**。**词法分析器**可以将词法符号分类，例如INT，FLOAT等等。词法符号包含至少两部分信息，词法符号的类型以及该符号对应的文本。
- **识别语句结构**：构造语法分析树来记录语法分析器的数据结构。

总的来说，就是先划分成词，再分析。

例如：

![image-20210426213615055](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210426213615055.png)

**语法分析树解析**：

- 叶子节点永远是输入的词法符号。
- 根节点是抽象的名字，stat是statement的简写。
- 树的结构在后续很好处理，并且能传递完整的符号信息。
- 子树的根节点是对应语法规则的名字。

![image-20210426215414826](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210426215414826.png)

这就是语法规则。

## 2. 实现一个语法分析器

ANTLR工具依赖于类似于assign的语法规则，产生递归下降的语法分析器。

**什么是递归下降？**

从根节点开始，向叶节点解析。

举例：

例如如下的语法规则

```apl
assin : ID '=' expr ';';
```

语法分析器实现的细节：

```java
void assign(){
    match(ID);
    match('=');
    expr();
    match(";");
}
```

`stat()`是根节点。

`assign()`和`expr()`都是调用路线，`match()`对应了语法规则的叶子节点。

`stat`规则能对应多个`assign`级别的分支。

例如：

```api
stat: assign   //第一备选分支
    | ifstat   //第二备选分支
    | whilestat
    ...
    ;
```

解析的话类似于`switch`语句：

```java
void stat(){
    switch(<< 当前输入的词法符号 >>){
            CASE ID ：assign();break;
            CASE IF : ifstat();break;
            CASE WHILE ：whilestat();break;
            ...
            default : <<抛出无可选方案的异常>>
    }
}
```

`stat()`通过检查下一个词法符号来做出决策。例如，如果遇到了WHILE关键字，则选择第三个分支备选。同时`stat()`调用`whilestat()`方法。

下一个词法符号又称为**前瞻词法符号（lookahead token）**

但是，选择分支往往很多，就导致语法分析器往往需要多个前瞻词法符号才能指导去哪个分支。

**ANTLR能够根据情况调整前瞻数量。**

所以：我们不用考虑了哈哈哈哈。

**合法语句**：能够顺利从开始走到语句结尾，就是合法语句。
也就是说，如果在中途没办法做出选择，表示，语句本身出了问题。

## 3. 避免歧义性语句

歧义性举例：

```api
stat: expr ';'
     | ID '(' ')' ';'
     ;
expr: ID '(' ')'
	| INT
	;
```

上述规则如果匹配 "`f();`"

![image-20210508160604632](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210508160604632.png)

就会出现如上两种情况。左边是匹配了expr规则，右边是匹配了stat规则的第二个备选分支。

ANTLR会选择左边的语法分析树对`f()`进行解释。

## 4. 利用语法分析树构建语言类应用程序

### 4.1 ANTLR使用的数据结构和类名

ANTLR类：`CharStream`，`Lexer`，`Token`，`Parser`，`ParseTree`。

连接词法分析器和语法分析器的管道是`TokenStream`。

整体交互如图所示：

<img src="https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210508162752717.png" alt="image-20210508162752717" style="zoom:50%;" />

`TokenStream`只记录了`CharStream`中字符序列的开始位置和结束位置。目的是：共享数据结构来节约内存。

`ParseTree`的子类`RuleNode`和`TerminalNode`，分别是子树的根节点和叶子节点。`RuleNode`规则不是一成不变，实际上是`StatContext`、`AssignContext`以及`ExprContext`。

<img src="https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210508163241574.png" alt="image-20210508163241574" style="zoom:75%;" />



