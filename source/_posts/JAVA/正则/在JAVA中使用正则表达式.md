---
title: 在JAVA中使用正则表达式
date: 2021-05-06 14:24:55
tags:
- JAVA
- 正则
---

# 在JAVA中使用正则表达式

## 1. 简介

正则表达式可以用字符串来描述规则，并用来匹配字符串。例如，判断手机号，我们用正则表达式`\d{11}`：

```java
boolean isValidMobileNumber(String s) {
    return s.matches("\\d{11}");
}
```

例子：

判断年份是否是20##年。

```java
public class Main {
    public static void main(String[] args) {
        String regex = "20\\d\\d";
        System.out.println("2019".matches(regex)); // true
        System.out.println("2100".matches(regex)); // false
    }
}
```

Java标准库`java.util.regex`内建了正则表达式引擎。

## 2. 匹配规则

| 正则表达式 | 规则                     | 可以匹配                       |
| :--------- | :----------------------- | :----------------------------- |
| `A`        | 指定字符                 | `A`                            |
| `\u548c`   | 指定Unicode字符          | `和`                           |
| `.`        | 任意字符                 | `a`，`b`，`&`，`0`             |
| `\d`       | 数字0~9                  | `0`~`9`                        |
| `\w`       | 大小写字母，数字和下划线 | `a`~`z`，`A`~`Z`，`0`~`9`，`_` |
| `\s`       | 空格、Tab键              | 空格，Tab                      |
| `\D`       | 非数字                   | `a`，`A`，`&`，`_`，……         |
| `\W`       | 非\w                     | `&`，`@`，`中`，……             |
| `\S`       | 非\s                     | `a`，`A`，`&`，`_`，……         |

多个字符的匹配规则如下：

| 正则表达式 | 规则             | 可以匹配                 |
| :--------- | :--------------- | :----------------------- |
| `A*`       | 任意个数字符     | 空，`A`，`AA`，`AAA`，…… |
| `A+`       | 至少1个字符      | `A`，`AA`，`AAA`，……     |
| `A?`       | 0个或1个字符     | 空，`A`                  |
| `A{3}`     | 指定个数字符     | `AAA`                    |
| `A{2,3}`   | 指定范围个数字符 | `AA`，`AAA`              |
| `A{2,}`    | 至少n个字符      | `AA`，`AAA`，`AAAA`，……  |
| `A{0,3}`   | 最多n个字符      | 空，`A`，`AA`，`AAA`     |

注意：在JAVA中`\`也是Java字符串的转义字符，两个`\\`实际上表示的是一个`\`：

例如：

```java
"\\d{3,4}-\\d{7,8}"
// 匹配国内的电话号码规则：3~4位区号加7~8位电话，中间用-连接
```

## 3. 复杂匹配规则

| 正则表达式 | 规则                 | 可以匹配                             |
| :--------- | :------------------- | :----------------------------------- |
| ^          | 开头                 | 字符串开头                           |
| $          | 结尾                 | 字符串结束                           |
| [ABC]      | […]内任意字符        | A，B，C                              |
| [A-F0-9xy] | 指定范围的字符       | `A`，……，`F`，`0`，……，`9`，`x`，`y` |
| [^A-F]     | 指定范围外的任意字符 | 非`A`~`F`                            |
| AB\|CD\|EF | AB或CD或EF           | `AB`，`CD`，`EF`                     |

可以使用小括号来进行匹配顺序的改变。

例如：

匹配字符串`learn java`、`learn php`和`learn go`

```java
public class Main {
    public static void main(String[] args) {
        String re = "learn\\s(java|php|go)";
        System.out.println("learn java".matches(re));
        System.out.println("learn Java".matches(re));
        System.out.println("learn php".matches(re));
        System.out.println("learn Go".matches(re));
    }
}
```

## 4. 分组匹配

如果要分组匹配，第一步修改正则表达式，将要分组的字串用括号括起来。

匹配后，如何按括号提取子串？

我们可以导入`java.util.regex`包，用`Pattern`对象匹配，匹配后获得一个`Matcher`对象，如果匹配成功，就可以直接从`Matcher.group(index)`返回子串：

```java
public class Main {
    public static void main(String[] args) {
        Pattern p = Pattern.compile("(\\d{3,4})\\-(\\d{7,8})");
        Matcher m = p.matcher("010-12345678");
        if (m.matches()) {
            String g1 = m.group(1);
            String g2 = m.group(2);
            System.out.println(g1);
            System.out.println(g2);
        } else {
            System.out.println("匹配失败!");
        }
    }
}
```

正则表达式用`(...)`分组可以通过`Matcher`对象快速提取子串：

- `group(0)`表示匹配的整个字符串；
- `group(1)`表示第1个子串，`group(2)`表示第2个子串，以此类推。

```java
public class Main {
    public static void main(String[] args) {
        Pattern p = Pattern.compile("(\\d{3,4})\\-(\\d{7,8})");
        Matcher m = p.matcher("010-12345678");
        if (m.matches()) {
            String g1 = m.group(1);
            String g2 = m.group(2);
            System.out.println(g1);
            System.out.println(g2);
        } else {
            System.out.println("匹配失败!");
        }
    }
}
```

`String.matches()`方法内部调用的就是`Pattern`和`Matcher`类的方法。

但是，在每一此匹配时都会重新生成`Pattern`对象，效率低。如果涉及到多次匹配，可以使用`Pattern`对象。

例如：

```java
public class Main {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile("(\\d{3,4})\\-(\\d{7,8})");
        pattern.matcher("010-12345678").matches(); // true
        pattern.matcher("021-123456").matches(); // true
        pattern.matcher("022#1234567").matches(); // false
        // 获得Matcher对象:
        Matcher matcher = pattern.matcher("010-12345678");
        if (matcher.matches()) {
            String whole = matcher.group(0); // "010-12345678", 0表示匹配的整个字符串
            String area = matcher.group(1); // "010", 1表示匹配的第1个子串
            String tel = matcher.group(2); // "12345678", 2表示匹配的第2个子串
            System.out.println(area);
            System.out.println(tel);
        }
    }
}
```

## 5. 非贪婪匹配

正则表达式默认使用贪婪匹配：任何一个规则，它总是尽可能多地向后匹配。

例如：`(\d+)(0*)`我们想把数字和后面的0区分开，利用matcher对象提取出前几位的非0数字以及后面的0。

但是由于正则表达式的贪婪性质，所以没办法提取出后面的0。

```java
public class Main {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile("(\\d+)(0*)");
        Matcher matcher = pattern.matcher("1230000");
        if (matcher.matches()) {
            System.out.println("group1=" + matcher.group(1)); // "1230000"
            System.out.println("group2=" + matcher.group(2)); // ""
        }
    }
}
```

要让`\d+`尽量少匹配，让`0*`尽量多匹配，我们就必须让`\d+`使用非贪婪匹配。

给定一个匹配规则，加上`?`后就变成了非贪婪匹配。

```java
public class Main {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile("(\\d+?)(0*)");
        Matcher matcher = pattern.matcher("1230000");
        if (matcher.matches()) {
            System.out.println("group1=" + matcher.group(1)); // "123"
            System.out.println("group2=" + matcher.group(2)); // "0000"
        }
    }
}
/**
group1=123
group2=0000
*/
```

## 6. 搜索和替换

### 6.1 分割字符串

使用正则表达式分割字符串可以实现更加灵活的功能。`String.split()`方法传入的正是正则表达式。我们来看下面的代码：

```java
"a b c".split("\\s"); // { "a", "b", "c" }
"a b  c".split("\\s"); // { "a", "b", "", "c" }
"a, b ;; c".split("[\\,\\;\\s]+"); // { "a", "b", "c" }
```

### 6.2 搜索字符串

```java
public class Main {
    public static void main(String[] args) {
        String s = "the quick brown fox jumps over the lazy dog.";
        Pattern p = Pattern.compile("\\wo\\w");
        Matcher m = p.matcher(s);
        while (m.find()) {
            String sub = s.substring(m.start(), m.end());
            System.out.println(sub);
        }
    }
}
/**
row
fox
dog
*/
```

获取到`Matcher`对象后，反复调用`find()`方法，在整个串中搜索能匹配上`\\wo\\w`规则的子串，并打印出来。

### 6.3 替换字符串

使用正则表达式替换字符串可以直接调用`String.replaceAll()`，它的第一个参数是正则表达式，第二个参数是待替换的字符串。

```java
public class Main {
    public static void main(String[] args) {
        String s = "The     quick\t\t brown   fox  jumps   over the  lazy dog.";
        String r = s.replaceAll("\\s+", " ");
        System.out.println(r); // "The quick brown fox jumps over the lazy dog."
    }
}
```

### 6.4 反向引用

如果我们要把搜索到的指定字符串按规则替换，比如前后各加一个`<b>xxxx</b>`，这个时候，使用`replaceAll()`的时候，我们传入的第二个参数可以使用`$1`、`$2`来反向引用匹配到的子串。例如：

```java
public class Main {
    public static void main(String[] args) {
        String s = "the quick brown fox jumps over the lazy dog.";
        String r = s.replaceAll("\\s([a-z]{4})\\s", " <b>$1</b> ");
        System.out.println(r);
    }
}
/*
the quick brown fox jumps <b>over</b> the <b>lazy</b> dog.
*/
```

它用匹配的分组子串`([a-z]{4})`替换了`$1`。