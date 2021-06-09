---
title: Spring框架
date: 2021-06-09 22:03:48
tags: 
- Spring
- JAVA
---

# Spring

## 1. 简介

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

## 2. Spring组成和扩展

![image-20210605233732916](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210605233732916.png)

- Spring Boot
  - 一个快速开发的脚手架
  - 基于SpringBoot可以快速的开发单个微服务
  - 约定大于配置
- Spring Cloud
  - SpringCloud是基于SpringBoot实现的

学习SpringBoot前提是必须学了Spring和SpingMVC。

**Spring弊端：发展太久，违背了原来的理念，配置繁琐，人称：配置地狱。**

所以诞生了SpringBoot。

## 3. IOC理论推导

原来写程序的步骤：

整体架构图：

![image-20210606002135773](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210606002135773.png)

1. UserDao接口

   ```java
   public interface UserDao {
       public void getUser();
   }
   ```

2. UserDaoImp实现类

   设计两个UserDao的实现类：

   ```java
   public class UserDaoMysqlImpl implements UserDao{
       @Override
       public void getUser() {
           System.out.println("Mysql database");
       }
   }
   ```

   ```java
   public class UserDaoOracleImpl implements UserDao{
       @Override
       public void getUser() {
           System.out.println("Oracle database");
       }
   }
   ```

3. UserService业务接口

   ```java
   public interface UserService {
       public void getUser();
   }
   ```

4. UserServiceImpl 业务实现类

   ```java
   public class UserServiceImp implements UserService{
   
       UserDao userDao = new UserDaoMysqlImpl();
   
       @Override
       public void getUser() {
           userDao.getUser();
       }
   }
   ```

5. 测试

   ```java
   public class MyTest {
       @Test
       public void testUserService(){
           UserService userService = new UserServiceImp();
           userService.getUser();
       }
   }
   ```

采用这种流程，我们发现每一次要重新使用一种UserDao的实现类时，都要修改UserServiceImp类，如果类很多的话就很麻烦。

我们可以使用set来进行改进。

UserServiceImpl 业务实现类改写成：

```java
public class UserServiceImp implements UserService{
    UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

这样的话，如果要新加入一种数据库，我们只需要修改Dao层，以及在测试的时候指定要具体实现的UserDao实现类。

```java
public class MyTest {
    @Test
    public void testUserService(){
        UserService userService = new UserServiceImp();
        ((UserServiceImp)userService).setUserDao(new UserDaoOracleImpl());
        userService.getUser();
    }
}
```

- 之前，程序主动创建对象，控制权是程序员的。
- 现在，程序不具有主动性，变成了被动的接受对象。

这样，程序员不用去管理对象的创建了，系统耦合性降低，可以专注在业务实现。

这是IOC的原型！

**控制反转IOC**就是获得依赖对象的方式反转了。

在Spring中实现控制反转的是**IOC容器**，实现方法是 **依赖注入（DI）**。

## 4. HelloSpring

实体类：Hello.class

```java
public class Hello {
    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    private String str;

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }
}
```

spring配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
        <!--
        使用Spring创建对象，在Spring中这些都称为Bean
        原来需要去new对象，现在让容器来做这件事，我们写配置就可以了。
        bean = 对象
        id=变量名
        class = new的对象
        property 相当于给对象中的属性赋值
        -->
    <bean id="hello" class="com.gao.pojo.Hello">
        <property name="str" value="Spring"></property>
    </bean>
</beans>
```

测试：

```java
public void testSpringHello(){
//        获取Spring上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
//        我们的对象现在都在Spring中管理，我们要使用，直接去里面取。
        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.toString());
}
```

ClassPathXmlApplicationContext类的继承关系如下：

![diagram](https://raw.githubusercontent.com/ghj1998/image_repository/main/diagram.png)

我们全程没有使用new关键字！

- Hello对象谁创建的？

  Spring。

- Hello对象的属性怎么设置的？

  Spring容器设置的。

这个过程就叫做控制反转！

**控制**：谁来控制对象的创建，传统应用对象是程序本身控制创建的，使用Spring后，由Spring来创建。

**反转**：程序本身不创建对象，而变成被动的接收对象。

**依赖注入**：使用set方法来进行注入。

IOC是一种编程思想，由主动的编程变成被动的接收。

所以！我们彻底不用去程序中改动了，要实现不同的操作，只需要在xml配置文件中进行修改，IOC，用一句话表示就是：**对象由Spring创建、管理、装配。**

例如利用Spring修改IOC理论推导中的代码。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="mysqlImp" class="com.gao.dao.UserDaoMysqlImpl"/>
    <bean id="oracleImp" class="com.gao.dao.UserDaoOracleImpl"/>
    <bean id="userServiceImp" class="com.gao.service.UserServiceImp">
        <property name="userDao" ref="oracleImp"/>
    </bean>
    <!--
        ref: 引用spring容器中创建好的对象
        value：具体的值，基本数据类型
    -->
</beans>
```

注意property中的value和ref的区别。

测试

```java
public void testUserService(){
    // 获取ApplicationContext
    final ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    // 需要什么get什么
    final UserServiceImp userServiceImp = (UserServiceImp)context.getBean("userServiceImp");
    userServiceImp.getUser();
}
```

使用spring后，如果想修改数据库类型，再也不用去动代码！

直接修改配置文件就可以，比如说想把数据库从oracle变成mysql。

用户直接修改```<property name="userDao" ref="oracleImp"/>```，把ref改成```<property name="userDao" ref="mysqlImp"/>```即可。不用去操作任何的代码。

这就是**Spring中的IOC**。

## 5. IOC创建对象的方式

`ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");`

**在执行到加载配置文件后，定义在配置文件中的对象已经创建出来了。**

1. 使用无参构造创建对象，默认！

2. 假设要使用有参构造创建对象：
	```java
	public class User {
   		private String name;
    	public User(String name) {
        	this.name = name;
    	}
	}
	```

   1. 下标赋值

      ```xml
      <bean id="user" class="com.gao.User">
          <constructor-arg index="0" value="gao"/>
      </bean>
      ```

   2. 类型赋值

      ```java
      <bean id="user" class="com.gao.User">
          <constructor-arg type="java.lang.String" value="gao"/>
      </bean>
      ```

   3. 参数名赋值

      ```xml
      <bean id="user" class="com.gao.User">
          <constructor-arg name="name" value="gao"/>
      </bean>
      ```

## 6. Spring配置

### 6.1 别名

```xml
<alias name="user" alias="bieming"/>
```

给某个bean起别名后，可以用别名来获得对象，也可以通过原来的名字获取别名。

```java
User user = (User) context.getBean("bieming");
```

### 6.2 Bean的配置

id：bean的唯一标识符。

class：bean对象对应的全限定名。

name：也是别名，可以取多个别名。

```xml
<bean id="user" class="com.gao.User" name="user2,user3,user4">
```

### 6.3 import

import多用于团队开发，将多个配置文件合并导入成一个。

例如，可以在`applicationContext.xml`中导入`beans.xml`,`beans1.xml`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <import resource="beans.xml"/>
</beans>
```

这样就可以通过

```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

导入多个xml配置文件。

内容相同会合并！

## 7. 依赖注入

### 7.1 构造器注入

前面的注入方法就是构造器注入！

### 7.2 Set方法注入

- 依赖注入：Set注入！
  - 依赖：bean对象的创建依赖于容器
  - 注入：bean对象中的所有属性由容器来注入

**环境**

1. 复杂类型

   复杂类型Student以及引用类Address：

   ```java
   public class Student {
       private String name;
       private Address address;
       private String[] books;
       private List<String> hobbys;
       private Map<String,String> card;
       private Set<String> games;
       private Properties info;
       private String wife;
   }
   ```

   ```java
   public class Address {
       private String address;
   }
   ```

   需要实现getset方法，该处省略没写。

2. 真实测试对象

   注入：

   其中包括普通值，引用值，数组，list，set，map，空指针以及props。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       <bean id="student" class="com.gao.pojo.Student">
   <!--        普通值注入-->
           <property name="name" value="gao"/>
   <!--        Bean注入，ref-->
           <property name="address" ref="address"/>
   <!--        数组注入，array,value-->
           <property name="books">
               <array>
                   <value>红楼梦</value>
                   <value>西游记</value>
                   <value>三体</value>
               </array>
           </property>
   <!--        list注入,list,value-->
           <property name="hobbys">
               <list>
                   <value>听歌</value>
                   <value>看电视</value>
               </list>
           </property>
   <!--        map注入-->
           <property name="card">
               <map>
                   <entry key="身份证" value="123456"/>
                   <entry key="银行卡" value="13456"/>
               </map>
           </property>
   <!--        set注入-->
            <property name="games">
                <set>
                    <value>LOL</value>
                    <value>COC</value>
                    <value>COC</value>
                </set>
            </property>
   <!--        null-->
           <property name="wife">
               <null></null>
           </property>
   <!--        properties  key=value的格式-->
           <property name="info">
               <props>
                   <prop key="学号">20190525</prop>
                   <prop key="姓名">gao</prop>
               </props>
           </property>
       </bean>
       <bean id="address" class="com.gao.pojo.Address">
           <property name="address" value="shanxi"/>
       </bean>
   </beans>
   ```

   **注意：property的注入方式和map有区别！**

3. 测试

   ```java
   import org.junit.Test;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;
   
   public class MyTest {
       @Test
       public void testStudentBean(){
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
           final Object student = context.getBean("student");
           System.out.println(student.toString());
       }
   }
   ```

   测试结果：

   ```yaml	
   Student{name='gao', address=shanxi, books=[红楼梦, 西游记, 三体], hobbys=[听歌, 看电视], card={身份证=123456, 银行卡=13456}, games=[LOL, COC], info={学号=20190525, 姓名=gao}, wife='null'}
   ```

### 7.3 拓展方法注入