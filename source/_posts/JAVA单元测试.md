---
title: JAVA - 单元测试
date: 2021-05-6 11:38:25
tags: 
- JAVA
- 测试
---

# 单元测试

## 1. JUnit测试

### 1.1 TDD

![image-20210422222709239](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210422222709239.png)

这就是传说的TDD。

![image-20210422222726238](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210422222726238.png)

### 1.2 **如何在idea使用Junit？**

1. 右键点击类名，点击`Generate`，选择`test`，点击`OK`。

![image-20210506093726089](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506093726089.png)

![image-20210506094116686](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506094116686.png)

2. 选择要测试的方法（不选也可以，选了会自动生成测试函数）。以及测试类的类名，通常采用默认即可。

![image-20210506094133977](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506094133977.png)

3. 注意到提示`Junit5 library not found in the module`。点击Fix进行修复，勾选Downlad 的选项。

![image-20210506094426316](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506094426316.png)

4. 点击`OK`会自动生成Test类，进去之后发现

![image-20210506095058368](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506095058368.png)

5. 点击Add Library Junit5 to Classpath。

![image-20210506095143021](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506095143021.png)

6. 同时，为了减少注释的复杂度。我们将`org.junit.jupiter.api.Test`导入进来。最终显示如下：

![image-20210506095348637](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506095348637.png)

接下来就可以使用idea进行单元测试了。

验证如下：

![image-20210506095607159](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506095607159.png)

点击左边的绿色三角即可进行测试。

![image-20210506095651944](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506095651944.png)

### 1.3 常用断言

`assertEquals(expected, actual)`是最常用的测试方法。

`Assertion`还定义了其他断言方法，例如：

- ![image-20210506110509206](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210506110509206.png)

**注意**：使用浮点数时，由于浮点数无法精确地进行比较，因此，我们需要调用`assertEquals(double expected, double actual, double delta)`这个重载方法，指定一个误差值：

```java
assertEquals(0.1, Math.abs(1 - 9 / 10.0), 0.0000001);
```

### 1.5 其他断言

`assertThat`：Junit4中有，Junit5中没有。

`assertThat`让开发者可以根据主语，动词，宾语进行断言，增加可读性。

```java
assertThat(x,is(3)); //断言x是3
assertThat(x,isnot(4)); //断言x不是4
assertThat(myList,hasItem("3")); //判断List包含“3”
```

此外，还有：

- allof：所有匹配条件都匹配通过。
- anyOf：任何一个匹配条件匹配则通过。
- not：与条件违背则通过。
- equalTo：使用Object.equals 方法测试对象相等。
- is：等同于euqalTo
- instanceOf，isCompatibleType：测试类型
- notNullValue，nullValue：测试null
- sameInstance：是否同一实例
- hasEntry，hasKey，hasValue：测试一个Map包含entry.key或者value。

## 2. Fixture

JUnit提供了编写测试前准备、测试后清理的固定代码，我们称之为Fixture。

### 2.1 `@BeforeEach`和   `@AfterEach`

我们不必在每个测试方法中都写上初始化代码，而是通过`@BeforeEach`来初始化，通过`@AfterEach`来清理资源：

```java
public class CalculatorTest {

    Calculator calculator;

    @BeforeEach
    public void setUp() {
        this.calculator = new Calculator();
    }

    @AfterEach
    public void tearDown() {
        this.calculator = null;
    }

    @Test
    void testAdd() {
        assertEquals(100, this.calculator.add(100));
        assertEquals(150, this.calculator.add(50));
        assertEquals(130, this.calculator.add(-20));
    }

    @Test
    void testSub() {
        assertEquals(-100, this.calculator.sub(100));
        assertEquals(-150, this.calculator.sub(50));
        assertEquals(-130, this.calculator.sub(-20));
    }
}
```

标记为`@BeforeEach`和`@AfterEach`的方法，它们会在运行每个`@Test`方法前后自动运行。

但是，这样在某些情况下会比较浪费系统资源。

### 2.2 `@BeforeAll`和`@AfterAll`

JUnit还提供了`@BeforeAll`和`@AfterAll`，它们在运行所有@Test前后运行。

因为`@BeforeAll`和`@AfterAll`在所有`@Test`方法运行前后仅运行一次，因此，它们只**能初始化静态变量**，同时**初始化方法也必须是静态方法**，例如：

```java
public class DatabaseTest {
    static Database db;

    @BeforeAll
    public static void initDatabase() {
        db = createDb(...);
    }
    
    @AfterAll
    public static void dropDatabase() {
        ...
    }
}
```

1. 对于实例变量，在`@BeforeEach`中初始化，在`@AfterEach`中清理，它们在各个`@Test`方法中互不影响，因为是不同的实例；
2. 对于静态变量，在`@BeforeAll`中初始化，在`@AfterAll`中清理，它们在各个`@Test`方法中均是唯一实例，会影响各个`@Test`方法。

大多数情况下，使用`@BeforeEach`和`@AfterEach`就足够了。只有某些测试资源初始化耗费时间太长，以至于我们不得不尽量“复用”时才会用到`@BeforeAll`和`@AfterAll`。

## 3. 异常测试

JUnit提供`assertThrows()`来期望捕获一个指定的异常。第二个参数`Executable`封装了我们要执行的会产生异常的代码。当我们执行`Factorial.fact(-1)`时，必定抛出`IllegalArgumentException`。`assertThrows()`在捕获到指定异常时表示通过测试，未捕获到异常，或者捕获到的异常类型不对，均表示测试失败。

```java
@Test
void testNegative() {
    assertThrows(IllegalArgumentException.class, new Executable() {
        @Override
        public void execute() throws Throwable {
            Factorial.fact(-1);
        }
    });
}
```

采用匿名类太繁琐，用Lambda更合适。

```java
@Test
void testNegative() {
    assertThrows(IllegalArgumentException.class, () -> {
        Factorial.fact(-1);
    });
}
```

## 4. 条件测试

### 4.1 `@Disabled`

`@Disabled`的作用是暂时排除某些方法，不让他运行。

测试时的显示结果如下样例：显示跳过。

```bash
Tests run: 68, Failures: 2, Errors: 0, Skipped: 5
```

### 4.2 `@EnableOnOs`

`@EnableOnOs`就是一个条件测试判断。判断操作系统，再选择是否要执行。

```java
@Test
@EnabledOnOs(OS.WINDOWS)
void testWindows() {
    assertEquals("C:\\test.ini", config.getConfigFile("test.ini"));
}

@Test
@EnabledOnOs({ OS.LINUX, OS.MAC })
void testLinuxAndMac() {
    assertEquals("/usr/local/test.cfg", config.getConfigFile("test.cfg"));
}
```

### 4.3 `@DisabledOnOs`

`@DisabledOnOs(OS.WINDOWS)`：表示不再Windows平台执行。

### 4.4 `@DisabledOnJre`

`@DisabledOnJre(JRE.JAVA_8)`：只能在Java 9或更高版本执行的测试

### 4.5 `@EnabledIfSystemProperty`

`@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")`：只能在64位操作系统上执行的测试.

### 4.6 `@EnabledIfEnvironmentVariable`

`@EnabledIfEnvironmentVariable(named = "DEBUG", matches = "true")`：需要传入环境变量`DEBUG=true`才能执行的测试

## 5. 参数化测试

如果待测试的输入和输出是一组数据： 可以把测试数据组织起来 用不同的测试数据调用相同的测试方法。

JUnit提供了一个`@ParameterizedTest`注解，用来进行参数化测试。

### 5.1 方法1：`@MethodSource`

通过`@MethodSource`注解，它允许我们编写一个**同名**的静态方法来提供测试参数：

```java
@ParameterizedTest
@MethodSource
void testCapitalize(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}

static List<Arguments> testCapitalize() {
    return List.of( // arguments:
            Arguments.arguments("abc", "Abc"), //
            Arguments.arguments("APPLE", "Apple"), //
            Arguments.arguments("gooD", "Good"));
}
```

### 5.2 `@CsvSource`和`@CsvFileSource`

```java
@ParameterizedTest
@CsvSource({ "abc, Abc", "APPLE, Apple", "gooD, Good" })
void testCapitalize(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}
```

它的每一个字符串表示一行，一行包含的若干参数用`,`分隔。

如果有成百上千的测试输入，那么，直接写`@CsvSource`就很不方便。这个时候，我们可以把测试数据提到一个独立的CSV文件中，然后标注上`@CsvFileSource`：

```java
@ParameterizedTest
@CsvFileSource(resources = { "/test-capitalize.csv" })
void testCapitalizeUsingCsvFile(String input, String result) {
    assertEquals(result, StringUtils.capitalize(input));
}
```

## 6. 命名规范和测试原则

### 6.1 命名规范

测试的函数命名应当尽可能易读，能够一眼就看明白这个测试的作用。

尽可能使用：given when then 的结构。

例如：should_return_3_when_add_given_input_1_and_2

再函数体中也要标注好given when then。

```java
@Test
public void should_return_3_when_add_given_input_1_and_2(){
    //given
    Calculate calculate = new Calculate();
    
    //when
    int actual = Calculate.add(1,2);
    
    //then
    Assert.assertEquals(actual,3);
}
```

### 6.2 测试原则

**FIRST**

- F：fast
- I：Isolated
- R：repeatable
- S：Self-Validating：测试用例自动告知运行结果，不依赖人工判断
- T：timely：测试先行或者尽快测试，不要拖延。

## 7. 测试套件和MOCK

### 7.1 测试套件

随着项目开展，单元测试类越来越多，实践中需要我们一次性测试许多类。

JUnit使用测试套件解决该问题。

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({CalculateTest.class, CalculateTest1.class})
public class FeatureTest{}
```

### 7.2 MOCK

MOCK用来构造一些不容易构造出来的对象和一些复杂的对象，比如HttpServletRequest必须在Servlet容器才能构造出来。

Mock解决的问题：

1. 解决不同单元耦合而难以测试的问题
2. 通过模拟依赖分解单元测试耦合的部分
3. 验证所调用的依赖

**mock框架**

**mock框架（Mockito 、jmock 、 powermock、EasyMock）**

mock具体内容用到在学。

