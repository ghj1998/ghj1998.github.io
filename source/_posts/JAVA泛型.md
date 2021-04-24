---
title: JAVA - 泛型
date: 2021-04-21 16:18:25
tags: JAVA
---

# 泛型

## 1. 泛型定义

泛型就是定义一种模板，例如`ArrayList<T>`，然后在代码中为用到的类创建对应的`ArrayList<类型>`：

```java
ArrayList<String> strList = new ArrayList<String>();
```

```java
public class ArrayList<T> {
    private T[] array;
    private int size;
    public void add(T e) {...}
    public void remove(int index) {...}
    public T get(int index) {...}
}
```

在Java标准库中的`ArrayList<T>`实现了`List<T>`接口，它可以向上转型为`List<T>`。

```java
public class ArrayList<T> implements List<T> {
    ...
}

List<String> list = new ArrayList<String>();
```

## 2. 使用泛型

```java
// 无编译器警告:
List<String> list = new ArrayList<String>();
list.add("Hello");
list.add("World");
// 无强制转型:
String first = list.get(0);
String second = list.get(1);
```

编译器看到泛型类型`List<Number>`就可以自动推断出后面的`ArrayList<T>`的泛型类型必须是`ArrayList<Number>`，因此，可以把代码简写为：

```java
// 可以省略后面的Number，编译器可以自动推断泛型类型：
List<Number> list = new ArrayList<>();
```

**泛型接口**

除了`ArrayList<T>`使用了泛型，还可以在接口中使用泛型。例如，`Arrays.sort(Object[])`可以对任意数组进行排序，但待排序的元素必须实现`Comparable<T>`这个泛型接口：

```java
public interface Comparable<T> {
    /**
     * 返回负数: 当前实例比参数o小
     * 返回0: 当前实例与参数o相等
     * 返回正数: 当前实例比参数o大
     */
    int compareTo(T o);
}
```



```java
public class Main {
    public static void main(String[] args) {
        Person[] ps = new Person[] {
            new Person("Bob", 61),
            new Person("Alice", 88),
            new Person("Lily", 75),
        };
        Arrays.sort(ps);
        System.out.println(Arrays.toString(ps));
    }
}
```



```java
class Person implements Comparable<Person> {
    String name;
    int score;
    Person(String name, int score) {
        this.name = name;
        this.score = score;
    }
    public int compareTo(Person other) {
        return this.name.compareTo(other.name);
    }
    public String toString() {
        return this.name + "," + this.score;
    }
}
```

## 3. 编写泛型

写泛型类的步骤：

1. 按照某种类型，例如：`String`，来编写类
2. 标记所有的特定类型
3. 把特定类型`String`替换为`T`，并申明`<T>`

**注意**：泛型类型`<T>`不能用于静态方法。例如：

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() { ... }
    public T getLast() { ... }

    // 对静态方法使用<T>:
    public static Pair<T> create(T first, T last) {
        return new Pair<T>(first, last);
    }
}
```

这样写会报错！

应当这么些：

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() { ... }
    public T getLast() { ... }

    // 静态泛型方法应该使用其他类型区分:
    public static <K> Pair<K> create(K first, K last) {
        return new Pair<K>(first, last);
    }
}
```

因为普通的方法是通过类的实例来调用的，创建实例的过程调用了构造方法，也就是说对象已经知道这个时候类上面定义的<T>的具体类型了；

而静态方法不需要对象实例来调用，所以也就不知道<T>的具体类型，虚拟机不允许这种情况发生，所以编译的时候就报错了。

<K>放在static后面，你可以理解为既然静态方法不知道Pair里面的具体类型，你就手动的告诉它具体的类型。

**多个泛型类型**

```java
public class Pair<T, K> {
    private T first;
    private K last;
    public Pair(T first, K last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() { ... }
    public K getLast() { ... }
}
```

使用的时候，需要指出两种类型：

```java
Pair<String, Integer> p = new Pair<>("test", 123);
```

## 4. 擦拭法

Java语言的泛型实现方式是擦拭法（Type Erasure）。

编译器看到的代码：

```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}
```

虚拟机执行的代码：

```java
public class Pair {
    private Object first;
    private Object last;
    public Pair(Object first, Object last) {
        this.first = first;
        this.last = last;
    }
    public Object getFirst() {
        return first;
    }
    public Object getLast() {
        return last;
    }
}
```

因此，Java使用擦拭法实现泛型，导致了：

- 编译器把类型`<T>`视为`Object`；
- 编译器根据`<T>`实现安全的强制转型。

使用泛型的时候，我们编写的代码也是编译器看到的代码：

```java
Pair<String> p = new Pair<>("Hello", "world");
String first = p.getFirst();
String last = p.getLast();		
```

而虚拟机执行的代码并没有泛型：

```java
Pair p = new Pair("Hello", "world");
String first = (String) p.getFirst();
String last = (String) p.getLast();
```

故缺陷：

擦拭法决定了泛型`<T>`：

- 不能是基本类型，例如：`int`；
- 不能获取带泛型类型的`Class`，例如：`Pair<String>.class`；
  - 无论`T`的类型是什么，`getClass()`返回同一个`Class`实例
- 不能判断带泛型类型的类型，例如：`x instanceof Pair<String>`；
- 不能实例化`T`类型，例如：`new T()`。
- 泛型方法要防止重复定义方法，例如：`public boolean equals(T obj)`；

**泛型继承**

```java
class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}
```

```java
public class IntPair extends Pair<Integer> {
}
```

使用的时候，因为子类`IntPair`并没有泛型类型，所以，正常使用即可：

```java
IntPair ip = new IntPair(1, 2);
```

子类可以获取父类的泛型类型`<T>`。具体很复杂，用到再查。

## 5. extends通配符

```java
public class Main {
    public static void main(String[] args) {
        Pair<Integer> p = new Pair<>(123, 456);
        int n = add(p);
        System.out.println(n);
    }
		//Pair<? extends Number> p
    static int add(Pair<? extends Number> p) {
        Number first = p.getFirst();
        Number last = p.getLast();
        return first.intValue() + last.intValue();
    }
}
class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}
```

使用`Pair<? extends Number>`使得方法接收所有泛型类型为`Number`或`Number`子类的`Pair`类型。

`<? extends Number>`通配符的一个重要限制：方法参数签名`setFirst(? extends Number)`无法传递任何`Number`的子类型给`setFirst(? extends Number)`。

使用类似`<? extends Number>`通配符作为方法参数时表示：

- 方法内部可以调用获取`Number`引用的方法，例如：`Number n = obj.getFirst();`；
- 方法内部无法调用传入`Number`引用的方法（`null`除外），例如：`obj.setFirst(Number n);`。

即一句话总结：**使用`extends`通配符表示可以读，不能写**。

**使用extends限定T类型**

在定义泛型类型`Pair<T>`的时候，也可以使用`extends`通配符来限定`T`的类型：

```
public class Pair<T extends Number> { ... }
```

使用类似`<T extends Number>`定义泛型类时表示：

- 泛型类型限定为`Number`以及`Number`的子类。

## 6. super通配符

```java
void set(Pair<? super Integer> p, Integer first, Integer last) {
    p.setFirst(first);
    p.setLast(last);
}	
```

注意到`Pair<? super Integer>`表示，方法参数接受所有泛型类型为`Integer`或`Integer`父类的`Pair`类型。

使用`<? super Integer>`通配符表示：

- 允许调用`set(? super Integer)`方法传入`Integer`的引用；
- 不允许调用`get()`方法获得`Integer`的引用。

```java
public class Main {
    public static void main(String[] args) {
        Pair<Number> p1 = new Pair<>(12.3, 4.56);
        Pair<Integer> p2 = new Pair<>(123, 456);
        setSame(p1, 100);
        setSame(p2, 200);
        System.out.println(p1.getFirst() + ", " + p1.getLast());
        System.out.println(p2.getFirst() + ", " + p2.getLast());
    }

    static void setSame(Pair<? super Integer> p, Integer n) {
        p.setFirst(n);
        p.setLast(n);
    }

}
```

使用`<? super Integer>`通配符表示：

- 允许调用`set(? super Integer)`方法传入`Integer`的引用；
- 不允许调用`get()`方法获得`Integer`的引用。
  - 唯一例外是可以获取`Object`的引用：`Object o = p.getFirst()`。

使用`<? super Integer>`通配符作为方法参数，表示方法内部代码对于参数**只能写，不能读**。

## 7. 对比extends和super通配符

作为方法参数，`<? extends T>`类型和`<? super T>`类型的区别在于：

- `<? extends T>`允许调用读方法`T get()`获取`T`的引用，但不允许调用写方法`set(T)`传入`T`的引用（传入`null`除外）；
- `<? super T>`允许调用写方法`set(T)`传入`T`的引用，但不允许调用读方法`T get()`获取`T`的引用（获取`Object`除外）。

一个是允许读不允许写，另一个是允许写不允许读。

```java
public class Collections {
    // 把src的每个元素复制到dest中:
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++) {
            T t = src.get(i);
            dest.add(t);
        }
    }
}
```

这个`copy()`方法的定义就完美地展示了`extends`和`super`的意图：

- `copy()`方法内部不会读取`dest`，因为不能调用`dest.get()`来获取`T`的引用；
- `copy()`方法内部也不会修改`src`，因为不能调用`src.add(T)`。

**PECS原则**

Producer Extends Consumer Super。

即：如果需要返回`T`，它是生产者（Producer），要使用`extends`通配符；如果需要写入`T`，它是消费者（Consumer），要使用`super`通配符。

```java
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++) {
            T t = src.get(i); // src是producer
            dest.add(t); // dest是consumer
        }
    }
}
```

需要返回`T`的`src`是生产者，因此声明为`List<? extends T>`，需要写入`T`的`dest`是消费者，因此声明为`List<? super T>`。

**无限定通配符**

无限定通配符：

```java
void sample(Pair<?> p) {
}
```

因为`<?>`通配符既没有`extends`，也没有`super`，因此：

- 不允许调用`set(T)`方法并传入引用（`null`除外）；
- 不允许调用`T get()`方法并获取`T`引用（只能获取`Object`引用）。

**既不能读，也不能写**

`<?>`通配符有一个独特的特点，就是：`Pair<?>`是所有`Pair<T>`的超类：

```java
public class Main {
    public static void main(String[] args) {
        Pair<Integer> p = new Pair<>(123, 456);
        Pair<?> p2 = p; // 安全地向上转型
        System.out.println(p2.getFirst() + ", " + p2.getLast());
    }
}
```

对于`class A<? extends T>：`

因为传入的必定是Pair<T或者T的子类>，所以调用get方法因为返回的必定是T或者T的子类，这样的话就可以用T进行接收，例如：`Number n = p.get()`，因为返回的必定是Number的子类，所以可以向上转型成Number接收。但是无法调用set方法，因为无法向下转型且在运行过程中无法获取T到底是什么类型，你想啊你要是**set(T)**，但实际上**?**(实际的类型)又是**T的子类**，又无法向下转型，那能怎么办，只能报错呗~所以在编译的时候就不会通过。所以无法调用`set`

对于`class A<? super T>:`

因为传入的`Pair<T或者T的父类>`，当调用`set`方法时，`set(T)`，不论T到底是什么，最后都可以向上转型到相应的父类。但是当调用`get`方法时由于不知道返回的T到底是什么类型，其无法向下转型就不能用T接收。所以无法调用`get()`

## 8. 泛型和反射

https://www.liaoxuefeng.com/wiki/1252599548343744/1265105940850016``

`Class<T>`也是泛型，例如：

```java
// compile warning:
Class clazz = String.class;
String str = (String) clazz.newInstance();

// no warning:
Class<String> clazz = String.class;
String str = clazz.newInstance();
```

通过泛型，可以避免类型转化。

调用`Class`的`getSuperclass()`方法返回的`Class`类型是`Class<? super T>`：

```java
Class<? super String> sup = String.class.getSuperclass();
```

构造方法也是泛型。

```java
Class<Integer> clazz = Integer.class;
Constructor<Integer> cons = clazz.getConstructor(int.class);
Integer i = cons.newInstance(123);
```

**定义泛型数组**

```java
Pair<String>[] ps = (Pair<String>[]) new Pair[2];
```

**创建数组T[]**

不能直接创建数组`T[]`，因为擦拭后代码变为`Object[]`：

```java
// compile error:
public class Abc<T> {
    T[] createArray() {
        return new T[5];
    }
}
```

必须借助`Class<T>`来创建数组T[]：

```java
T[] createArray(Class<T> cls) {
    return (T[]) Array.newInstance(cls, 5);
}
```

