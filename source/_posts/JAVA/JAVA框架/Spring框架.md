---
title: Spring框架
date: 2021-05-04 20:03:48
tags:
---

# Spring

# 1. 简介

- 免费开源的框架
- 轻量级、非入侵（不会对原来的项目造成影响）
- 控制反转（IOC），面向切面编程（AOP）
- 支持事务处理，对框架整合的支持

Maven导入：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.6</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.6</version>
</dependency>
```

导入MVC是因为MVC能够自动导入很多其他依赖的包。
