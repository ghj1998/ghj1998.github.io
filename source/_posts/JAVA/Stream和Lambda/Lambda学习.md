---
title: Lambda学习
date: 2021-05-03 12:11:58
tags: JAVA
---

# Lambda基础

## 1. 函数式编程

函数式编程把函数作为基本运算单元，函数可以作为变量，可以接受函数，可以返回函数。*常把支持函数式编程的风格策划归纳为Lambda表达式*

## 2. Lambda表达式

单方法接口：一个接口只定义了一个方法：

- Comparator
- Runnable
- Callable

例如，调用`Arrays.sort()`时，可以传入一个`Comparator`实例，以前的写法是使用匿名类，如下：

```java
String[] array = ...
Arrays.sort(array, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
```

较为繁琐，JAVA 8 开始引入了Lambda表达式来替换**单方法接口**。

```java
public class Main {
    public static void main(String[] args) {
        String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
        Arrays.sort(array, (s1, s2) -> {
            return s1.compareTo(s2);
        });
        System.out.println(String.join(", ", array));
    }
}
```

```java
(s1, s2) -> {
    return s1.compareTo(s2);
}
```

括号内是参数，参数类型可以忽略。`->{}`表示方法体。

如果只有一行，可以省略大括号以及`return`。

例如：

```java
Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));
```

## 3. FunctionalInterface

只定义了单方法的接口称之为`FunctionalInterface`，用注解`@FunctionalInterface`标记。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

接口中的`static`方法和`default`方法不被`FunctionalInterface`计算在内。

`Object`定义的方法也不算在接口方法内。

例如`Comparator`也是`FunctionalInterface`。

```java
@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    default Comparator<T> thenComparing(Comparator<? super T> other) {
        ...
    }
    ...
}
```

## 4. 方法引用

方法引用：如果某个方法签名和接口恰好一致，就可以直接传入方法引用。

注意：在这里，方法签名只看**参数类型和返回类型**，**不看方法名称，也不看类的继承关系。**

**引用静态方法**

```java
public class Main {
    public static void main(String[] args) {
        String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
        Arrays.sort(array, Main::cmp);
        System.out.println(String.join(", ", array));
    }

    static int cmp(String s1, String s2) {
        return s1.compareTo(s2);
    }
}
```

`Arrays.sort()`中直接传入了静态方法`cmp`的引用，用`Main::cmp`表示。

因为`Comparator<String>`接口定义的方法是`int compare(String, String)`，和静态方法`int cmp(String, String)`相比，除了方法名外，方法参数一致，返回类型相同，因此，我们说两者的方法签名一致，可以直接把方法名作为Lambda表达式传入。

**引用实例方法**

实例方法有一个隐含的`this`参数，`String`类的`compareTo()`方法在实际调用的时候，第一个隐含参数总是传入`this`，相当于静态方法：

```java
public static int compareTo(this, String o);
```

所以引用实例方法可以这么写：

```java
public class Main {
    public static void main(String[] args) {
        String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
        Arrays.sort(array, String::compareTo);
        System.out.println(String.join(", ", array));
    }
}
```

**构造方法引用**

如果要把一个`List<String>`转换为`List<Person>`。

传统的做法是：定义一个`ArrayList<Person>`，然后用`for`循环填充这个`List`：

```java
List<String> names = List.of("Bob", "Alice", "Tim");
List<Person> persons = new ArrayList<>();
for (String name : names) {
    persons.add(new Person(name));
}
```

使用构造方法的话，可以改写成：

```java
public class Main {
    public static void main(String[] args) {
        List<String> names = List.of("Bob", "Alice", "Tim");
        List<Person> persons = names.stream().map(Person::new).collect(Collectors.toList());
        System.out.println(persons);
    }
}

class Person {
    String name;
    public Person(String name) {
        this.name = name;
    }
    public String toString() {
        return "Person:" + this.name;
    }
}
```

> map笔记见：[map学习](https://ghj1998.github.io/2021/05/03/使用Stream)

`map()`需要传入的FunctionalInterface的定义是：

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

把泛型对应上就是方法签名`Person apply(String)`，即传入参数`String`，返回类型`Person`。而`Person`类的构造方法恰好满足这个条件，因为构造方法的参数是`String`，而构造方法虽然没有`return`语句，但它会隐式地返回`this`实例，类型就是`Person`，因此，此处可以引用构造方法。构造方法的引用写法是`类名::new`，因此，此处传入`Person::new`。

