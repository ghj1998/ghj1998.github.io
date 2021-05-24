---
title: Mybatis框架
date: 2021-05-04 20:19:12
tags: 
- JAVA
- Mybatis
---

# Mybatis

## 1. 简介

- MyBatis 是一款优秀的**持久层**框架，

- 它支持自定义 SQL、存储过程以及高级映射。
- MyBatis 免除了几乎所有的 JDBC 代码以及设置 参数和获取结果集的工作。
- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 1.1 **如何使用？**

- Maven

  ```xml
  <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.2</version>
  </dependency>
  ```

  

- Github开源

- 中文文档https://mybatis.org/mybatis-3/zh/index.html

### 1.2 持久化

**数据持久化**：

持久化就是将程序中的数据再持久状态和瞬时状态转化的过程。

数据持久化存储：数据库，IO文件。

**为什么要持久化?**

因为内存断电即失，所以要将数据持久化存储。

### 1.3 持久层

层：例如：Dao层，Service层，Controller层。

- 完成持久化工作的代码块。
- 层的界限很明显。

### 1.4 为什么要Mybatis？

- 比JDBC代码好些。
- 填写框架就可以，自动化。
- sql和代码分离。
- 提供映射标签，支持对象和数据库的orm字段关系映射
- 提供对象关系映射标签，支持对象关系组建维护。
- 提供xml标签，支持编写动态sql。

### 1.4 学完之后学啥

Spring，SpringMVC，SpringBoot。



 