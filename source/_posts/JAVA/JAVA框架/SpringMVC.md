---
title: SpringMVC
date: 2021-06-30 11:23:07
tags:
- Spring
- JAVA
- MVC
---

# SpringMVC

## 1. MVC

- MVC是模型(Model)、视图(View)、控制器(Controller)的简写。

- 是将业务逻辑、数据、显示分离的方法来组织代码。

- MVC主要作用是**降低了视图与业务逻辑间的双向偶合**。
- **MVC是一种架构模式**。

**Model（模型）：**数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。

**View（视图）：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

**Controller（控制器）：**接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。也就是说控制器做了个调度员的工作。

## 2. SpringMVC快速入门

Spring MVC是Spring Framework的一部分，是基于Java实现MVC的轻量级Web框架。

[官方文档](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/web.html#spring-web)

Spring的web框架围绕**DispatcherServlet** [ 调度Servlet ] 设计。

DispatcherServlet的作用是将请求分发到不同的处理器。

### 2.1 **DispatcherServlet** 中心控制器

**DispatcherServlet的作用是将请求分发到不同的处理器。**

SpringMVC执行流程如下：

![图片](https://raw.githubusercontent.com/ghj1998/image_repository/main/a.jpg)

![图片](https://raw.githubusercontent.com/ghj1998/image_repository/main/b.jpg)

上图为SpringMVC的一个较完整的流程图，实线表示SpringMVC框架提供的技术，不需要开发者实现，虚线表示需要开发者实现。

**整体的执行流程如下所示：**

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。

   我们假设请求的url为 : http://localhost:8080/SpringMVC/hello

   **如上url拆分成三部分：**

   http://localhost:8080服务器域名

   SpringMVC部署在服务器上的web站点

   hello表示控制器

   通过分析，如上url表示为：请求位于服务器localhost:8080上的SpringMVC站点的hello控制器。

2. HandlerMapping为处理器映射。**DispatcherServlet调用HandlerMapping**,**HandlerMapping根据请求url查找Handler**。

3. **HandlerExecution表示具体的Handler**,其主要作用是**根据url查找控制器**，如上url被查找控制器为：hello。

4. **HandlerExecution将解析后的信息传递给DispatcherServlet,如解析控制器映射等。**

5. **HandlerAdapter表示处理器适配器**，其按照特定的规则去执行Handler。

6. **Handler让具体的Controller执行**。

7. **Controller将具体的执行信息返回**给HandlerAdapter,如**ModelAndView**。

8. **HandlerAdapter**将视图逻辑名或模型**传递给DispatcherServlet**。

9. **DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。**

10. **视图解析器**将解析的逻辑视图名传给DispatcherServlet。

11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。

12. 最终视图呈现给用户。

### 2.2 第一个SpringMVC项目

1. 新建一个**空白的Maven**项目

2. **添加Web框架支持**（会多出来一个web目录，并且上面有个蓝点）

   ![](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630113649057.png)

   ![image-20210630113847782](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630113847782.png)

3. 配置web.xml ，注册DispatcherServlet。

   web.xml在目录web/WEB-INF/web.xml下。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
   
      <!--1.注册DispatcherServlet-->
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc-servlet.xml</param-value>
          </init-param>
          <!--启动级别-1-->
          <load-on-startup>1</load-on-startup>
      </servlet>
   
      <!--/ 匹配所有的请求；（不包括.jsp）-->
      <!--/* 匹配所有的请求；（包括.jsp）-->
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
   
   </web-app>
   ```

4. 编写SpringMVC的配置文件，放在src/main/resources/springmvc-servlet.xml下。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   </beans>
   ```

5. 在springMVC配置文件中注册 处理映射器

   ```xml
   <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
   ```

6. 在springMVC配置文件中注册 处理器适配器

   ```xml
   <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
   ```

7. 在springMVC配置文件中注册视图解析器

   ```xml
   <!--视图解析器:DispatcherServlet给他的ModelAndView-->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
      <!--前缀-->
      <property name="prefix" value="/WEB-INF/jsp/"/>
      <!--后缀-->
      <property name="suffix" value=".jsp"/>
   </bean>
   ```

8. 编写业务：

   编写我们要操作业务Controller ，要么实现Controller接口，要么增加注解；需要返回一个ModelAndView，装数据，封视图；

   放在main/java目录下，建个包放进去。
   
   ```java
   public class HelloController implements Controller {
   
      public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
          //ModelAndView 模型和视图
          ModelAndView mv = new ModelAndView();
   
          //封装对象，放在ModelAndView中。Model
          mv.addObject("msg","HelloSpringMVC!");
          //封装要跳转的视图，放在ModelAndView中
          mv.setViewName("hello"); //: /WEB-INF/jsp/hello.jsp
          return mv;
     }  
   }
   ```
   
9. 在springMVC配置文件中注册HelloController这个bean

   ```java
   <bean id="/hello" class="com.gao.controller.HelloController"/>
   ```

10. 编写jsp页面

    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
       <title>Kuangshen</title>
    </head>
    <body>
    ${msg}
    </body>
    </html>
    ```

11. 配置Tomcat

    ![image-20210630114652889](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630114652889.png)

    ![image-20210630114714186](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630114714186.png)

12. 启动Tomcat，完成测试。

**我们发现，这种开发流程实在是太冗长了，很多步骤可以省略，因此，我们也可以采用注解开发**。

### 2.3 使用注解开发第一个SpringMVC项目

1. 配置web.xml

   ```xml
   
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
   
      <!--1.注册servlet-->
      <servlet>
          <servlet-name>SpringMVC</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!--通过初始化参数指定SpringMVC配置文件的位置，进行关联-->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc-servlet.xml</param-value>
          </init-param>
          <!-- 启动顺序，数字越小，启动越早 -->
          <load-on-startup>1</load-on-startup>
      </servlet>
   
      <!--所有请求都会被springmvc拦截 -->
      <servlet-mapping>
          <servlet-name>SpringMVC</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
   
   </web-app>
   ```

   **/ 和 /\* 的区别：**`< url-pattern > / </ url-pattern >` 不会匹配到.jsp， 只针对我们编写的请求；即：.jsp 不会进入spring的 DispatcherServlet类 。`< url-pattern > /* </ url-pattern >` 会匹配 *.jsp，会出现返回 jsp视图 时再次进入spring的DispatcherServlet 类，导致找不到对应的controller所以报404错。

   **注意：**

   - 注意web.xml版本问题，要最新版！
   - 注册DispatcherServlet
   - 关联SpringMVC的配置文件
   - 启动级别为1
   - 映射路径为 / 【不要用/*，会404】

2. springMVC配置文件

   ```xml
   
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/mvc
          https://www.springframework.org/schema/mvc/spring-mvc.xsd">
   
      <!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
      <context:component-scan base-package="com.kuang.controller"/>
      <!-- 让Spring MVC不处理静态资源 -->
      <mvc:default-servlet-handler />
      <!--
      支持mvc注解驱动
          在spring中一般采用@RequestMapping注解来完成映射关系
          要想使@RequestMapping注解生效
          必须向上下文中注册DefaultAnnotationHandlerMapping
          和一个AnnotationMethodHandlerAdapter实例
          这两个实例分别在类级别和方法级别处理。
          而annotation-driven配置帮助我们自动完成上述两个实例的注入。
       -->
      <mvc:annotation-driven />
   
      <!-- 视图解析器 -->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
            id="internalResourceViewResolver">
          <!-- 前缀 -->
          <property name="prefix" value="/WEB-INF/jsp/" />
          <!-- 后缀 -->
          <property name="suffix" value=".jsp" />
      </bean>
   
   </beans>
   ```

1. 在视图解析器中我们把所有的视图都存放在/WEB-INF/目录下，这样可以保证视图安全，因为这个目录下的文件，客户端不能直接访问。

2. - **让IOC的注解生效**
   - **静态资源过滤 ：HTML . JS . CSS . 图片 ， 视频 .....**
   - **MVC的注解驱动**
   - **配置视图解析器**

3. **创建Controller**

   ```java
   @Controller
   public class HelloController {
   
      //真实访问地址 : 项目名/hello
      @RequestMapping("/hello")
      public String sayHello(Model model){
          //向模型中添加属性msg与值，可以在JSP页面中取出并渲染
          model.addAttribute("msg","hello,SpringMVC");
          //web-inf/jsp/hello.jsp
          return "test";
     }
   }
   ```

   - @Controller是为了让Spring IOC容器初始化时自动扫描到；
   - @RequestMapping是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/hello；
   - 方法中声明Model类型的参数是为了把Action中的数据带到视图中；
   - 方法返回的结果是视图的名称hello，加上配置文件中的前后缀变成WEB-INF/jsp/**test**.jsp。

4. **创建视图层**

   在WEB-INF/ jsp目录中创建test.jsp，视图可以直接取出并展示从Controller带回的信息；

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
      <title>SpringMVC</title>
   </head>
   <body>
   ${msg}
   </body>
   </html>
   ```

5. 打开服务器，访问http://localhost:8080/hello

   ![image-20210630121118223](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630121118223.png)

使用springMVC必须配置的三大件：

**处理器映射器、处理器适配器、视图解析器**

通常，我们只需要**手动配置视图解析器**，而**处理器映射器**和**处理器适配器**只需要开启**注解驱动**即可，而省去了大段的xml配置

**注意：**请求都可以指向一个视图，但是页面结果的结果是不一样的，从这里可以看出**视图是被复用**的，而**控制器与视图之间是弱偶合**关系。

## 3. RestFul

Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

**传统的方法：**

url是这种结构：

http://127.0.0.1/item/queryItem.action?id=1

**RestFul方法：**

http://127.0.0.1/item/1

**学习**

```java
@RequestMapping(value = "/hello/{a}/{b}")
public String test(@PathVariable String a, @PathVariable String b, Model model){
    model.addAttribute("msg",a+b);
    return "hello";
}
```

将控制器的方法改写成上述结构：

之后我们可以通过类似http://localhost:8080/hello/a/bbb的结构来给该控制器中的方法传入参数。

参数需要使用注解修饰。@PathVariable让方法参数的值对应绑定到一个URI模板变量上。

**使用method属性指定请求类型**

用于约束请求的类型，可以收窄请求范围。指定请求谓词的类型如GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE等。

如下：

```java
@RequestMapping(value = "/hello/{a}/{b}",method = RequestMethod.POST)
public String test(@PathVariable String a, @PathVariable String b, Model model){
    model.addAttribute("msg",a+b);
    return "hello";
}
```

由于使用地址栏是get，因此无法正常访问域名了。

![image-20210630153247890](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210630153247890.png)

我们还可以使用方法级别的注解：

```java
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

@GetMapping = @RequestMapping(method = RequestMethod.GET)

## 4. 结果跳转

**ModelAndView**

设置ModelAndView对象 , 根据view的名称 , 和视图解析器跳到指定的页面 .

页面 : {视图解析器前缀} + viewName +{视图解析器后缀}

```xml
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
     id="internalResourceViewResolver">
   <!-- 前缀 -->
   <property name="prefix" value="/WEB-INF/jsp/" />
   <!-- 后缀 -->
   <property name="suffix" value=".jsp" />
</bean>
```

控制器：

```java
public class ControllerTest1 implements Controller {

   public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
       //返回一个模型视图对象
       ModelAndView mv = new ModelAndView();
       mv.addObject("msg","ControllerTest1");
       mv.setViewName("test");
       return mv;
  }
}
```

视图可以直接取出msg渲染到页面。

setViewName设置名字后会由视图解析器进行拼接。 {视图解析器前缀} + viewName +{视图解析器后缀}。

要想访问该资源，需要在springmvc-servlet.xml进行注册。【因为该资源在WEB-INF目录下，不能直接访问。】

```xml
<bean id="/gao" class="com.gao.controller.TestController"></bean>
```

**SpringMVC**

**无视图解析器**

```java
@Controller
public class TestController1 {
    @RequestMapping("/test1")
    public String test1(){
        return "/index.jsp";
    }
    @RequestMapping("/test2")
    public String test1(){
        return "forward:/index.jsp";
    }
    @RequestMapping("/test3")
    public String test2(){
        return "redirect:/index.jsp";
    }
}
```

转发直接return目标或者使用forward:就可以，重定向需要添加redirect:。

**有视图解析器**

```java
@Controller
public class TestController1 {
    @RequestMapping("/test1")
    public String test1(){
        return "hello";
    }
    @RequestMapping("/test2")
    public String test1(){
        return "forward:/index.jsp";
    }
    @RequestMapping("/test3")
    public String test2(){
        return "redirect:/index.jsp";
    }
}
```

转发时需要考虑前后缀的问题或者使用forward或者redirect。

## 5. 数据处理

**1、提交的域名称和处理方法的参数名一致**

http://localhost:8080/hello?name=gao

```java
@RequestMapping("/hello")
public String hello(String name){
   System.out.println(name);
   return "hello";
}
```

**2、提交的域名称和处理方法的参数名不一致**

http://localhost:8080/hello?username=gao

```
//@RequestParam("username") : username提交的域的名称 .
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name){
   System.out.println(name);
   return "hello";
}
```

尽量不管想不想同都加上@RequestParam注解，表示在这是一个从前端传过来的数据。

**3、提交的是一个对象**

要求提交的表单域和对象的属性名一致  , 参数使用对象即可。

```java
public class User {
   private int id;
   private String name;
   private int age;
   //构造
   //get/set
   //tostring()
}
```

http://localhost:8080/mvc04/user?name=gao&id=1&age=15

```java
@RequestMapping("/user")
public String user(User user){
   System.out.println(user);
   return "hello";
}
```

如果使用对象的话，前端传递的参数名和对象名必须一致，否则就是null。

### 6. 数据显示在前端

**ModelAndView**

```java
public class ControllerTest1 implements Controller {

   public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
       //返回一个模型视图对象
       ModelAndView mv = new ModelAndView();
       mv.addObject("msg","ControllerTest1");
       mv.setViewName("test");
       return mv;
  }
}
```

**ModelMap**

```java
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name, ModelMap model){
   //封装要显示到视图中的数据
   //相当于req.setAttribute("name",name);
   model.addAttribute("name",name);
   System.out.println(name);
   return "hello";
}
```

**Model**

```java
@RequestMapping("/ct2/hello")
public String hello(@RequestParam("username") String name, Model model){
   //封装要显示到视图中的数据
   //相当于req.setAttribute("name",name);
   model.addAttribute("msg",name);
   System.out.println(name);
   return "test";
}
```

## 7. 乱码问题

SpringMVC给我们提供了一个过滤器，在web.xml中配置 .

```xml
<filter>
   <filter-name>encoding</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
   <init-param>
       <param-name>encoding</param-name>
       <param-value>utf-8</param-value>
   </init-param>
</filter>
<filter-mapping>
   <filter-name>encoding</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```

乱码问题多见于post方法传递参数中。