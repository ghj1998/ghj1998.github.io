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

我们可以使用p命名空间和c命名空间进行注入：

p命名空间相当于get，set注入。

c命名空间相当于构造器注入。

**实体类：**

```java
public class User {
    private String name;
    private int age;
}
```

省略了get，set，有参，无参构造方法。

**bean配置文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user1" class="com.gao.pojo.User" c:name="gao" c:age="18"/>
    <bean id="user2" class="com.gao.pojo.User" p:age="19" p:name="wei"/>
</beans>
```

注意：

- 要使用p命名空间和c命名空间，需要在开头加入配置信息。

  `xmlns:c="http://www.springframework.org/schema/c"
   xmlns:p="http://www.springframework.org/schema/p"`

**测试:**

```java
public void testCP(){
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final User user1 = context.getBean("user1", User.class);
    System.out.println(user1);
    final User user2 = context.getBean("user2", User.class);
    System.out.println(user2)
}
/*测试结果：
User{name='gao', age=18}
User{name='wei', age=19}
*/
```

### 7.4 bean作用域

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

1. 单例模式（Spring默认机制）：从一个bean中只能get到一个对象。

   两种写法等同。

   ```xml
   <bean id="user1" class="com.gao.pojo.User" c:name="gao" c:age="18"/>
   <bean id="user1" class="com.gao.pojo.User" c:name="gao" c:age="18" scope="singleton"/>
   ```

2. 原型模式：每次从容器中get都会获得一个新对象。

   ```xml
   <bean id="user2" class="com.gao.pojo.User" p:age="19" p:name="wei" scope="prototype"/>
   ```

3. 其余的request、session、application在web开发中使用。

**测试：**

```java
public void testCP(){
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final User user1 = context.getBean("user1", User.class);
    final User user11 = context.getBean("user1", User.class);
    System.out.println(user1==user11);
    final User user2 = context.getBean("user2", User.class);
    final User user22 = context.getBean("user2", User.class);
    System.out.println(user2==user22);
}
/*
true
false*/
```

可以看出从user1中获取的两个对象引用指向同一个对象，从user2中获取出两个对象。

## 8. bean自动装配

自动装配是Spring是满足bean依赖的一种方式。

spring会在上下文自动寻找，并自动给bean装配属性。

spring中有三种装配方式：

- 在xml中显示配置【已经学了】
- 在java中显示配置 【后面学】
- 隐式自动装配【重要】

### 8.1 byName自动装配

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="cat" class="Cat"/>
    <bean id="dog" class="Dog"/>
    <bean id="person" class="Person" autowire="byName">
        <property name="name" value="gao"/>
    </bean>
</beans>
```

### 8.2 byType自动装配

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="Cat"/>
    <bean class="Dog"/>
    <bean id="person" class="Person" autowire="byType">
        <property name="name" value="gao"/>
    </bean>
</beans>
```

### 8.3 注意

- byName要保证所有bean的id唯一，并且和自动注入中的set方法的参数名一致。
- byType，要保证Type的class唯一，并且bean和注入的参数类型一致。

### 8.4 注解自动装配

sping推荐注解而不是xml。

使用注解约束：

1. 导入约束`xmlns:context="http://www.springframework.org/schema/context"` `http://www.springframework.org/schema/context`

2. 配置注解的支持`<context:annotation-config/>`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
       <context:annotation-config/>
   </beans>
   ```

**@Autowired**

- @Autowired是按类型进行自动装配的，不支持id匹配。
- 使用@Autowired可以不用写set方法。
- @Autowired可以放在属性字段前，set方法前，构造方法前。一般放在字段前。
- 如果有多个相同的类型，需要使用@Qualifier来指定id从而选择装bean。@Qualifier不可以单独使用。
- @Autowired(required = false)表示该对象可以为空。默认为true，即如果找不到dog的bean会报错，加上required=false即可避免该问题。

实体类：

```java
public class Person {
    @Qualifier("cat11")
    @Autowired
    private Cat cat;
    @Autowired(required = false)
    private Dog dog;
    private String name;
}
```

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>
    <bean class="Cat"/>
    <bean id="cat11" class="Cat"/>
<!--    <bean class="Dog"/>-->
    <bean id="person" class="Person"/>
</beans>
```

测试：

```java
public void testAutowire() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        final Person person = context.getBean("person", Person.class);
        System.out.println(person);
}
/*
Person{cat=Cat@1d5872c, dog=null, name='null'}
*/
```

cat因为指定了Qualifier，所以在bean中选择id=cat11进行装配，dog因为设置了@Autowired(required = false)，所以找不到dog的bean，依旧可以完成装配，不会报错，输出的dog=null。

**@Resource**

- @Resource如果有指定的name属性，按该属性进行byName方式查找装配。
- 再按照默认的byName方式进行装配
- 如果都不成功，则按照byType方式进行自动装配。

实体类：

```java
public class Person {
    @Resource(name = "cat11")
    private Cat cat;
    @Resource
    private Dog dog;
    private String name;
}
```

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>
    <bean class="Cat"/>
    <bean id="cat11" class="Cat"/>
    <bean class="Dog"/>
    <bean id="person" class="Person"/>
</beans>
```

测试：

```java
public void testAutowire() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final Person person = context.getBean("person", Person.class);
    System.out.println(person);
}
```

cat因为制定了@Resource(name = "cat11")，所以使用id=cat11的bean进行装配，dog使用默认Resource，先去找id=dog的bean，没有找到，然后再根据byType进行查找，装配class=Dog的bean。

**比较**

1. @Autowired和@Resource都可以用来装配bean，都可以写在字段上或者setter方法上。

2. @Autowired默认按照类型装配（属于Spring规范），默认情况下要求依赖对象必须存在，如果要允许null值，可以设置他的required属性，如果想使用名称装配，可以搭配使用@Qualifier。
3. @Resource默认按照名称进行装配，名称可以通过name属性进行指定，如果没有，默认按照字段名进行查找，当找不到名称匹配的bean时才使用类型进行装配。如果name指定，则只会根据name进行装配。

## 9. 使用注解开发

spring4之后，使用注解开发必须导入AOP包。

使用注解需要导入context约束，增加注解的支持。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

<!--    指定要扫描的包，这个包下的注解就会生效-->
    <context:component-scan base-package="com.gao"/>
    <context:annotation-config/>
</beans>
```

注意：xsi:schemaLocation中的顺序不可以乱。

`<context:component-scan base-package="com.gao"/>`用来指定把哪些包里面的注解扫描近容器中。

### 9.1 bean

1. 将需要加入进容器中的类在配置中进行注册。

   ```xml
   <context:component-scan base-package="com.gao"/>
   // 假设这个类在com.gao这个包下
   ```

2. 编写实体类，增加注解

   ```java
   @Component
   public class User {
       public String name = "gao";
   }
   ```

3. 测试

   ```java
   public void testAnnotation(){
       final ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
       final User user = context.getBean("user", User.class);
       System.out.println(user.name);
   }
   ```

   测试结果：gao

   由此可见，成功将User类注入到IOC容器内。

### 9.2 属性如何注入

1. 不用使用set方法，可以在变量名上添加@Value(“值”)。

   ```java
   @Component
   public class User {
       @Value("gao")
       public String name;
   }
   ```

2. 使用set方法，可以在set方法上添加注解

   ```java
   @Component
   public class User {
       @Value("gao")
       public void setName(String name) {
           this.name = name;
       }
       public String name;
   }
   ```

### 9.3 衍生的注解

**@Component的三个衍生注解**

为了更好的分层，Spring可以使用其他三个注解，功能一样。

- @Controller ：web层
- @Service ：service层
- @Reposity ：dao层

### 9.4 自动装配

上面有。

### 9.5 作用域

**@Scope**

```java
@Component
@Scope("prototype")
public class User {
    @Value("gao")
    public String name;
}
```

默认时单例模式，可以使用**@Scope("prototype")**声明为原型模式。

### 9.6 小结

XML与注解比较

- XML可以适用于任何场景，结构清晰，维护方便。
- 注解开发简单方便

XML与注解整合开发：**推荐**

- XML管理Bean
- 注解完成属性注入
- 这种方法不需要扫描，因为扫描是为了类上的注解

```xml
<context:annotation-config/>
```

作用：

- 进行注解驱动注册，从而使注解生效
- 用于激活已经在spring注册过的bean上的注解
- 不扫描包的话，需要手动配置bean
- 如果不加注解驱动，注入的值为null

## 10. 基于JAVA类进行配置

### 10.1 @Bean

1. 实体类：

   ```java
   public class Dog {
       @Value("gao")
       public String name;
   }
   ```

2. JavaConfig配置

   ```java
   @Configuration
   public class MyConfig {
       @Bean
       public Dog dog(){
           return new Dog();
       }
   }
   ```

   用@Configuration声明表示这是一个javaconfig配置文件，其中需要用@Bean来声明要交给IOC管理的类。

   需要定义一个public方法，返回一个该类的对象，然后这个对象就交给了IOC容器去管理。

3. 测试

   ```java
   public void testJAVAConfig(){
       ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
       final Dog dog = context.getBean("dog", Dog.class);
       System.out.println(dog.name);
   }
   ```

   使用javaconfig需要使用`AnnotationConfigApplicationContext(MyConfig.class)`来获取context对象。

   context获取Bean的时候，传入的参数时在javaconfig定义的方法的方法名。

### 10.2 @ComponentScan(“包名”)

1. 实体类

   ```java
   @Component
   public class Dog {
       @Value("gao")
       public String name;
   }
   ```

2. javaConfig配置

   ```java
   @Configuration
   @ComponentScan("com.gao.pojo")
   public class MyConfig {
       public Dog dog(){
           return new Dog();
       }
   }
   ```

   使用@Configuration声明这是个配置文件，用@ComponentScan("com.gao.pojo")声明去哪些包扫描带有@Component声明的组件。

3. 测试不变。

### 10.3 @import

使用Import可以导入其他的javaConfig配置到一个JavaConfig配置中

```java
@Configuration
@ComponentScan("com.gao.pojo")
@Import({MyConfig2.class,MyConfig3.class})
public class MyConfig {
    public Dog dog(){
        return new Dog();
    }
}
```

这样，在使用`AnnotationConfigApplicationContext(MyConfig.class)`获取context的时候就不用引入MyConfig2和MyConfig3。

