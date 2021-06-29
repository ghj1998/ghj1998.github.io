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

      ```xml
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

## 11 AOP

### 11.1 代理模式

代理模式是Spring-AOP的底层。

代理模式分类：

- 静态代理
- 动态代理

#### 11.1.1 静态代理

角色分析：

- 抽象角色：一般会采用接口或者抽象类解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，一般会做一些附属操作
- 客户：使用代理角色进行一些操作

例子1：租房

代码实现：

Rent.java

```java
// 抽象角色
public interface Rent(){
    public void rent();
}
```

Host.java

```java
//真实角色：房东，房东要出租房子
public class Host implements Rent{
    public void rent(){
        System.out.println("房屋出租");
    }
}
```

Proxy.java

```java
public class Proxy implements Rent{
    private Host host;
    public Proxy(){}
    public Proxy(Host host){
        this.host=host;
    }
    
    //租房
    public void rent(){
        seeHouse();
        host.rent();
        fee();
    }
    //看房
    public void seeHouse(){
        System.out.println("带房客看房");
    }
    //收中介费
    public void fee(){
        System.out.prinln("收中介费");
    }
}
```

Client.java客

```java
//客户类，一般客户不找真实对象，找代理。
public class Client{
    public static void main(String[] args){
        //房东要租房
        Host host = new Host();
        //中介帮助房东
        Proxy proxy = new Proxy(host);
        
        //找中介租房
        proxy.rent();
    }
}
```

例子2：扩展原有功能，增加日志功能

抽象角色

```java
// 抽象角色：增删改查业务
public interface UserService{
    void add();
    void delete();
    void update();
    void query();
}
```

真实对象

```java
// 真实对象，完成增删改查。
public class UserServiceImpl implements UserService{
    public void add(){
        System.out.println("增加一个用户");
    }
    public void delete(){
        System.out.println("删除了一个用户");
    }
    public void update(){
        System.out.println("更新了一个用户");
    }
    public void query(){
        System.out.println("查询了一个用户");
    }
}
```

使用代理对象来增加日志功能。

```java
//代理角色，增加日志实现
public class UserServiceProxy implements UserService{
    private UserServiceImpl userService;
    
    public void setUserService(UserServiceImpl userService){
        this.userService = userService;
    }
    public void log(String msg){
        System.out.println("执行了"+msg+"方法");
    }
    public void add(){
        log("add");
        userService.add();
    }
    public void delete(){
        log("delete");
        userService.delete();
    }
    public void update(){
        log("update");
        userService.update();
    }
    public void query(){
        log("query");
        userService.query();
    }
}
```

用户类

```java
public class Client{
    public static void main(String[] args){
        //真实业务
        UserService userService= new UserServiceImpl();
        //代理类
        UserService  proxy = new UserServiceProxy();
        //使用代理类来实现日志功能
        proxy.setUserService(userService);
        proxy.add()
    }
}
```

**我们在不改变原来代码的情况下，实现了对原有功能的增强，这是AOP中最核心的思想。**

静态代理的好处：

- 可以使真实角色更加纯粹，不再关注公共的事情。
- 公共的业务由代理来完成，实现了业务的分工
- 公共业务发生扩展时更加方便和集中

缺点：

- 工作量增大，开发效率降低

因此，有了动态代理。

#### 11.1.2 动态代理

JDK提供了java.lang.reflect.InvocationHandler接口和 java.lang.reflect.Proxy类，这两个类相互配合实现动态代理。

Proxy类，调用它的newInstance方法可以生成某个对象的代理对象,该方法需要三个参数：

```java
newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
```

- 参数一：生成代理对象使用哪个类装载器【一般我们使用的是被代理类的装载器】
- 参数二：生成哪个对象的代理对象，通过接口指定【指定要被代理类的接口】
- 参数三：生成的代理对象的方法里干什么事【实现handler接口，我们想怎么实现就怎么实现】

在编写动态代理之前，要明确几个概念：

- 代理对象拥有目标对象相同的方法【因为参数二指定了对象的接口，代理对象会实现接口的所有方法】
- 用户调用代理对象的什么方法，都是在调用处理器的invoke方法。【被拦截】
- 使用JDK动态代理必须要有接口【参数二需要接口】

代理对象会实现接口的所有方法，这些实现的方法交由我们的**handler**来处理！

- 所有通过动态代理**实现的方法全部**通过`invoke()`调用

```java
invoke(Object proxy, Method method, Object[] args)
```

动态代理的调用关系：

![image-20210614124325848](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210614124325848.png)

见如下例子：

功能接口：

```java
public interface UserService {
    public void add();
    public void delete();
    public void query();
}
```

真实对象：

```java
public class UserServiceImpl implements UserService{

    @Override
    public void add() {
        System.out.println("add");
    }

    @Override
    public void delete() {
        System.out.println("delete");
    }

    @Override
    public void query() {
        System.out.println("query");
    }
}
```

代理工厂：生成代理对象

```java
public class UserServiceProxyFactory implements InvocationHandler {
    public void setTarget(Object target) {
        this.target = target;
    }
    private Object target;
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println(method.getName());
        Object ref = method.invoke(target,args);
        System.out.println(method.getName()+"结束");
        return ref;
    }
}
```

`Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),this);`

使用Proxy类的newProxyInstance静态方法，传入要代理的对象的类加载器，以及这个对象的接口，还要传入当前的这个代理工厂对象【目的是为了找到invoke方法】。

invoke方法是InvocationHandler唯一的接口方法，在其中可以实现代理类的一些操作，例如打印日志等，使用反射来调用真实对象的方法。

用户类：

```java
public class Client {
    public static void main(String[] args) {
        // 原对象
        UserService userService = new UserServiceImpl();
        userService.add();
        userService.delete();
        
        //代理工厂
        UserServiceProxyFactory userServiceProxyFactory = new UserServiceProxyFactory();
        
        //从代理工厂生成代理对象
        userServiceProxyFactory.setTarget(userService);
        UserService userServiceProxy = (UserService)userServiceProxyFactory.getProxyInstance();
        userServiceProxy.add();
        userServiceProxy.delete();
    }
}
```

### 11.2 AOP简介

AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。

利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

![image-20210623194313619](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210623194313619.png)

### 11.3 Spring中Aop的相关概念

提供声明式事务；允许用户自定义切面

以下名词需要了解下：

- 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志 , 安全 , 缓存 , 事务等等 ....
- 切面（ASPECT）：横切关注点 被模块化 的特殊对象。即，它是一个类。
- 通知（Advice）：切面必须要完成的工作。即，它是类中的一个方法。
- 目标（Target）：被通知对象。
- 代理（Proxy）：向目标对象应用通知之后创建的对象。
- 切入点（PointCut）：切面通知 执行的 “地点”的定义。
- 连接点（JointPoint）：与切入点匹配的执行点。

![image-20210623194533722](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210623194533722.png)

SpringAOP中，通过Advice定义横切逻辑，Spring中支持5种类型的Advice:

| 通知类型     | 连接点               | 实现接口                                        |
| ------------ | -------------------- | ----------------------------------------------- |
| 前置通知     | 方法前               | org.springframework.aop.MethodBeforeAdvice      |
| 后置通知     | 方法后               | org.springframework.aop.AfterReturningAdvice    |
| 环绕通知     | 方法前后             | org.springframework.aop.MethodInterceptor       |
| 异常抛出通知 | 方法抛出异常         | org.springframework.aop.ThrowsAdvice            |
| 引介通知     | 类中增加新的方法属性 | org.springframework.aop.IntroductionInterceptor |

### 11.4 Spring实现AOP

**【重点】使用AOP织入，需要导入一个依赖包！**

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjweaver</artifactId>
   <version>1.9.4</version>
</dependency>
```

#### 11.4.1 通过Spring API实现

业务接口：

```java
public interface UserService {
   public void add();
   public void delete();
   public void update();
   public void search();
}
```

业务实现：

```java
public class UserServiceImpl implements UserService{
   @Override
   public void add() {
       System.out.println("增加用户");
  }
   @Override
   public void delete() {
       System.out.println("删除用户");
  }
   @Override
   public void update() {
       System.out.println("更新用户");
  }
   @Override
   public void search() {
       System.out.println("查询用户");
  }
}
```

为了增强业务功能，编写两个增强类，一个前置，一个后置。这两个类就是**切面**

```java
public class Log implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("执行了"+o.getClass().getSimpleName()+"的"+method.getName()+"方法");
    }
}
```

```java
public class AfterLog implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println(method.getName()+"的返回值是"+returnValue);
    }
}
```

编写配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--注册bean-->
    <bean id="userService" class="com.gao.service.UserServiceImpl"/>
    <bean id="log" class="com.gao.log.Log"/>
    <bean id="afterLog" class="com.gao.log.AfterLog"/>
    <!--aop的配置-->
    <aop:config>
        <!--切入点 expression:表达式匹配要执行的方法-->
        <aop:pointcut id="pointcut" expression="execution(* com.gao.service.UserServiceImpl.*(..))"/>
        <!--执行环绕; advice-ref执行方法 . pointcut-ref切入点-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

1. 第0步：导入AOP依赖
2. 第一步：注册所有的Bean
3. 配置AOP `<aop:config>`
   1. aop:pointcut用来指定要插入的位置，即**切入点**，其中execution(修饰符  返回值  包名.类名/接口名.方法名(参数列表))。(..)可以代表所有参数,(*)代表一个参数。execution用来指定在什么地方插入通知。
   2. aop:advisor指定要插入的日志。即**通知**。

测试：

```java
public void testAOP(){
    ApplicationContext context = new ClassPathXmlApplicationContext("config.xml");
    final UserService service = (UserService) context.getBean("userService");
    service.add();
}
/*
执行了UserServiceImpl的add方法
增加用户
add的返回值是null
*/
```

注意：因为AOP返回的动态代理的对象，不是原来的UserServiceImp对象，所以需要使用UserService接口来接受这个代理对象。

代理对象和UserServiceImp都实现了UserService接口。

**Spring的Aop就是将公共的业务 (日志 , 安全等) 和领域业务结合起来 , 当执行领域业务时 , 将会把公共业务加进来 . 实现公共业务的重复利用 . 领域业务更纯粹 , 程序猿专注领域业务 , 其本质还是动态代理** 

#### 11.4.2 自定义类来实现AOP

实体类不变。

增加一个自定义类：

```java
public class Log {
    public void before(){
        System.out.println("======before=======");
    }
    public void after(){
        System.out.println("======after========");
    }
}
```

修改配置文件，增加如下内容。

```xml
<bean id="diyLog" class="com.gao.diy.Log"/>
<aop:config>
    <aop:aspect ref="diyLog">
        <aop:pointcut id="pointcut" expression="execution(* com.gao.service.UserServiceImpl.*(..))"/>
        <aop:before pointcut-ref="pointcut" method="before"/>
        <aop:after method="after" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

在aop:aspect中指定要插入的**切面**，也就是我们自定义的日志。

在其中用aop:before和aop:after来指定方法在什么地方触发。

但是没办法处理复杂问题。

测试类不变：

输出：

```java
/*
======before=======
增加用户
======after========
*/
```

#### 11.4.3 注解实现AOP

第一步：编写一个注解实现的增强类

```java
@Aspect
public class AnnotationPointcut {
   @Before("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void before(){
       System.out.println("---------方法执行前---------");
  }

   @After("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void after(){
       System.out.println("---------方法执行后---------");
  }

   @Around("execution(* com.kuang.service.UserServiceImpl.*(..))")
   public void around(ProceedingJoinPoint jp) throws Throwable {
       System.out.println("环绕前");
       System.out.println("签名:"+jp.getSignature());
       //执行目标方法proceed
       Object proceed = jp.proceed();
       System.out.println("环绕后");
       System.out.println(proceed);
  }
}
```

第二步：修改配置文件

```xml
<bean id="annotationPointcut" class="com.kuang.config.AnnotationPointcut"/>
<aop:aspectj-autoproxy/>
```

<aop:aspectj-autoproxy />有一个proxy-target-class属性，默认为false，表示使用jdk动态代理织入增强，当配为<aop:aspectj-autoproxy  poxy-target-class="true"/>时，表示使用CGLib动态代理技术织入增强。不过即使proxy-target-class设置为false，如果目标类没有声明接口，则spring将自动使用CGLib动态代理。

第三步：测试

测试类不变

```java
/*
环绕前
签名:void com.gao.service.UserService.add()
---------方法执行前---------
增加用户
---------方法执行后---------
环绕后
null
*/
```

## 12. Spring-Mybatis

### 12.1 回顾Mybatis

- Mybatis-config配置

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <typeAliases>
          <package name="com.gao.pojo"/>
      </typeAliases>
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"/>
                  <property name="username" value="root"/>
                  <property name="password" value="19980913"/>
              </dataSource>
          </environment>
      </environments>
      <mappers>
          <package name="com.gao.mapper"/>
      </mappers>
  </configuration>
  ```

  注意要注册mapper。

- 实体类

  ```java
  @Data
  public class User {
      private int id;
      private String name;
      private String pwd;
  }
  ```

- UserMapper接口

  ```java
  public interface UserMapper {
      public List<User> getUserList();
  }
  ```

- Mapper映射文件

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.gao.mapper.UserMapper">
      <select id="getUserList" resultType="User">
          select * from user
      </select>
  </mapper>
  ```

- 测试

  ```java
  public void testGetUserList() throws FileNotFoundException {
      Reader reader = new FileReader("C:\\Users\\gao\\Desktop\\Spring_study\\Spring_Mybatis\\src\\main\\resources\\Mybatis-config.xml");
      final SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
      final SqlSession sqlSession = sqlSessionFactory.openSession();
      final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      final List<User> userList = mapper.getUserList();
      for (User user : userList) {
          System.out.println(user);
      }
  }
  /*
  User(id=1, name=高一, pwd=19980913)
  User(id=2, name=高二, pwd=19980914)
  User(id=3, name=高三, pwd=19980915)
  User(id=4, name=高四四, pwd=159357)
  User(id=5, name=高五, pwd=147258369)
  User(id=6, name=高五五, pwd=123741)
  */
  ```

### 12.2 mybatis-spring

http://www.mybatis.org/spring/zh/index.html  mybatis-spring官网。

MyBatis-Spring 需要以下版本：

| MyBatis-Spring | MyBatis | Spring 框架 | Spring Batch | Java    |
| :------------- | :------ | :---------- | :----------- | :------ |
| 2.0            | 3.5+    | 5.0+        | 4.0+         | Java 8+ |
| 1.3            | 3.4+    | 3.2.2+      | 2.1+         | Java 6+ |

要和 Spring 一起使用 MyBatis，需要在 Spring 应用上下文中定义至少两样东西：一个 **SqlSessionFactory** 和至少一个数据映射器类。

#### 12.2.1 整合方法1

**第一步：在Spring中配置SqlSessionFactory：**

在 MyBatis-Spring 中，可使用**SqlSessionFactoryBean**来创建 **SqlSessionFactory**。要配置这个工厂 **bean**，还需要一个**dataSource**的Bean，只需要把下面代码放在 Spring 的 XML 配置文件中：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
</bean>
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=utf8"/>
    <property name="password" value="19980913"/>
    <property name="username" value="root"/>
</bean>
```

这一步写完，就不再需要mybatis中的配置了，即不需要再mybatis-config.xml中配置environment。

通过在sqlSessionFactory下进行配置，可以将spring配置文件和mybatis配置文件连接起来，同时可以设置别名，mapper注册等等。

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <!--绑定mybatis配置文件-->
    <property name="configLocation" value="classpath:Mybatis-config.xml"/>
    <property name="mapperLocations" value="classpath:com/gao/mapper/*.xml"/>
</bean>
```

这样修改spring配置文件后，mybatis中的很多内容可以删除。

```xml
<configuration>
    <typeAliases>
        <package name="com.gao.pojo"/>
    </typeAliases>
</configuration>
```

**第二步：设置SqlSessionTemplate**

SqlSessionTemplate就是Mybatis中的SqlSession。

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```

SqlSessionTemplate没有set方法，因此只能选择构造器注入。

此时注册完后，就可以从Spring容器中获得SqlSession进而访问数据库。

```java
public void testGetUserList() throws FileNotFoundException {
    final ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final SqlSession sqlSession = context.getBean("sqlSession", SqlSession.class);
    final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    final List<User> userList = mapper.getUserList();
    for (User user : userList) {
        System.out.println(user);
    }
}
/*
User(id=1, name=高一, pwd=19980913)
User(id=2, name=高二, pwd=19980914)
User(id=3, name=高三, pwd=19980915)
User(id=4, name=高四四, pwd=159357)
User(id=5, name=高五, pwd=147258369)
User(id=6, name=高五五, pwd=123741)
*/
```

**第三步：将getMapper封装到UserMapper的实现类中**

```java
public class UserMapperImpl implements UserMapper{
    private SqlSessionTemplate sqlSession;
    @Override
    public List<User> getUserList() {
        final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.getUserList();
        
    }
    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }
}
```

注册Bean：

```xml
<bean id="UserMapper" class="com.gao.mapper.UserMapperImpl">
    <property name="sqlSession" ref="sqlSession"></property>
</bean>
```

测试：

```java
public void testGetUserList() throws FileNotFoundException {
    final ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final UserMapper mapper = context.getBean("UserMapper",UserMapper.class);
    for (User user : mapper.getUserList()) {
        System.out.println(user);
    }
}
/*
User(id=1, name=高一, pwd=19980913)
User(id=2, name=高二, pwd=19980914)
User(id=3, name=高三, pwd=19980915)
User(id=4, name=高四四, pwd=159357)
User(id=5, name=高五, pwd=147258369)
User(id=6, name=高五五, pwd=123741)
*/
```

#### 12.2.1 整合方法2

**第一步和整合方法1相同**，仍然是需要注册一个**SqlSessionFactory** bean。

**第二步**：Mapper的实现类直接继承**SqlSessionDaoSupportSupport**类 , 直接利用 **getSqlSession()** 获得**sqlSession**。注意在注册bean的时候要注入SqlSessionFactory。

```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {
    @Override
    public List<User> getUserList() {
        final UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        return mapper.getUserList();
    }
}
```

```xml
<bean id="userMapper2" class="com.gao.mapper.UserMapperImpl2">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
</bean>
```

测试：

```java
public void testGetUserList() throws FileNotFoundException {
    final ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final UserMapper mapper = context.getBean("userMapper2",UserMapper.class);
    for (User user : mapper.getUserList()) {
        System.out.println(user);
    }
}
```

输出相同。

## 13. 声明式事务

- 事务在项目开发过程非常重要，涉及到数据的一致性的问题，不容马虎！
- 事务管理是企业级应用程序开发中必备技术，用来确保数据的完整性和一致性。

事务就是把一系列的动作当成一个独立的工作单元，这些动作要么全部完成，要么全部不起作用。

**事务四个属性ACID**

1. 原子性（atomicity）

2. - 事务是原子性操作，由一系列动作组成，事务的原子性确保动作要么全部完成，要么完全不起作用

3. 一致性（consistency）

4. - 一旦所有事务动作完成，事务就要被提交。数据和资源处于一种满足业务规则的一致性状态中

5. 隔离性（isolation）

6. - 可能多个事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏

7. 持久性（durability）

   - 事务一旦完成，无论系统发生什么错误，结果都不会受到影响。通常情况下，事务的结果被写到持久化存储器中

**Spring支持编程式事务管理和声明式的事务管理。**

### 13.1 编程式事务管理

- 将事务管理代码嵌到业务方法中来控制事务的提交和回滚
- 缺点：必须在每个事务操作业务逻辑中包含额外的事务管理代码

```java
public void createUser() {
    TransactionStatus txStatus =
        transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
      userMapper.insertUser(user);
    } catch (Exception e) {
      transactionManager.rollback(txStatus);
      throw e;
    }
    transactionManager.commit(txStatus);
  }
```

**一般不用编程式事务管理**！

### 13.2 声明式事务管理

**声明式事务管理**

- 一般情况下比编程式事务好用。
- 将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。
- 将事务管理作为横切关注点，通过aop方法模块化。Spring中通过Spring AOP框架支持声明式事务管理。

**使用Spring管理事务，注意头文件的约束导入 : tx**

```xml
xmlns:tx="http://www.springframework.org/schema/tx"

http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx.xsd">
```

**事务管理器**

- 无论使用Spring的哪种事务管理策略（编程式或者声明式）事务管理器都是必须的。

**JDBC事务管理器**

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource" />
</bean>
```

**配置好事务管理器后我们需要去配置事务的通知**

```xml
<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
   <tx:attributes>
       <!--配置哪些方法使用什么样的事务,配置事务的传播特性-->
       <tx:method name="*" propagation="REQUIRED"/>
   </tx:attributes>
</tx:advice>
```

**spring事务传播特性：**

事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。spring支持7种事务传播行为：

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作

Spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。

**配置AOP**

导入aop的头文件，手动或者报错后让idea自动导入！

```xml
<!--配置aop织入事务-->
<aop:config>
   <aop:pointcut id="txPointcut" expression="execution(* com.gao.mapper*.*(..))"/>
   <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
</aop:config>
```

**实体类：**

```java
@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

**UserMapper：**

```java
public interface UserMapper {
    public List<User> getUserList();
    public void addUser(@Param("id")int id, @Param("name")String  name,@Param("pwd") String pwd);
    public void delete(@Param("id") int id);
}
```

**UserMapper.xml:**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.gao.mapper.UserMapper">
    <insert id="addUser">
        insert into mybatis.user (id,name,pwd) values (#{id},#{name},#{pwd})
    </insert>
    <delete id="delete">
        deletes from mybatis.user where id = #{id}
    </delete>
    <select id="getUserList" resultType="User">
        select * from user
    </select>
</mapper>
```

注意：delete中是故意写错的。

**UserMapperImpl:**

```java
public class UserMapperImp extends SqlSessionDaoSupport implements UserMapper{
    @Override
    public List<User> getUserList() {
        addUser(7,"高七","1111111");
        delete(7);
        return getSqlSession().getMapper(UserMapper.class).getUserList();
    }

    @Override
    public void addUser(int id, String name, String pwd) {
        getSqlSession().getMapper(UserMapper.class).addUser(id,name,pwd);
    }

    @Override
    public void delete(int id) {
        getSqlSession().getMapper(UserMapper.class).delete(id);
    }

}
```

```xml
<bean id="UserMapper" class="com.gao.mapper.UserMapperImp">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
</bean>
```

**测试：**

```java
public void test(){
    final ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    final UserMapper userMapper = context.getBean("UserMapper", UserMapper.class);
    for (User user : userMapper.getUserList()) {
        System.out.println(user);
    }
}
```

此时，因为getUserList()被AOP切入事务通知，所以会将其中的所有数据库相关的操作加入一个事务中，此时执行发现：delete()方法报错，同时addUser方法也没有操作数据库。这就是数据库的原子性！

如果删掉AOP的事务通知切入，则addUser方法执行成功，数据库发生了变化，delete方法执行失败。并不符合我们的要求。