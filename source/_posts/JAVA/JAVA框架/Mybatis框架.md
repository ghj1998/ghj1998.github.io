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

## 2. 第一个Mybatis程序

路径：搭建环境-> 导入Mybatis -> 编写代码 ->测试。

### 2.1 创建数据库

建议直接使用navicat。

创建

![image-20210524185853081](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210524185853081.png)

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

## 3 CRUD

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

## 4. 万能的Map

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

## 6. 模糊查询like

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

