---
title: Mybatis框架
date: 2021-06-05 21:13:12
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

## 2. 第一个Mybatis程序

路径：搭建环境-> 导入Mybatis -> 编写代码 ->测试。

### 2.1 创建数据库

建议直接使用navicat。

创建

![image-20210524185853081.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210524185853081.png)

 ### 2.2 新建Maven项目

导入依赖：

```xml
    <dependencies>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
```

在src/main/recources中新建一个配置文件，例如：mybatis-config.xml。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="19980913"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

创建一个工具类，用来连接数据库(其中的内容基本上都是不可变的)：

#### SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 

#### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

#### SqlSession

每个线程都应该有它自己的 SqlSession 实例。

SqlSession 的实例**不是线程安全**的，因此是不能被共享的，所以它的**最佳的作用域是请求或方法作用域**。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，**每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作**，**你应该把这个关闭操作放到 finally 块中。** 下面的示例就是一个确保 SqlSession 关闭的标准模式：

```java
//sqlSessionFactory  --> Session
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try{
            // 获取Mybatis获取sqlSessionFactory对象。
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }
        catch (IOException e){
            e.printStackTrace();
        }
    }

    //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    // SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。

    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

### 2.3 编写代码

- 实体类

  和数据库保持一致。

  ```java
  package com.gao.pojo;
  // 实体类
  public class User {
      private int id;
      private String name;
      private String pwd;
  
      public User() {
      }
  
      public User(int id, String name, String pwd) {
          this.id = id;
          this.name = name;
          this.pwd = pwd;
      }
  
      public int getId() {
          return id;
      }
  
      public void setId(int id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPwd() {
          return pwd;
      }
  
      public void setPwd(String pwd) {
          this.pwd = pwd;
      }
  
      @Override
      public String toString() {
          return "User{" +
                  "id=" + id +
                  ", name='" + name + '\'' +
                  ", pwd='" + pwd + '\'' +
                  '}';
      }
  }
  ```

- Dao接口

  UserDao.java

  ```java
  public interface UserDao {
      List<User> getUserList();
  }
  ```

- 实现接口配置文件

  任意地方：UserMapper.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.gao.dao.UserDao">
      <select id="getUserList"resultType="com.gao.pojo.User">
          <!--查询-->
          select * from mybatis.user
      </select>
  </mapper>
  ```

  - `<mapper namespace="com.gao.dao.UserDao">` 输入对应的接口的全称。

  - `<select id="getUserList"resultType="com.gao.pojo.User">`

    id 是方法名。resultType是返回的类型，后面跟返回的结果的类型全称（即实现类）。select 标签内是查询语句。

### 2.4 测试

```java
public class UserDaoTest {
    @Test
    public void test(){
        //第一步：获取SqlSession对象。
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        //方式一： getMapper
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> userList = userDao.getUserList();

        for (User user :
                userList) {
            System.out.println(user);
        }

        //关闭SqlSession
        sqlSession.close();
    }
}
```

**会出现的错误：**

**错误注意点1**：`org.apache.ibatis.binding.BindingException: Type interface com.gao.dao.UserDao is not known to the MapperRegistry.` 

这是很常见的错误，因为每一个Mapper.xml需要在Mybatis核心配置中注册！

```xml
<mappers>
    <mapper resource="com/gao/dao/UserMapper.xml"></mapper>
</mappers>
```

**错误注意点2**：`Could not find resource com/gao/dao/UserMapper.xml`,提示找不到xml配置文件，这是因为maven没有把xml这个配置文件当作是配置文件，没有出现在编译后的结果内，所以jvm找不到这个文件。

在父项目和子项目的Maven配置文件pom.xml 中添加：

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

**错误注意点3**：`The error may exist in com/gao/dao/UserMapper.xml   Cause: org.apache.ibatis.builder.BuilderException: Error parsing SQL Mapper Configuration. Cause: org.apache.ibatis.builder.BuilderException: Error creating document instance.  Cause: org.xml.sax.SAXParseException; lineNumber: 7; columnNumber: 13; 1 字节的 UTF-8 序列的字节 1 无效。`

这是因为UserMapper.xml中的配置文件写的是UTF-8，改成UTF8即可。

```xml
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.gao.dao.UserDao">
    <select id="getUserList" resultType="com.gao.pojo.User">
        <!--查询-->
        select * from mybatis.user
    </select>
</mapper>
```

**错误注意点4**：`### Error querying database.  Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure`  这是因为不支持SSL协议，将mybatis配置中的useSSL改成false。

```xml
<dataSource type="POOLED">
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
    <property name="username" value="root"/>
    <property name="password" value="19980913"/>
</dataSource>
```

方法二（不推荐使用）

```java
//方式二：
List<User> userList = sqlSession.selectList("com.gao.dao.UserDao.getUserList");
for (User user :
        userList) {
    System.out.println(user);
}
```

因为SqlSession每次都需要关闭，所以应该放在finally里。所以应该改造成:

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

## 3. CRUD

### 3.1 namespace

namespace的包名要和接口名一致。

### 3.2 select

选择查询语句

- id ：对应的namespace中的方法名
- resultType：sql语句执行的返回值
- parameterType：参数类型

例如有参查询：

1. 增加接口函数。

```java
public interface UserMapper {
    //查询全部用户
    List<User> getUserList();

    //根据ID查询
    User getUserByID(int id);
}
```

​	2. 给UserMapper.xml中添加配置。

```xml
<select id="getUserByID" resultType="com.gao.pojo.User" parameterType="int">
    <!--查询-->
    select * from mybatis.user where id = #{id}
</select>
```

 	3. 增加测试

```java
@Test
public void getUserByID(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()) {
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        final User userByID = userMapper.getUserByID(1);
        System.out.println(userByID);
    }
}
```

### 3.3 insert

1. 添加接口函数

```java
public interface UserMapper {
    //查询全部用户
    List<User> getUserList();

    //根据ID查询
    User getUserByID(int id);

    //insert一个用户
    int addUser(User user);
}
```

2. 修改UserMapper.xml配置文件

```xml
<insert id="addUser" parameterType="com.gao.pojo.User">
    insert into mybatis.user (id,name,pwd) values (#{id},#{name},#{pwd})
</insert>
```

id，name，pwd是实体类中定义的名字，所以尽可能定义实体类的属性名字和数据库相同。

3. 测试

```java
public void addUser(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()) {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        int res = mapper.addUser(new User(6, "高六", "147258369"));
        if(res>0){
            System.out.println("插入成功");
        }
        sqlSession.commit();
    }
```

**注意**：执行完一定要提交一定要提交！` sqlSession.commit();`

返回的res是被修改的行数。

### 3.4 update

1. 添加接口函数

```java
public interface UserMapper {
    //查询全部用户
    List<User> getUserList();

    //根据ID查询
    User getUserByID(int id);

    //insert一个用户
    int addUser(User user);

    //修改用户
    int updateUser(User user);
}
```

2. 修改UserMapper.xml配置文件

```xml
<update id="updateUser" parameterType="com.gao.pojo.User">
    update mybatis.user set name = #{name},pwd = #{pwd} where  id = #{id}
</update>
```

3. 测试

```java
public void updateUser(){
    try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
        final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.updateUser(new User(4,"高四四","159357"));
        sqlSession.commit();
    }
}
```

**注意:** update同样要commit。

### 3.5 delete

1. 添加接口函数

```java
//删除用户
int deleteUser(int id);
```

2. 修改配置文件UserMapper.xml

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id = #{id}
</delete>
```

3. 测试

```java
public void deleteUser(){
    try(final SqlSession sqlSession = MybatisUtils.getSqlSession()) {
        final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(6);
        sqlSession.commit();
    }
}
```

同样要commit。

### 3.5 . 万能的Map

假设实体类或者数据库的表的字段过多，应当考虑使用map。

1. 添加接口函数

```java
int addUser2(Map<String,Object> map);
```

2. 修改配置文件UserMapper.xml

```xml
<insert id="addUser2" parameterType="map" >
    insert into mybatis.user (id,name,pwd) values (#{userid},#{userName},#{passWord})
</insert>
```

3. 测试

```java
public void addUser2(){
    try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
        final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<>();
        map.put("userid",6);
        map.put("userName","高六");
        map.put("passWord","123741");

        mapper.addUser2(map);
        sqlSession.commit();
    }
}
```

**注意：** 不能生命同名的接口函数，例如`addUser(Map<String,Object> map)`和`int addUser(User user);`

这种写法不用每次new一个新的实体对象出来，所以不用把所有的字段都列出来然后new一个对象。

map传递参数，直接在sql中去除key即可。

只有一个基本类型参数，可以直接在sql中取。

**多个参数用Map或者注解。**

### 3. 6. 模糊查询like

1. 添加接口函数

```java
List<User> getUserLike(String value);
```

2. 修改配置文件UserMapper.xml

```xml
<select id="getUserLike" parameterType="String" resultType="com.gao.pojo.User">
    select * from mybatis.user where name like #{value}
</select>
```

3. 测试

```java
public void getUserLike(){
    try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
        final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        final List<User> userList = mapper.getUserLike("高五%");
        for (User user :
                userList) {
            System.out.println(user);
        }
    }
}
```

%是通配符，表示0个活多个字符。

注意要写resultType。

模糊查询有两种方式：

- JAVA代码中传递通配符：

```java
List<User> userList = mapper.getUserLike("高五%");
```

- Sql拼接中使用通配符

```sql
select * from mybatis.user where name like "%"#{value}"%"
```

## 4. 配置 

mybatis-config.xml

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 

![image-20210528093457804.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528100704196.png)

重点掌握这几个配置，其他的了解即可。

###  4.1 环境（environment）配置

了解这一点以及配置环境即可：

- **Mybatis默认事务管理器是JDBC，使用连接池，POOLED。**

MyBatis 可以配置成适应多种环境。

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

```xml
<environments default="test">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="19980913"/>
            </dataSource>
        </environment>
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="19980913"/>
            </dataSource>
        </environment>
```

可以配置多套环境，修改`default="test"`即可切换环境。

每个环境下有一些配置，如下：

**事务管理器**

在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

- JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
- MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。

 **如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。**

所以其实用不到MANAGED

**数据源**（连接数据库：dbpc，c3p0，druid）

有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：

**UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢。

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。

### 4.2 属性（properties）配置

我们可以通过properties属性来实现引用配置文件。

属性都是可外部配置且动态替换的，可以在典型的JAVA属性文件中配置，也可以通过properties元素的子元素来传递。【db.properties】

1. 编写数据库配置文件。

![image-20210528100704196.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528093457804.png)

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=UTF-8
username=root
password=19980913
```

2. 在mybatis的配置中引入db.properties配置文件。导入配置文件后就可以将环境中的配置编程动态的，

   `<properties resource="db.properties"/>`

   `<property name="driver" value="${driver}"/>`

   ```xml
   <configuration>
       <properties resource="db.properties"/>
   
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="${driver}"/>
                   <property name="url" value="${url}"/>
                   <property name="username" value="${username}"/>
                   <property name="password" value="${password}"/>
               </dataSource>
           </environment>
       </environments>
   
       <mappers>
           <mapper resource="com/gao/dao/UserMapper.xml"></mapper>
       </mappers>
   </configuration>
   ```

   **注意：properties标签必须放在第一个的位置，否则会报错。**

   同时，也可以在`<properties resource="db.properties">`标签对中添加属性。

   优先使用外部配置文件。

   ```xml
   <properties resource="org/mybatis/example/config.properties">
     <property name="username" value="dev_user"/>
     <property name="password" value="F2Fa3!33TYyg"/>
   </properties>
   ```

   *用处不大，用外部配置还用这个？脱裤子放屁。*

### 4.3  别名（typeAliases）

- 类型别名可为 Java 类型设置一个缩写名字。
- 降低冗余的全限定类名书写

**写法1：**

```xml
<typeAliases>
        <typeAlias type="com.gao.pojo.User" alias="User"></typeAlias>
    </typeAliases>
```

完整如下：（<typeAliases>标签必须放在properties后，xml中有固定的位置，写错了没关系，会有提示的。）

```xml
<configuration>
    <properties resource="db.properties"/>

    <typeAliases>
        <typeAlias type="com.gao.pojo.User" alias="User"></typeAlias>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/gao/dao/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

加入别名后：

```xml
<select id="getUserByID" resultType="User" parameterType="int">
    <!--查询-->
    select * from mybatis.user where id = #{id}
</select>
```

就可以把原来的`com.gao.pojo.User`简写成`User`。

**写法2：**

可以指定一个包名，Mybatis会在包名下搜索Java Bean，扫描实体类的包，他的默认别名是**类名的小写**。

```xml
<typeAliases>
        <package name="com.gao.pojo"/>
</typeAliases>
```

其实大小写都可以：
```xml
<select id="getUserByID" resultType="User" parameterType="int">
    <!--查询-->
    select * from mybatis.user where id = #{id}
</select>
```

实体类多建议使用第二种，实体类比较少使用第一个。

写法二可以使用**注解**来声明别名。

```java
@Alias("author")
public class Author {
    ...
}
```

如果有注解，优先使用注解作为别名。

Java 类型内建的类型别名：

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

### 4.4 设置（settings）

设置很多，具体查询https://mybatis.org/mybatis-3/zh/configuration.html#settings

![image-20210528105032738.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528105032738.png)

![image-20210528105107848.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528112148642.png)

### 4.5 其他设置

很少用。

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)

- [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)

  -  [MyBatis Generator Core](https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core)
  -  [MyBatis Plus](https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter)
  -  等等

### 4.6 映射器（Mappers）

MapperRegistry：注册绑定我们的Mapper文件

**方式1：**【推荐使用，写一个注册一个】

```xml
<mappers>
    <mapper resource="com/gao/dao/UserMapper.xml"></mapper>
</mappers>
```

**方式2**：使用class文件绑定注册

```xml
<mappers>
        <mapper class="com.gao.dao.UserMapper"></mapper>
</mappers>
```

注意点：

- 接口和Mapper配置文件必须同名。
- 接口和Mapper配置文件必须在同一个包下。

**方式3**：使用扫描包进行注入绑定。

```xml
<mappers>
        <package name="com.gao.dao"/>
</mappers>
```

注意点：

- 接口和Mapper配置文件必须同名。
- 接口和Mapper配置文件必须在同一个包下。

### 4.7 生命周期和注册域

![image-20210528111650684.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528105107848.png)

作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的**并发问题**。

**SqlSessionFactoryBuilder**

- 一旦拆功能键SqlSessionFactory，就没用了。
- 局部变量

**SqlSessionFactory**

- 类似于数据库连接池。
- 一旦创建，运行期间一直存在，不要丢弃或重新创建。全局唯一。
- 最佳作用域是应用作用域
- 最简单的就是使用**单例模式**或者静态单例模式。

**SqlSession**

- 连接到连接池的请求！
- 使用完必须关闭，否则资源被占用。
- 最佳作用域是方法内。使用try-resource。
- 一个SqlSession创建多个Mapper。

## 5. 解决属性名和字段名不一致问题

### 5.1 问题

字段名：pwd

![image-20210528112148642.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528111650684.png)

属性名：

![image-20210528112210572.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528120420406.png)

此时如果执行查询，会出现查询结果如下：

`User{id=3, name='高三', password='null'}`

发生原因：

```sql
// 这是原来的查询
select id,name,pwd from mybatis.user where id = #{id}
// 这是现在的查询
select id,name,password from mybatis.user where id = #{id}
```

解决方法：

- 起别名。

  ```xml
  <select id="getUserByID" resultType="user" parameterType="int">
      <!--查询-->
      select id,name,pwd as password from mybatis.user where id = #{id}
  </select>
  ```

  但是该方法明显不合理。

### 5.2 resultMap

结果集映射：

```
id name pwd
id name password
```

```xml
<select id="getUserByID" resultMap="UserMap" parameterType="int">
    <!--查询-->
    select * as password from mybatis.user where id = #{id}
</select>

<resultMap id="UserMap" type="User">
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>
```

resultMap中对应很多result，每个result有两个标签，一个是column，一个是property。

colunm对应的是数据库中的列，property对应的是实体类的属性。

- `resultMap` 元素是 MyBatis 中最重要最强大的元素。

- ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

- 什么不一样转义什么。

  ```xml
  <select id="getUserByID" resultMap="UserMap" parameterType="int">
      <!--查询-->
      select * as password from mybatis.user where id = #{id}
  </select>
  
  <resultMap id="UserMap" type="User">
      <result column="pwd" property="password"/>
  </resultMap>
  ```

## 6. 日志

### 6.1 日志工厂

如果一个数据库操作出现异常，我们需要排错，日志就是最好的助手。

曾经：sout、debug。

![image-20210528120420406.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528121204384.png)

- SLF4J
- LOG4J【掌握】
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
- STDOUT_LOGGING【掌握】
- NO_LOGGING

在Mybatis具体使用哪个日志实现，在设置中设定！

### 6.2 STDOUT_LOGGING标准日志输出

第一步：配置。

```xml
<configuration>
    <properties resource="db.properties"/>
    <settings>
            <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

第二步：打印日志。

添加完设置后再执行测试，会多出来很多输出。

主要关注位置：

![image-20210528121204384.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528112210572.png)

### 6.3 LOG4J

什么是LOG4J？

- Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995)的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件
- 可以控制每一条日志的输出格式
- 定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程
- 通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

1. LOG4J是第三方包，需要导入包。

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

2. 再resources中建立log4j.properties配置文件

```properties
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/kuang.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

3. 配置log4j为日志实现

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

4. 直接测试运行看控制台。

**简单使用：**

1. 再要使用Log4j的类中，导入包  `import org.apache.log4j.Logger`

2. 日志对象，参数为当前类的class

   ```java
   static Logger logger = Logger.getLogger(UserDaoTest.class);
   ```

3. 日志级别

   ```java
   logger.info("info:进入了testLog4j");
   logger.debug("debug:进入了testLog4j");
   logger.error("error:进入了testLog4j");
   ```

## 7. 分页

为什么要分页？

- 减少数据处理量。

### 7.1 使用Limit分页

```sql
SELECT * FROM user limit startIndex,pageSize;
SELECT * FROM user limit 3; #[0,n]
```

使用Mybatis分页：

1. 接口

   ```java
   List<User> getUserByLimit(Map<String,Integer> map);
   ```

2. Mapper.xml

   ```xml
   <select id="getUserByLimit" resultMap="UserMap" parameterType="map">
       select * from user limit #{startIndex},#{pageSize}
   </select>
   
   <resultMap id="UserMap" type="User">
       <result column="id" property="id"/>
       <result column="name" property="name"/>
       <result column="pwd" property="password"/>
   </resultMap>
   ```

3. 测试

   ```java
   public void testGetUserByLimit(){
       try(SqlSession sqlSession = MybatisUtils.getSqlSession()){
           final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           Map<String, Integer> map = new HashMap<>();
           map.put("startIndex",0);
           map.put("pageSize",3);
           final List<User> users = mapper.getUserByLimit(map);
           for (User user :
                   users) {
               System.out.println(user);
           }
       }
   }
   ```

### 7.2 RowBounds分页

**不推荐使用**   

不再使用SQL实现分页。

1. 接口

   ```java
   List<User> getUserByRowBounds();
   ```

2. Mapper.xml

   ```xml
   <select id="getUserByRowBounds" resultMap="UserMap">
       select * from user
   </select>
   ```

3. 测试

   ```java
   public void getUserByRowBounds(){
       SqlSession session = MybatisUtils.getSqlSession();
       // RowBounds 实现
       RowBounds rowBounds = new RowBounds(1,2);
       //通过JAVA代码层实现分页
       List<User> users = session.selectList("com.gao.dao.UserMapper.getUserByRowBounds",null,rowBounds);
       for (User user : users) {
           System.out.println(user);
       }
   }
   ```

### 7.3 PageHelper

https://pagehelper.github.io/docs/howtouse/

了解即可，用到再看。

## 8. 注解开发

本质是反射，底层是动态代理。

1. 注解在接口上实现

   ```java
   public interface UserMapper {
       @Select("select * from user")
       List<User> getUsers();
   }
   ```

2. 在核心配置文件中绑定接口

   ```xml
   <mappers>
       <mapper class="com.gao.dao.UserMapper"/>
   </mappers>
   ```

3. 测试

   ```java
   @Test
   public void test(){
       try(final SqlSession  sqlSession = MybatisUtils.getSqlSession()){
           final UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           final List<User> users = mapper.getUsers();
           for (User user : users) {
               System.out.println(user);
           }
       }
   }
   ```

### 8.1 使用注解CRUD

**自动commit**

在工具类中，在创建sqlSession时，传入true，即可实现自动提交。

原来：

```java
public static SqlSession getSqlSession(){
    return sqlSessionFactory.openSession();
}
```

自动提交：

```java
public static SqlSession getSqlSession(){
    return sqlSessionFactory.openSession(true);
}
```

**查询**

1. 增加接口方法

   ```java
   // 方法存在多个参数，所有参数必须加@Param("id")注解
   @Select("select * from user where id=#{id}")
   User getUserByID(@Param("id") int id2);
   ```

   **注意：@Param("id")表明了使用的参数，以@param注解提供的名称查询。**

2. 测试

   ```java
   UserMapper mapper = sqlSession.getMapper(UserMapper.class);
   User userByID = mapper.getUserByID(2);
   System.out.println(userByID);
   ```

**@param()**

- 基本类型的参数或者String类型，要加上
- 引用类型不用加
- 如果只有一个基本类型，可以忽略，但是建议加
- 在SQL中引用的时@param() 设置的属性名

其他的不写了！

## 9. Lombok

使用步骤：

1. 安装Lombox插件

2. 导入lombox的jar包

3. 在实体类添加注解：注解有如下：

   ```java
   @Getter and @Setter
   @FieldNameConstants
   @ToString
   @EqualsAndHashCode
   @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
   @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
   @Data
   @Builder
   @SuperBuilder
   @Singular
   @Delegate
   @Value
   @Accessors
   @Wither
   @With
   @SneakyThrows
   @val
   @var
   ```

@Data：自动生成无参构造，get，set，tostring，hashcode，equals

@AllArgsConstructor：有参构造

@NoArgsConstructor：无参构造

其他的大多数都很好理解。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

![image-20210528213828582.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210528213828582.png)

## 10. 处理多对一查询

例如：

- 多个学生**关联**一个老师【多对一】

- 一个老师有很多学生，**集合**【一对多】

SQL：

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
INSERT INTO teacher(`id`, `name`) VALUES (1, 秦老师); 
INSERT INTO teacher ( `id`, `name` )
VALUES ( 1, '秦老师' );
CREATE TABLE `student` (
	`id` INT ( 10 ) NOT NULL,
	`name` VARCHAR ( 30 ) DEFAULT NULL,
	`tid` INT ( 10 ) DEFAULT NULL,
	PRIMARY KEY ( `id` ),
	KEY `fktid` ( `tid` ),
	CONSTRAINT `fktid` FOREIGN KEY ( `tid` ) REFERENCES `teacher` ( `id` ) 
) ENGINE = INNODB DEFAULT CHARSET = utf8;
INSERT INTO `student` ( `id`, `name`, `tid` )
VALUES ( 1, '小明', 1 );
INSERT INTO `student` ( `id`, `name`, `tid` )
VALUES ( 2, '小红', 1 );
INSERT INTO `student` ( `id`, `name`, `tid` )
VALUES ( 3, '小张', 1 );
INSERT INTO `student` ( `id`, `name`, `tid` )
VALUES ( 4, '小李', 1 );
INSERT INTO `student` ( `id`, `name`, `tid` )
VALUES ( 5, '小王', 1 );
```

### 10.1 环境搭建

- 导入Lombok

  /pom.xml

  ```xml
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.20</version>
      <!--            <scope>provided</scope>-->
  </dependency>
  ```

- 新建Teacher，Student实体类

  ```java
  package com.gao.pojo;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Student {
      private int id;
      private String name;
      private  Teacher teacher;
  }
  ```

  ```java
  package com.gao.pojo;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Teacher {
      private int id;
      private String name;
  }
  ```

  **注意，在学生实体类中有Teacher对象表示外键**

- 建立Mapper接口

  在dao包下建立TeacherMapper和StudentMapper接口

- 建立Mapper.xml文件

  建议在resources下建立同名包，将Mapper.xml放入其中，要保证文件目录一致。创建包时用`/`分割。

![image-20210529174451565.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210529174451565.png)

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.gao.dao.StudentMapper">
  </mapper>
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.gao.dao.TeacherMapper">
  </mapper>
  ```

- 在核心注册文件注册Mapper接口或文件

  mybatis-config.xml

  ```xml
  <mappers>
      <package name="com.gao.dao"/>
  </mappers>
  ```

  直接使用包导入。

### 10.1 多对一查询

1. 添加接口方法

   ```java
   public interface StudentMapper {
       public List<Student> getStudent();
   }
   ```

2. 修改mapper配置文件。

   **方法1：**使用类似于子查询的方式

   ```xml
   <select id="getStudent" resultMap="StudentTeacher">
           select * from student
       </select>
       <resultMap id="StudentTeacher" type="Student">
           <result property="id" column="id"></result>
   	    <result property="name" column="name"></result>
           <association property="teacher" column="tid" select="getTeacher"></association>
       </resultMap>
       <select id="getTeacher" resultType="Teacher">
           select * from teacher where id=#{tid}
       </select>
   ```

   - 因为要涉及到对象，所以必须使用resultMap
   - resultMap中，如果结果是常量，就选择result即可
   - 因为Student对象中的第三个属性是Teacher对象，而Teacher对象来自于teacher 表，因此需要使用association
   - association的属性中，添加select属性，指向另一个子查询。
   - 子查询的tid随意取名字都可以，mybatis会根据association中的tid自动填充，但是尽量标准填写。

   **方法2：**

   ```xml
   <select id="getStudent" resultMap="StudentTeacher">
       select s.id sid,s.name sname,t.id tid,t.name tname
       from student s,teacher t
       where s.tid=t.id
   </select>
   <resultMap id="StudentTeacher" type="Student">
       <result column="sid" property="id"/>
       <result column="sname" property="name"/>
       <association property="teacher" javaType="Teacher">
           <result property="id" column="tid"></result>
           <result property="name" column="tname"/>
       </association>
   </resultMap>
   ```

   - 同样因为涉及到对象，所以需要用resultMap
   - 第二种写法需要在sql语句中完成连接操作
   - 在resultMap中的每一个result的conlumn对应的是sql语句中的别名
   - association不关联其他的子查询，而是使用javaType绑定对象对应的实体类，并嵌入子标签，子标签中的是Teacher对应的属性和sql查询中对应的列。

## 11. 一对多

1. 添加接口方法

   ```java
   public interface TeacherMapper {
       public List<Teacher> getTeacherList();
   }
   ```

2. 修改Mapper配置

   **方法1**

   ```xml
   <select id="getTeacherList" resultMap="TeacherStudent">
       select t.id tid,t.name tname,s.id sid,s.name sname from teacher t,student s where t.id = s.tid
   </select>
   <resultMap id="TeacherStudent" type="Teacher">
       <result column="tid" property="id"/>
       <result property="name" column="tname"/>
       <collection property="students" ofType="Student">
           <result column="sid" property="id"/>
           <result column="sname" property="name"/>
       </collection>
   </resultMap>
   ```

   - 方法一需要完整的sql语句
   - 因为不是基本类型，所以需要使用resultMap
   - resultMap中的column是sql语句中的别名
   - 因为一个老师对应多个学生，所以在resultMap中的标签要使用collection标签，property是java中的属性，JavaType对应的是属性的类型，ofType对应的是集合中的泛型约束类型。

   **方法2：**

   ```xml
   <select id="getTeacherList" resultMap="TeacherStudent">
       select * from teacher
   </select>
   <resultMap id="TeacherStudent" type="Teacher">
       <collection property="students" ofType="Student" select="getStudent" javaType="ArrayList" column="id"></collection>
   </resultMap>
   <select id="getStudent" resultType="Student">
       select * from student where tid=#{id}
   </select>
   ```

   - 方法二类似于嵌套查询
   - 在select语句中因为teacher查询结果有对象，所以需要使用resultMap
   - resultMap中可以只指定集合，因为属性名字和数据库中的字段名一致，只需要指定collection即可
   - collection中property是对象中的列表属性，用javaType指定property的类型，如果是ArrayList，可以不写。ofType指定泛型中的约束类型，colunm为数据库中的对应外键的列，将这一列传给子查询。用select指定子查询的名称。
   - 子查询用collection提供的id进行查询。where语句中的id可以是任何名称，mybatis会自动优化。

3. 测试

   ```java
   public void selectTeacher(){
       try(final SqlSession sqlSession = MybatisUtils.getSqlSession()) {
           final TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
           final List<Teacher> teacherList = mapper.getTeacherList();
           for (Teacher teacher : teacherList) {
               System.out.println(teacher);
           }
       }
   }
   /*
   Teacher(id=0, name=秦老师, students=[Student(id=1, name=小明, tid=1), Student(id=2, name=小红, tid=1)])
   Teacher(id=0, name=高老师, students=[Student(id=3, name=小张, tid=2), Student(id=4, name=小李, tid=2), Student(id=5, name=小王, tid=2)])
   */
   ```

## 12. 动态SQL

动态SQL就是根据不同的条件生成不同的SQL。有如下几个标签

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

### 12.1 搭建环境

- **数据库中创建表**

```sql
CREATE TABLE `blog`(
`id` VARCHAR(50) NOT NULL COMMENT 博客id,
`title` VARCHAR(100) NOT NULL COMMENT 博客标题,
`author` VARCHAR(30) NOT NULL COMMENT 博客作者,
`create_time` DATETIME NOT NULL COMMENT 创建时间,
`views` INT(30) NOT NULL COMMENT 浏览量
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

- **编写实体类**

```java
import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

**注意**：Date类型要使用util包下的Date类。

- **修改核心配置文件mybatis-config.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.gao.pojo"/>
    </typeAliases>

    <environments default="development">
        ...
    </environments>

    <mappers>
        <package name="com.gao.dao"/>
    </mappers>
    
</configuration>
```

在setting中设置mapUnderscoreToCamelCase为true可自动将数据库中的_分割的字段名对应到JAVA中的驼峰命名法的属性。

- **增加生成ID工具类**

```java
import java.util.UUID;

public class IdUtils {
    static public String getId(){
        return UUID.randomUUID().toString().replaceAll(".","");
    }
}
```

该方法可以随机生成不重复的ID。

- **增加接口类来给数据库添加数**据

```java
public interface BlogMapper {
    int addBlog(Blog blog);
}
```

- **修改Mapper配置文件**

```xml
<mapper namespace="com.gao.dao.BlogMapper">
    <insert id="addBlog" parameterType="Blog">
        insert
        into blog (id, title, author, create_time, views)
        VALUES (#{id}, #{title}, #{author}, #{createTime}, #{views})
    </insert>
</mapper>
```

- **执行并且测试**

```java
public class MyTest {
    @Test
    public void addInitBlog() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
        Blog blog = new Blog();
        blog.setId(IdUtils.getId());
        blog.setTitle("Mybatis如此简单");
        blog.setAuthor("gao");
        blog.setCreateTime(new Date());
        blog.setViews(5201);

        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("Java如此简单");
        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("Spring如此简单");
        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("微服务如此简单");
        mapper.addBlog(blog);
        sqlSession.commit();
        sqlSession.close();
    }
}
```

### 12.2 IF

- 创建接口方法

```java
List<Blog> queryBolgIF(Map map);
```

实现if查询需要传入Map参数。

- 修改Mapper配置文件

```xml
<select id="queryBolgIF" parameterType="map" resultType="Blog">
    select * from blog where 1=1
    <if test="views != null">
        and views = #{views}
    </if>
</select>
```

mybatis通过if语句来完成判断型查询。

1=1是为了让即使所有的判断都失败，仍然能查询出结果而不是报错。

if后面的test属性是必须的属性，进行判断，如果条件成立，则给sql语句追加条件。

- 测试

```java
public void testQueryBolgIF(){
    try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
        final BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
        Map map = new HashMap();
        map.put("views",3000);
        final List<Blog> blogs = mapper.queryBolgIF(map);
        for (Blog blog : blogs) {
            System.out.println(blog);
        }
    }
}
```

创建map，然后向map中添加元素，之后就根据map中的键值对来判断配置文件中的if标签是否成立，成立则追加条件，并且向追加的条件中加入参数。

#### 12.3  trim (where, set)

**where**

where是为了改造原来的mapper配置文件。不用写1=1来保证sql语句完整了。

原来：
```xml
<select id="queryBolgIF" parameterType="map" resultType="Blog">
    select * from blog where 1=1
    <if test="views != null">
        and views = #{views}
    </if>
</select>
```
改造后：

```xml
<select id="queryBolgIF" parameterType="map" resultType="Blog">
    select * from blog
    <where>
      <if test="views != null">
        and views = #{views}
      </if>
    </where>
</select>
```

**set**

set的用途是为了去除无用的逗号。例如在下面的update中每一个if后面都可以加逗号，set标签标识后会自动去除多余的逗号。并且在前面加set。

```xml
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
        where id =#{id}
</update>
```

### 12.4 choose (when, otherwise)

**choose**

choose相当于java中的switch语句。

- 接口方法

```java
List<Blog> queryBolgChoose(Map map);
```

- Mapper配置文件

```xml
<select id="queryBolgChoose" resultType="blog" parameterType="map">
        select * from blog
        <where>
            <choose>
                <when test="title != null">
                    title = #{title}
                </when>
                <when test="views != null">
                    views = #{views}
                </when>
                <otherwise>
                    views = 3000
                </otherwise>
            </choose>
        </where>
    </select>
```

通过choose标签来进行选择，满足条件则给select后追加其中的条件。只能满足一项。

如果when中的条件都不满足，则可以在otherwise中填入其他情况下应当追加的条件。

### 12.4 代码片段

可以用sql标签将重复的片段进行抽出。在需要导入的地方用include导入即可。 	

原来：

```xml
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
        where id =#{id}
</update>
```

利用sql提取后：

```xml
<sql id="if">
    <if test="title != null">
        title = #{title},
    </if>
    <if test="author != null">
        author = #{author},
    </if>
</sql>
<update id="updateBlog" parameterType="map">
    update blog
    <set>
       <include refid="if"></include>
    </set>
        where id =#{id}
</update>
```

### 12.4 Foeach

```sql
select from where id in (1,2,3)
```

写在配置文件中是：
```xml
<select id="getBlogFromID" parameterType="map" resultType="Blog">
    select * from blog where id in (1,2,3)
</select>
```
利用foreach可以改写成。

```xml
<select id="getBlogFromID" parameterType="map" resultType="Blog">
    select * from blog
    <where>
        id in
        <foreach collection="ids" open="(" close=")" separator="," item="id">
            #{id}
        </foreach>
    </where>
</select>
```

open表示以什么开头，close表示以什么结尾，separator表示分隔符。item是传入list的属性是什么。

```java
public void getBlogFromID(){
    try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
        final BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
        final HashMap map = new HashMap();
        final ArrayList<String> integers = new ArrayList<>();
        integers.add("1");
        integers.add("2");
        map.put("ids",integers);
        mapper.getBlogFromID(map);
    }
}
```

通过传入一个list就可以实现将list中的所有元素遍历并且插入sql中。

## 13. 缓存

一次查询的结果，可以放在内存的缓冲区，再次查询相同数据可以直接从缓存中拿。

这样不用频繁访问硬盘，提高查询速度，对于高并发很有帮助。

什么数据用缓存？

读多写少！

### 13.1 Mybatis缓存

Mybatis默认定义了两级缓存

- 默认只有一级开启（sqlsession级别的缓存，也称为本地缓存）
- 二级缓存要手动开启，基于namespce的缓存。（也就是一个Mapper对应一个缓存）
- 为了提高扩展性，Mybatis提供了缓存接口Cache，可以通过实现Cache来定义二级缓存。

### 13.2 一级缓存

一级缓存作用域是sqlsession

首先，设置日志。

执行两次查找：

```java
System.out.println(mapper.getBlogFromID(map));
System.out.println("======================");
System.out.println(mapper.getBlogFromID(map));
```

只有一次查询数据库

![image-20210605195406525.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210605195406525.png)

这就是一级缓存的作用。

### 13.3 二级缓存

如果开启二级缓存，当sqlsession关闭后，会把一级缓存导入二级缓存。

一级缓存作用域太小，用处不大，因此需要二级缓存。

1. 设置开启全局缓存

```xml
<setting name="cacheEnabled" value="true"/>
```

2. 实体类实现序列化

```java
public class Blog implements Serializable {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

3. 在mapper配置文件中开启cache

```xml
<cache/>
```

可以直接加入一个标签表示使用二级cache

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

也可以自定义配置。

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

4. 测试

```java
try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
    final BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    final HashMap map = new HashMap();
    final ArrayList<String> integers = new ArrayList<>();
    integers.add("1");
    integers.add("2");
    map.put("ids",integers);
    System.out.println(mapper.getBlogFromID(map));
}
try(final SqlSession sqlSession = MybatisUtils.getSqlSession()){
    final BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    final HashMap map = new HashMap();
    final ArrayList<String> integers = new ArrayList<>();
    integers.add("1");
    integers.add("2");
    map.put("ids",integers);
    System.out.println(mapper.getBlogFromID(map));
}
```

在两个sqlsession中进行查询同一条数据。

![image-20210605204512709.png](https://ghj1998.oss-cn-beijing.aliyuncs.com/image-20210605204512709.png)

结果显示只查询了一次数据库。

### 13.4 缓存查询顺序

寻找数据，先查找二级缓存，再找一级缓存，再找数据库。

数据库找到后放入一级缓存。

sqlsession关闭后将一级缓存中的数据放入二级缓存。

### 13.5 自定义缓存Ehcache

1. 导入Ehcache依赖
2. 在cache标签中选择type，然后指定新加的cache包。
3. 加入新导入的cache包配置文件。

**现在基本上都用redis，用不着这个**
