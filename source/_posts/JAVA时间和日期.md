---
title: JAVA - 时间和日期
date: 2021-04-22 21:58:25
tags: JAVA
---

# 时间和日期

## 1. 概念

**时区**：

- 以`GMT`或者`UTC`加时区偏移表示，例如：`GMT+08:00`或者`UTC+08:00`表示东八区。
- 缩写，例如，`CST`表示`China Standard Time`，也就是中国标准时间。但是`CST`也可以表示美国中部时间`Central Standard Time USA`，因此，缩写容易产生混淆，我们尽量不要使用缩写。
- 洲／城市表示，例如，`Asia/Shanghai`，表示上海所在地的时区。

**夏令时**：

所谓夏令时，就是夏天开始的时候，把时间往后拨1小时，夏天结束的时候，再把时间往前拨1小时。美国仍在用。

 **计算夏令时请使用标准库提供的相关类，不要试图自己计算夏令时。**

**本地化**

在计算机中，通常使用`Locale`表示一个国家或地区的日期、时间、数字、货币等格式。`Locale`由`语言_国家`的字母缩写构成，例如，**`zh_CN`**表示中文+中国，**`en_US`**表示英文+美国。语言使用小写，国家使用大写。

## 2. Date 和 Calendar

Java标准库提供的API。Java标准库有两套处理日期和时间的API：

- 一套定义在`java.util`这个包里面，主要包括`Date`、`Calendar`和`TimeZone`这几个类；
- 一套新的API是在Java 8引入的，定义在`java.time`这个包里面，主要包括`LocalDateTime`、`ZonedDateTime`、`ZoneId`等。

很多源代码仍在使用旧的API，故仍要学习旧的API

**Date**

`java.util.Date`是用于表示一个日期和时间的对象

```java
public class Main {
    public static void main(String[] args) {
        // 获取当前时间:
        Date date = new Date();
        System.out.println(date.getYear() + 1900); // 必须加上1900
        System.out.println(date.getMonth() + 1); // 0~11，必须加上1
        System.out.println(date.getDate()); // 1~31，不能加1
        // 转换为String:
        System.out.println(date.toString());
        // 转换为GMT时区:
        System.out.println(date.toGMTString());
        // 转换为本地时区:
        System.out.println(date.toLocaleString());
    }
}
```

`SimpleDateFormat`

- yyyy：年
- MM：月
- dd: 日
- HH: 小时
- mm: 分钟
- ss: 秒

```java
public class Main {
    public static void main(String[] args) {
        // 获取当前时间:
        Date date = new Date();
        var sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(date));
    }
}
// 2021-04-21 14:49:09
```

```java
public class Main {
    public static void main(String[] args) {
        // 获取当前时间:
        Date date = new Date();
        var sdf = new SimpleDateFormat("E MMM dd, yyyy");
        System.out.println(sdf.format(date));
    }
}
// Wed Apr 21, 2021
```

一般来说，字母越长，输出越长。以`M`为例，假设当前月份是9月：

- `M`：输出`9`
- `MM`：输出`09`
- `MMM`：输出`Sep`
- `MMMM`：输出`September`

**Calendar**

`Calendar`可以用于获取并设置年、月、日、时、分、秒，它和`Date`比，主要多了一个可以做简单的日期和时间运算的功能。

```java
public class Main {
    public static void main(String[] args) {
        // 获取当前时间:
        Calendar c = Calendar.getInstance();
        int y = c.get(Calendar.YEAR);
        int m = 1 + c.get(Calendar.MONTH);
        int d = c.get(Calendar.DAY_OF_MONTH);
        int w = c.get(Calendar.DAY_OF_WEEK);
        int hh = c.get(Calendar.HOUR_OF_DAY);
        int mm = c.get(Calendar.MINUTE);
        int ss = c.get(Calendar.SECOND);
        int ms = c.get(Calendar.MILLISECOND);
        System.out.println(y + "-" + m + "-" + d + " " + w + " " + hh + ":" + mm + ":" + ss + "." + ms);
    }
}
// 2021-4-21 4 14:50:29.350
```

返回的年份不必转换，返回的月份仍然要加1，返回的星期要特别注意，`1`~`7`分别表示周日，周一，……，周六。

`Calendar`只有一种方式获取，即`Calendar.getInstance()`，而且一获取到就是当前时间。

设置时间：

```java
public class Main {
    public static void main(String[] args) {
        // 当前时间:
        Calendar c = Calendar.getInstance();
        // 清除所有:
        c.clear();    //一开始获取到当前时间，调用clear()删除掉
        // 设置2019年:
        c.set(Calendar.YEAR, 2019);
        // 设置9月:注意8表示9月:
        c.set(Calendar.MONTH, 8);
        // 设置2日:
        c.set(Calendar.DATE, 2);
        // 设置时间:
        c.set(Calendar.HOUR_OF_DAY, 21);
        c.set(Calendar.MINUTE, 22);
        c.set(Calendar.SECOND, 23);
        System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(c.getTime()));
        // 2019-09-02 21:22:23
    }
}
```

**TimeZone**

`Calendar`和`Date`相比，它提供了时区转换的功能。时区用`TimeZone`对象表示：

```java
public class Main {
    public static void main(String[] args) {
        TimeZone tzDefault = TimeZone.getDefault(); // 当前时区
        TimeZone tzGMT9 = TimeZone.getTimeZone("GMT+09:00"); // GMT+9:00时区
        TimeZone tzNY = TimeZone.getTimeZone("America/New_York"); // 纽约时区
        System.out.println(tzDefault.getID()); // Asia/Shanghai
        System.out.println(tzGMT9.getID()); // GMT+09:00
        System.out.println(tzNY.getID()); // America/New_York
    }
}
/*
Etc/UTC
GMT+09:00
America/New_York
*/
```

时区的唯一标识是以字符串表示的ID，我们获取指定`TimeZone`对象也是以这个ID为参数获取。

有了时区，我们就可以对指定时间进行转换。例如，下面的例子演示了如何将北京时间`2019-11-20 8:15:00`转换为纽约时间：

```java
public class Main {
    public static void main(String[] args) {
        // 当前时间:
        Calendar c = Calendar.getInstance();
        // 清除所有:
        c.clear();
        // 设置为北京时区:
        c.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
        // 设置年月日时分秒:
        c.set(2019, 10 /* 11月 */, 20, 8, 15, 0);
        // 显示时间:
        var sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        sdf.setTimeZone(TimeZone.getTimeZone("America/New_York"));
        System.out.println(sdf.format(c.getTime()));
        // 2019-11-19 19:15:00
    }
}
```

利用`Calendar`进行时区转换的步骤是：

1. 清除所有字段；
2. 设定指定时区；
3. 设定日期和时间；
4. 创建`SimpleDateFormat`并设定目标时区；
5. 格式化获取的`Date`对象（注意`Date`对象无时区信息，时区信息存储在`SimpleDateFormat`中）。

`Calendar`也可以对日期和时间进行简单的加减：

```java
public class Main {
    public static void main(String[] args) {
        // 当前时间:
        Calendar c = Calendar.getInstance();
        // 清除所有:
        c.clear();
        // 设置年月日时分秒:
        c.set(2019, 10 /* 11月 */, 20, 8, 15, 0);
        // 加5天并减去2小时:
        c.add(Calendar.DAY_OF_MONTH, 5);
        c.add(Calendar.HOUR_OF_DAY, -2);
        // 显示时间:
        var sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date d = c.getTime();
        System.out.println(sdf.format(d));
        // 2019-11-25 6:15:00
    }
}
```

## 3. LocalDateTime

`java.time`包提供了新的日期和时间API，主要涉及的类型有：

- 本地日期和时间：`LocalDateTime`，`LocalDate`，`LocalTime`；
- 带时区的日期和时间：`ZonedDateTime`；
- 时刻：`Instant`；
- 时区：`ZoneId`，`ZoneOffset`；
- 时间间隔：`Duration`。

以及一套新的用于取代`SimpleDateFormat`的格式化类型`DateTimeFormatter`。

新API的类型几乎全部是不变类型（和String类似），可以放心使用不必担心被修改。

**LocalDateTime**

```java
public class Main {
    public static void main(String[] args) {
        /* 
        LocalDate d = LocalDate.now(); // 当前日期
        LocalTime t = LocalTime.now(); // 当前时间
        LocalDateTime dt = LocalDateTime.now(); // 当前日期和时间
        为了避免程序执行时间的不一样，导致时间不准确，采用下面的方法*/
        LocalDateTime dt = LocalDateTime.now(); // 当前日期和时
        LocalDate d = dt.toLocalDate(); // 转换到当前日期
        LocalTime t = dt.toLocalTime(); // 转换到当前时间
        System.out.println(d); // 严格按照ISO 8601格式打印
        System.out.println(t); // 严格按照ISO 8601格式打印
        System.out.println(dt); // 严格按照ISO 8601格式打印
    }
}
/*
2021-04-22
11:36:59.238843262
2021-04-22T11:36:59.238871627
*/
```

指定的日期和时间创建`LocalDateTime`可以通过`of()`方法：

```java
// 指定日期和时间:
LocalDate d2 = LocalDate.of(2019, 11, 30); // 2019-11-30, 注意11=11月
LocalTime t2 = LocalTime.of(15, 16, 17); // 15:16:17
LocalDateTime dt2 = LocalDateTime.of(2019, 11, 30, 15, 16, 17);
LocalDateTime dt3 = LocalDateTime.of(d2, t2);
```

将字符串转换为`LocalDateTime`就可以传入标准格式：

```java
LocalDateTime dt = LocalDateTime.parse("2019-11-19T15:16:17");
LocalDate d = LocalDate.parse("2019-11-19");
LocalTime t = LocalTime.parse("15:16:17");
```

注意ISO 8601规定的日期和时间分隔符是`T`。标准格式如下：

- 日期：`yyyy-MM-dd`
- 时间：`HH:mm:ss`
- 带毫秒的时间：`HH:mm:ss.SSS`
- 日期和时间：`yyyy-MM-dd'T'HH:mm:ss`
- 带毫秒的日期和时间：`yyyy-MM-dd'T'HH:mm:ss.SSS`

**DateTimeFormatter**

```java
public class Main {
    public static void main(String[] args) {
        // 自定义格式化:
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
        System.out.println(dtf.format(LocalDateTime.now()));

        // 用自定义格式解析:
        LocalDateTime dt2 = LocalDateTime.parse("2019/11/30 15:16:17", dtf);
        System.out.println(dt2);
    }
}
/*
2021/04/22 11:46:18
2019-11-30T15:16:17
*/
```

`LocalDateTime`提供了对日期和时间进行加减的非常简单的链式调用：

```java
public class Main {
    public static void main(String[] args) {
        LocalDateTime dt = LocalDateTime.of(2019, 10, 26, 20, 30, 59);
        System.out.println(dt);
        // 加5天减3小时:
        LocalDateTime dt2 = dt.plusDays(5).minusHours(3);
        System.out.println(dt2); // 2019-10-31T17:30:59
        // 减1月:
        LocalDateTime dt3 = dt2.minusMonths(1);
        System.out.println(dt3); // 2019-09-30T17:30:59
    }
}
/*
2019-10-26T20:30:59
2019-10-31T17:30:59
2019-09-30T17:30:59
月份加减会自动调整日期，例如从2019-10-31减去1个月得到的结果是2019-09-30，因为9月没有31日。
*/
```

对日期和时间进行调整则使用`withXxx()`方法

- 调整年：`withYear()`
- 调整月：`withMonth()`
- 调整日：`withDayOfMonth()`
- 调整时：`withHour()`
- 调整分：`withMinute()`
- 调整秒：`withSecond()`

```java
public class Main {
    public static void main(String[] args) {
        LocalDateTime dt = LocalDateTime.of(2019, 10, 26, 20, 30, 59);
        System.out.println(dt);
        // 日期变为31日:
        LocalDateTime dt2 = dt.withDayOfMonth(31);
        System.out.println(dt2); // 2019-10-31T20:30:59
        // 月份变为9:
        LocalDateTime dt3 = dt2.withMonth(9);
        System.out.println(dt3); // 2019-09-30T20:30:59
    }
}
```

`LocalDateTime`还有一个通用的`with()`方法:

```java
public class Main {
    public static void main(String[] args) {
        // 本月第一天0:00时刻:
        LocalDateTime firstDay = LocalDate.now().withDayOfMonth(1).atStartOfDay();
        System.out.println(firstDay);

        // 本月最后1天:
        LocalDate lastDay = LocalDate.now().with(TemporalAdjusters.lastDayOfMonth());
        System.out.println(lastDay);

        // 下月第1天:
        LocalDate nextMonthFirstDay = LocalDate.now().with(TemporalAdjusters.firstDayOfNextMonth());
        System.out.println(nextMonthFirstDay);

        // 本月第1个周一:
        LocalDate firstWeekday = LocalDate.now().with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
        System.out.println(firstWeekday);
    }
}
```

要判断两个`LocalDateTime`的先后，可以使用`isBefore()`、`isAfter()`方法。

```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime target = LocalDateTime.of(2019, 11, 19, 8, 15, 0);
System.out.println(now.isBefore(target));
System.out.println(LocalDate.now().isBefore(LocalDate.of(2019, 11, 19)));
System.out.println(LocalTime.now().isAfter(LocalTime.parse("08:15:00")));
```

**Duration和Period**

`Duration`表示两个时刻之间的时间间隔。另一个类似的`Period`表示两个日期之间的天数：

```java
public class Main {
    public static void main(String[] args) {
        LocalDateTime start = LocalDateTime.of(2019, 11, 19, 8, 15, 0);
        LocalDateTime end = LocalDateTime.of(2020, 1, 9, 19, 25, 30);
        Duration d = Duration.between(start, end);
        System.out.println(d); // PT1235H10M30S

        Period p = LocalDate.of(2019, 11, 19).until(LocalDate.of(2020, 1, 9));
        System.out.println(p); // P1M21D
    }
}
```

`Duration`和`Period`的表示方法,以`P...T...`的形式表示，`P...T`之间表示日期间隔，`T`后面表示时间间隔。如果是`PT...`的格式表示仅有时间间隔。

利用`ofXxx()`或者`parse()`方法也可以直接创建`Duration`：

```java
Duration d1 = Duration.ofHours(10); // 10 hours
Duration d2 = Duration.parse("P1DT2H3M"); // 1 day, 2 hours, 3 minutes
```

## 4. ZonedDateTime

`LocalDateTime`总是表示本地日期和时间，要表示一个带时区的日期和时间，我们就需要`ZonedDateTime`。

`ZonedDateTime`理解成`LocalDateTime`加`ZoneId`。

```java
public class Main {
    public static void main(String[] args) {
        ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
        ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York")); // 用指定时区获取当前时间
        System.out.println(zbj);
        System.out.println(zny);
    }
}
/*
2021-04-22T13:21:21.244909512Z[Etc/UTC]
2021-04-22T09:21:21.246806905-04:00[America/New_York]
*/
```

通过给一个`LocalDateTime`附加一个`ZoneId`，就可以变成`ZonedDateTime`：

```java
public class Main {
    public static void main(String[] args) {
        LocalDateTime ldt = LocalDateTime.of(2019, 9, 15, 15, 16, 17);
        ZonedDateTime zbj = ldt.atZone(ZoneId.systemDefault());
        ZonedDateTime zny = ldt.atZone(ZoneId.of("America/New_York"));
        System.out.println(zbj);
        System.out.println(zny);
    }
}
/*
2019-09-15T15:16:17Z[Etc/UTC]
2019-09-15T15:16:17-04:00[America/New_York]
*/
```

**时区转换**

要转换时区，首先我们需要有一个`ZonedDateTime`对象，然后，通过`withZoneSameInstant()`将关联时区转换到另一个时区，转换后日期和时间都会相应调整。

将北京时间转换为纽约时间：

```java
public class Main {
    public static void main(String[] args) {
        // 以中国时区获取当前时间:
        ZonedDateTime zbj = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
        // 转换为纽约时间:
        ZonedDateTime zny = zbj.withZoneSameInstant(ZoneId.of("America/New_York"));
        System.out.println(zbj);
        System.out.println(zny);
    }
}
/*
2021-04-22T21:34:20.051086886+08:00[Asia/Shanghai]
2021-04-22T09:34:20.051086886-04:00[America/New_York]
*/
```

## 5. DateTimeFormatter

使用旧的`Date`对象时，我们用`SimpleDateFormat`进行格式化显示。使用新的`LocalDateTime`或`ZonedLocalDateTime`时，我们要进行格式化显示，就要使用`DateTimeFormatter`。

创建`DateTimeFormatter`时，我们仍然通过传入格式化字符串实现：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
```

使用方式与`SimpleDateFormat`完全一致。

另一种创建`DateTimeFormatter`的方法是，传入格式化字符串时，同时指定`Locale`：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("E, yyyy-MMMM-dd HH:mm", Locale.US);
```

```java
public class Main {
    public static void main(String[] args) {
        ZonedDateTime zdt = ZonedDateTime.now();
        var formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm ZZZZ");
        System.out.println(formatter.format(zdt));

        var zhFormatter = DateTimeFormatter.ofPattern("yyyy MMM dd EE HH:mm", Locale.CHINA);
        System.out.println(zhFormatter.format(zdt));

        var usFormatter = DateTimeFormatter.ofPattern("E, MMMM/dd/yyyy HH:mm", Locale.US);
        System.out.println(usFormatter.format(zdt));
    }
}
/*
2021-04-22T13:37 GMT
2021 4月 22 周四 13:37
Thu, April/22/2021 13:37
*/
```

我们可以通过`DateTimeFormatter`预定义的几个静态变量来引用：

```java
var ldt = LocalDateTime.now();
System.out.println(DateTimeFormatter.ISO_DATE.format(ldt));
System.out.println(DateTimeFormatter.ISO_DATE_TIME.format(ldt));
/*
2019-09-15
2019-09-15T23:16:51.56217
*/
```

## 6. Instant

`Instant.now()`获取当前时间戳。

```java
public class Main {
    public static void main(String[] args) {
        Instant now = Instant.now();
        System.out.println(now.getEpochSecond()); // 秒
        System.out.println(now.toEpochMilli()); // 毫秒
    }
}
/*
1619098921
1619098921281
*/
```

`Instant`就是时间戳，那么，给它附加上一个时区，就可以创建出`ZonedDateTime`：

```java
// 以指定时间戳创建Instant:
Instant ins = Instant.ofEpochSecond(1568568760);
ZonedDateTime zdt = ins.atZone(ZoneId.systemDefault());
System.out.println(zdt); // 2019-09-16T01:32:40+08:00[Asia/Shanghai]
```

![image-20210422214742768](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210422214742768.png)

## 7. 转化

如果要把旧式的`Date`或`Calendar`转换为新API对象，可以通过`toInstant()`方法转换为`Instant`对象，再继续转换为`ZonedDateTime`：

```java
// Date -> Instant:
Instant ins1 = new Date().toInstant();

// Calendar -> Instant -> ZonedDateTime:
Calendar calendar = Calendar.getInstance();
Instant ins2 = calendar.toInstant();
ZonedDateTime zdt = ins2.atZone(calendar.getTimeZone().toZoneId());
```

**数据库的Date**

在使用Java程序操作数据库时，我们需要把数据库类型与Java类型映射起来。

| 数据库    | 对应Java类（旧）     | 对应Java类（新） |
| :-------- | :------------------- | :--------------- |
| DATETIME  | `java.util.Date`     | `LocalDateTime`  |
| DATE      | `java.sql.Date`      | `LocalDate`      |
| TIME      | `java.sql.Time`      | `LocalTime`      |
| TIMESTAMP | `java.sql.Timestamp` | `LocalDateTime`  |

```java
public class Main {
    public static void main(String[] args) {
        long ts = 1574208900000L;
        System.out.println(timestampToString(ts, Locale.CHINA, "Asia/Shanghai"));
        System.out.println(timestampToString(ts, Locale.US, "America/New_York"));
    }

    static String timestampToString(long epochMilli, Locale lo, String zoneId) {
        Instant ins = Instant.ofEpochMilli(epochMilli);
        DateTimeFormatter f = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM, FormatStyle.SHORT);
        return f.withLocale(lo).format(ZonedDateTime.ofInstant(ins, ZoneId.of(zoneId)));
    }
}
/*
2019年11月20日 上午8:15
Nov 19, 2019, 7:15 PM
*/
```

