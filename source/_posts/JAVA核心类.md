---
title: JAVA-核心类
date: 2021-04-2 21:19:25
tags: JAVA
---

# JAVA核心类

感谢廖雪峰老师的教程！

[JAVA教程](https://www.liaoxuefeng.com/wiki/1252599548343744)

## 1. 字符串String

```
String s1 = "Hello!";
```

字符串具有**不可变性**。

**字符串比较**

比较字符串必须使用`equals()`，不能使用==。

要忽略大小写比较，使用`equalsIgnoreCase()`方法。

**String常用的方法**

- 搜索子串

```java
"Hello".contains("ll"); // true
"Hello".indexOf("l"); // 2
"Hello".lastIndexOf("l"); // 3
"Hello".startsWith("He"); // true
"Hello".endsWith("lo"); // true
```

- 提取子串

```java
"Hello".substring(2); // "llo"
"Hello".substring(2, 4); "ll"
```

- 去除首位空白字符    `\t \r \n`

```java
"  \tHello\r\n ".trim(); // "Hello"
```

`strip()`与`trim()`不同的是，`strip()`能够移除中文空格，所以立即推，用`strip()`就可以了。

```java
"\u3000Hello\u3000".strip(); // "Hello"
" Hello ".stripLeading(); // "Hello "
" Hello ".stripTrailing(); // " Hello"
```

- 判断是否为空

```java
"".isEmpty(); // true，因为字符串长度为0
"  ".isEmpty(); // false，因为字符串长度不为0
"  \n".isBlank(); // true，因为只包含空白字符
" Hello ".isBlank(); // false，因为包含非空白字符
```

- 替换子串

```java
String s = "hello";
s.replace('l', 'w'); // "hewwo"，所有字符'l'被替换为'w'
s.replace("ll", "~~"); // "he~~o"，所有子串"ll"被替换为"~~"
```

```java
String s = "A,,B;C ,D";
s.replaceAll("[\\,\\;\\s]+", ","); // "A,B,C,D"
```

正则表达式(后面学)替换。*PS：正则表达式忒烦了*

- 分割字符串

```
String s = "A,B,C,D";
String[] ss = s.split("\\,"); // {"A", "B", "C", "D"}
```

- 拼接字符串

```
String[] arr = {"A", "B", "C"};
String s = String.join("***", arr); // "A***B***C"
```

- 格式化字符串

字符串提供了`formatted()`方法和`format()`静态方法，可以传入其他参数，替换占位符，然后生成新的字符串：

```java
public class Main {
    public static void main(String[] args) {
        String s = "Hi %s, your score is %d!";
        System.out.println(s.formatted("Alice", 80));
        System.out.println(String.format("Hi %s, your score is %.2f!", "Bob", 59.5));
    }
}
```

常用的占位符：

1. `%s`：显示字符串；

2. `%d`：显示整数；

3. `%x`：显示十六进制整数；

4. `%f`：显示浮点数。

占位符还可以带格式，例如`%.2f`表示显示两位小数。

- 类型转化 ：使用静态方法valueof

```java
String.valueOf(123); // "123"
String.valueOf(45.67); // "45.67"
String.valueOf(true); // "true"
String.valueOf(new Object()); // 类似java.lang.Object@636be97c
```

要把字符串转换为其他类型，就需要根据情况。例如，把字符串转换为`int`类型：

```java
int n1 = Integer.parseInt("123"); // 123
int n2 = Integer.parseInt("ff", 16); // 按十六进制转换，255
```

```java
boolean b1 = Boolean.parseBoolean("true"); // true
boolean b2 = Boolean.parseBoolean("FALSE"); // false
```

- 转化成`char[]`

```java
char[] cs = "Hello".toCharArray(); // String -> char[]
String s = new String(cs); // char[] -> String
```

## 2. StringBuilder

```java
String s = "";
for (int i = 0; i < 1000; i++) {
    s = s + "," + i;
}
```

`String`每一次拼接字符串都会把旧的扔掉，然后换新的，浪费内存，效率低。

`StringBuilder`解决了这个问题，`StringBuilder`是一个可变对象，可以预分配缓冲区，往`StringBuilder`增加字符时，不会创建新的临时对象。

```java
StringBuilder sb = new StringBuilder(1024);
for (int i = 0; i < 1000; i++) {
    sb.append(',');
    sb.append(i);
}
String s = sb.toString();
```

`StringBuilder`还可以进行链式操作：

```java
public class Main {
    public static void main(String[] args) {
        var sb = new StringBuilder(1024);
        sb.append("Mr ")
          .append("Bob")
          .append("!")
          .insert(0, "Hello, ");
        System.out.println(sb.toString());
    }
}
```

`StringBuilder`有`append`和`insert`的方法。

对于普通的字符串`+`操作，并不需要我们将其改写为`StringBuilder`，因为Java编译器在编译时就自动把多个连续的`+`操作编码为`StringConcatFactory`的操作。在运行期，`StringConcatFactory`会自动把字符串连接操作优化为数组复制或者`StringBuilder`操作。

## 3. StringJoiner

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};
        var sb = new StringBuilder();
        sb.append("Hello ");
        for (String name : names) {
            sb.append(name).append(", ");
        }
        // 注意去掉最后的", ":
        sb.delete(sb.length() - 2, sb.length());
        sb.append("!");
        System.out.println(sb.toString());
    }
}
```

使用`StringBuilder`拼接字符串太复杂了。

为了很好的拼接数组，Java标准库还提供了一个`StringJoiner`

```java
public class Main {
    public static void main(String[] args) {
        String[] names = {"Bob", "Alice", "Grace"};
        var sj = new StringJoiner(", ", "Hello ", "!");
        //为String joiner指定开头和结尾
        for (String name : names) {
            sj.add(name);
        }
        System.out.println(sj.toString());
    }
}
```

`String`还提供了一个静态方法`join()`，这个方法在内部使用了`StringJoiner`来拼接字符串，在不需要指定“开头”和“结尾”的时候，用`String.join()`更方便：

```java
String[] names = {"Bob", "Alice", "Grace"};
var s = String.join(", ", names);
```

## 4. 包装类型

将基础类型：`byte`，`short`，`int`，`long`，`boolean`，`float`，`double`，`char`封装成类。

| 基本类型 | 对应的引用类型      |
| :------- | :------------------ |
| boolean  | java.lang.Boolean   |
| byte     | java.lang.Byte      |
| short    | java.lang.Short     |
| int      | java.lang.Integer   |
| long     | java.lang.Long      |
| float    | java.lang.Float     |
| double   | java.lang.Double    |
| char     | java.lang.Character |

`int`类型转化成`Interger`对象：

- `Integer n1 = new Integer(i);`

  通过构造方法将`int`类型包装成`Interger`对象。

- 通过`Integer`的静态方法`valueOf(i)`将`i`包装成`Interger`对象。

- 通过静态方法valueOf(String)创建Integer实例:

  `Integer n3 = Integer.valueOf("100");`
  

`Interger`对象转化成`int`类型：

```java
int n3 = n1.intValue();    //n1是一个Interger对象
```



**Auto Boxing**

Java编译器可以帮助我们自动在`int`和`Integer`之间转型：

```
Integer n = 100; // 编译器自动使用Integer.valueOf(int)
int x = n; // 编译器自动使用Integer.intValue()
```

这种直接把`int`变为`Integer`的赋值写法，称为**自动装箱（Auto Boxing）**。

反过来，把`Integer`变为`int`的赋值写法，称为**自动拆箱（Auto Unboxing）**。

`Integer`对象之间的比较必须使用`equals()`。

`Integer`类本身还提供了大量方法，例如，最常用的静态方法`parseInt()`可以把字符串解析成一个整数：

```java
int x1 = Integer.parseInt("100"); // 100
int x2 = Integer.parseInt("100", 16); // 256,因为按16进制解析
```

`Integer`还可以把整数格式化为指定进制的字符串：

```java
Integer.toString(100);    //"100",表示为10进制
Integer.toString(100, 36);   // "2s",表示为36进制
Integer.toHexString(100);   // "64",表示为16进制
Integer.toOctalString(100);   // "144",表示为8进制
Integer.toBinaryString(100);   // "1100100",表示为2进制
```

包装类型定义了一些有用的静态变量。

```java
// boolean只有两个值true/false，其包装类型只需要引用Boolean提供的静态字段:
Boolean t = Boolean.TRUE;
Boolean f = Boolean.FALSE;
// int可表示的最大/最小值:
int max = Integer.MAX_VALUE; // 2147483647
int min = Integer.MIN_VALUE; // -2147483648
// long类型占用的bit和byte数量:
int sizeOfLong = Long.SIZE; // 64 (bits)
int bytesOfLong = Long.BYTES; // 8 (bytes)
```

所有的整数和浮点数的包装类型都继承自`Number`，可以非常方便地直接通过包装类型获取各种基本类型：

```java
// 向上转型为Number:
Number num = new Integer(999);
// 获取byte, int, long, float, double:
byte b = num.byteValue();
int n = num.intValue();
long ln = num.longValue();
float f = num.floatValue();
double d = num.doubleValue();
```

处理无符号整型变量

```java
public class Main {
    public static void main(String[] args) {
        byte x = -1;
        byte y = 127;
        System.out.println(Byte.toUnsignedInt(x)); // 255
        System.out.println(Byte.toUnsignedInt(y)); // 127
    }
}
```

## 5. JavaBean

JavaBean是一种符合命名规范的`class`，它通过`getter`和`setter`来定义属性；

属性是一种通用的叫法，并非Java语法规定；

可以利用IDE快速生成`getter`和`setter`；

使用`Introspector.getBeanInfo()`可以获取属性列表。

总结：

1、 所有属性为`private`
2、 提供默认构造方法
3、 提供`getter`和`setter`
4、 实现`serializable`接口

的`class`叫做JavaBean

## 6. 枚举类

```java
enum Weekday {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}
public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        if (day == Weekday.SAT || day == Weekday.SUN) {
            System.out.println("Work at home!");
        } else {
            System.out.println("Work at office!");
        }
    }
}
```

`enum`类型的每个常量在JVM中只有一个唯一实例，所以可以直接用`==`比较：

**常用方法**：

- name()

```java
String s = Weekday.SUN.name(); // "SUN"
```

- ordinal()

```java
int n = Weekday.MON.ordinal(); // 1
```

枚举类可以应用在`switch`语句中。

```java
public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        switch(day) {
        case MON:
        case TUE:
        case WED:
        case THU:
        case FRI:
            System.out.println("Today is " + day + ". Work at office!");
            break;
        case SAT:
        case SUN:
            System.out.println("Today is " + day + ". Work at home!");
            break;
        default:
            throw new RuntimeException("cannot process " + day);
        }
    }
}

enum Weekday {
    MON, TUE, WED, THU, FRI, SAT, SUN;
}
```

Java使用`enum`定义枚举类型，它被编译器编译为`final class Xxx extends Enum { … }`；

通过`name()`获取常量定义的字符串，注意不要使用`toString()`；

通过`ordinal()`返回常量定义的顺序（无实质意义）；

可以为`enum`编写构造方法、字段和方法

`enum`的构造方法要声明为`private`，字段强烈建议声明为`final`；

`enum`适合用在`switch`语句中。

## 7. 记录类  （Java14）

```java
public class Main {
    public static void main(String[] args) {
        Point p = new Point(123, 456);
        System.out.println(p.x());
        System.out.println(p.y());
        System.out.println(p);
    }
}

public record Point(int x, int y) {}
```

使用`record`定义的是不变类

编译器默认按照`record`声明的变量顺序自动创建一个构造方法，并在方法内给字段赋值。那么问题来了，如果我们要检查参数，应该怎么办？

假设`Point`类的`x`、`y`不允许负数，我们就得给`Point`的构造方法加上检查逻辑：

```java
public record Point(int x, int y) {
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException();
        }
    }
}
```

## 8. BigInteger

```java
BigInteger bi = new BigInteger("1234567890");
System.out.println(bi.pow(5)); // 2867971860299718107233761438093672048294900000
```

对`BigInteger`做运算的时候，只能使用实例方法，例如，加法运算：

```java
BigInteger i1 = new BigInteger("1234567890");
BigInteger i2 = new BigInteger("12345678901234567890");
BigInteger sum = i1.add(i2); // 12345678902469135780
```

`BigInteger`和0`Integer`、`Long`一样，也是不可变类，并且也继承自`Number`类。因为`Number`定义了转换为基本类型的几个方法：

- 转换为`byte`：`byteValue()`
- 转换为`short`：`shortValue()`
- 转换为`int`：`intValue()`
- 转换为`long`：`longValue()`
- 转换为`float`：`floatValue()`
- 转换为`double`：`doubleValue()`

如果需要准确地转换成基本类型，可以使用`intValueExact()`、`longValueExact()`等方法，在转换时如果超出范围，将直接抛出`ArithmeticException`异常。

## 9. BigDecimal

和`BigInteger`类似，`BigDecimal`可以表示一个任意大小且精度完全准确的浮点数。

```java
BigDecimal bd = new BigDecimal("123.4567");
System.out.println(bd.multiply(bd)); // 15241.55677489
```

`BigDecimal`用`scale()`表示小数位数，例如：

```java
BigDecimal d1 = new BigDecimal("123.45");
BigDecimal d2 = new BigDecimal("123.4500");
BigDecimal d3 = new BigDecimal("1234500");
System.out.println(d1.scale()); // 2,两位小数
System.out.println(d2.scale()); // 4
System.out.println(d3.scale()); // 0
```

通过`BigDecimal`的`stripTrailingZeros()`方法，可以将一个`BigDecimal`格式化为一个相等的，但去掉了末尾0的`BigDecimal`：

```java
BigDecimal d1 = new BigDecimal("123.4500");
BigDecimal d2 = d1.stripTrailingZeros();
System.out.println(d1.scale()); // 4
System.out.println(d2.scale()); // 2,因为去掉了00

BigDecimal d3 = new BigDecimal("1234500");
BigDecimal d4 = d3.stripTrailingZeros();
System.out.println(d3.scale()); // 0
System.out.println(d4.scale()); // -2
```

如果一个`BigDecimal`的`scale()`返回负数，例如，`-2`，表示这个数是个整数，并且末尾有2个0。

可以对一个`BigDecimal`设置它的`scale`，如果精度比原始值低，那么按照指定的方法进行四舍五入或者直接截断：

```java
BigDecimal d2 = d1.setScale(4, RoundingMode.HALF_UP); // 四舍五入，123.4568
BigDecimal d3 = d1.setScale(4, RoundingMode.DOWN); // 直接截断，123.4567
```

对`BigDecimal`做加、减、乘时，精度不会丢失，但是做除法时，存在无法除尽的情况，这时，就必须指定精度以及如何进行截断：

```java
BigDecimal d1 = new BigDecimal("123.456");
BigDecimal d2 = new BigDecimal("23.456789");
BigDecimal d3 = d1.divide(d2, 10, RoundingMode.HALF_UP); // 保留10位小数并四舍五入
BigDecimal d4 = d1.divide(d2); // 报错：ArithmeticException，因为除不尽
```

还可以对`BigDecimal`做除法的同时求余数：

```java
public class Main {
    public static void main(String[] args) {
        BigDecimal n = new BigDecimal("12.345");
        BigDecimal m = new BigDecimal("0.12");
        BigDecimal[] dr = n.divideAndRemainder(m);
        System.out.println(dr[0]); // 102
        System.out.println(dr[1]); // 0.105
    }
}
```

调用`divideAndRemainder()`方法时，返回的数组包含两个`BigDecimal`，分别是商和余数，其中商总是整数，余数不会大于除数。我们可以利用这个方法判断两个`BigDecimal`是否是整数倍数：

在比较两个`BigDecimal`的值是否相等时，要特别注意，使用`equals()`方法不但要求两个`BigDecimal`的值相等，还要求它们的`scale()`相等：

```java
BigDecimal d1 = new BigDecimal("123.456");
BigDecimal d2 = new BigDecimal("123.45600");
System.out.println(d1.equals(d2)); // false,因为scale不同
System.out.println(d1.equals(d2.stripTrailingZeros())); // true,因为d2去除尾部0后scale变为2
System.out.println(d1.compareTo(d2)); // 0
```

必须使用`compareTo()`方法来比较，它根据两个值的大小分别返回负数、正数和`0`，分别表示小于、大于和等于。

## 10. 工具类

**Math**

求绝对值：

```
Math.abs(-100); // 100
Math.abs(-7.8); // 7.8
```

取最大或最小值：

```
Math.max(100, 99); // 100
Math.min(1.2, 2.3); // 1.2
```

计算xy次方：

```
Math.pow(2, 10); // 2的10次方=1024
```

计算√x：

```
Math.sqrt(2); // 1.414...
```

计算ex次方：

```
Math.exp(2); // 7.389...
```

计算以e为底的对数：

```
Math.log(4); // 1.386...
```

计算以10为底的对数：

```
Math.log10(100); // 2
```

三角函数：

```
Math.sin(3.14); // 0.00159...
Math.cos(3.14); // -0.9999...
Math.tan(3.14); // -0.0015...
Math.asin(1.0); // 1.57079...
Math.acos(1.0); // 0.0
```

Math还提供了几个数学常量：

```
double pi = Math.PI; // 3.14159...
double e = Math.E; // 2.7182818...
Math.sin(Math.PI / 6); // sin(π/6) = 0.5
```

生成一个随机数x，x的范围是`0 <= x < 1`：

```
Math.random(); // 0.53907... 每次都不一样
```

**Random**

`Random`用来创建伪随机数。所谓伪随机数，是指只要给定一个初始的种子，产生的随机数序列是完全一样的。

要生成一个随机数，可以使用`nextInt()`、`nextLong()`、`nextFloat()`、`nextDouble()`：

```
Random r = new Random();
r.nextInt(); // 2071575453,每次都不一样
r.nextInt(10); // 5,生成一个[0,10)之间的int
r.nextLong(); // 8811649292570369305,每次都不一样
r.nextFloat(); // 0.54335...生成一个[0,1)之间的float
r.nextDouble(); // 0.3716...生成一个[0,1)之间的double
```

**SecureRandom**

`SecureRandom`就是用来创建安全的随机数的：

```java
SecureRandom sr = new SecureRandom();
System.out.println(sr.nextInt(100));
```

