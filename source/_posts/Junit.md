---
title: Junit
date: 2021-05-04 20:20:13
tags: 
- Junit
- JAVA
---

# junit

## 1. 安装

安装步骤：

1. 下载Junit jar包
2. 增加CLASSPATH环境变量，把JUnit jar放入CLASSPATH

**注意**：Maven需要在pom.xml中添加依赖。

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

