---
title: JAVA-多线程
date: 2021-05-09 21:13:25
tags: 
- JAVA
- 多线程
---

# 多线程

## 1. 创建新线程

方法一：从`Thread`派生一个自定义类，然后覆写`run()`方法：

```java
public class Main {
    public static void main(String[] args) {
        Thread t = new MyThread();
        t.start(); // 启动新线程
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
```

方法二：创建`Thread`实例时，传入一个`Runnable`实例：

```java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start(); // 启动新线程
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
```

用lambda语法进一步简写为：

```java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("start new thread!");
        });
        t.start(); // 启动新线程
    }
}
```

两个线程同时运行，由操作系统调度，程序本身无法确定线程的调度顺序。

`Thread.sleep(20);``sleep()`传入的参数是毫秒。调整暂停时间的大小。

必须调用`Thread`实例的`start()`方法才能启动新线程，如果我们查看`Thread`类的源代码，会看到`start()`方法内部调用了一个`private native void start0()`方法，`native`修饰符表示这个方法是由JVM虚拟机内部的C代码实现的，不是由Java代码实现的。

**进程优先级**

可以对线程设定优先级，设定优先级的方法是：

```java
Thread.setPriority(int n) // 1~10, 默认值5
```

## 2. 进程状态

![image-20210423092711785](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210423092711785.png)

- New：新创建的线程，尚未执行；
- Runnable：运行中的线程，正在执行`run()`方法的Java代码；
- Blocked：运行中的线程，因为某些操作被阻塞而挂起；
- Waiting：运行中的线程，因为某些操作在等待中；
- Timed Waiting：运行中的线程，因为执行`sleep()`方法正在计时等待；
- Terminated：线程已终止，因为`run()`方法执行完毕。

一个线程还可以等待另一个线程直到其运行结束。例如，`main`线程在启动`t`线程后，可以通过`t.join()`等待`t`线程结束后再继续运行：

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            System.out.println("hello");
        });
        System.out.println("start");
        t.start();
        t.join();
        System.out.println("end");
    }
}
```

当`main`线程对线程对象`t`调用`join()`方法时，主线程将等待变量`t`表示的线程运行结束，即`join`就是指等待该线程结束，然后才继续往下执行自身线程。

`join(long)`的重载方法也可以指定一个等待时间，超过等待时间后就不再继续等待。

## 3. 中断线程

中断一个线程非常简单，只需要在其他线程中对目标线程调用`interrupt()`方法，目标线程需要反复检测自身状态是否是interrupted状态，如果是，就立刻结束运行。

```JAVA
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new MyThread();
        t.start();
        Thread.sleep(1); // 暂停1毫秒
        t.interrupt(); // 中断t线程
        t.join(); // 等待t线程结束
        System.out.println("end");
    }
}

class MyThread extends Thread {
    public void run() {
        int n = 0;
        while (! isInterrupted()) {
            n ++;
            System.out.println(n + " hello!");
        }
    }
}
```

`main`线程通过调用`t.interrupt()`方法中断`t`线程，但是要注意，`interrupt()`方法仅仅向`t`线程发出了“中断请求”，至于`t`线程是否能立刻响应，要看具体代码。而`t`线程的`while`循环会检测**`isInterrupted()`**，所以上述代码能正确响应`interrupt()`请求，使得自身立刻结束运行`run()`方法。

如果线程处于等待状态，例如，`t.join()`会让`main`线程进入等待状态，此时，如果对`main`线程调用`interrupt()`，`join()`方法会立刻抛出`InterruptedException`。

目标线程只要捕获到`join()`方法抛出的`InterruptedException`，就说明有其他线程对其调用了`interrupt()`方法，通常情况下该线程应该立刻结束运行。

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new MyThread();
        t.start();
        Thread.sleep(1000);
        t.interrupt(); // 中断t线程
        t.join(); // 等待t线程结束
        System.out.println("end");
    }
}

class MyThread extends Thread {
    public void run() {
        Thread hello = new HelloThread();
        hello.start(); // 启动hello线程
        try {
            hello.join(); // 等待hello线程结束
        } catch (InterruptedException e) {
            System.out.println("interrupted!");
        }
        hello.interrupt();
    }
}

class HelloThread extends Thread {
    public void run() {
        int n = 0;
        while (!isInterrupted()) {
            n++;
            System.out.println(n + " hello!");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
}
```

`main`线程通过调用`t.interrupt()`从而通知`t`线程中断，而此时`t`线程正位于`hello.join()`的等待中，此方法会立刻结束等待并抛出`InterruptedException`。由于我们在`t`线程中捕获了`InterruptedException`，因此，就可以准备结束该线程。在`t`线程结束前，对`hello`线程也进行了`interrupt()`调用通知其中断。如果去掉这一行代码，可以发现`hello`线程仍然会继续运行，且JVM不会退出。

**设置标志位中断线程**

另一个常用的中断线程的方法是设置标志位。我们通常会用一个`running`标志位来标识线程是否应该继续运行，在外部线程中，通过把`HelloThread.running`置为`false`，就可以让线程结束：

```java
public class Main {
    public static void main(String[] args)  throws InterruptedException {
        HelloThread t = new HelloThread();
        t.start();
        Thread.sleep(1);
        t.running = false; // 标志位置为false
    }
}

class HelloThread extends Thread {
    public volatile boolean running = true;
    public void run() {
        int n = 0;
        while (running) {
            n ++;
            System.out.println(n + " hello!");
        }
        System.out.println("end!");
    }
}
```

`HelloThread`的标志位`boolean running`是一个线程间共享的变量。

线程间共享变量需要使用`volatile`关键字标记，确保每个线程都能读取到更新后的变量值。

`volatile`关键字解决的是可见性问题：当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值。

## 4. 守护线程

守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。

因此，JVM退出时，不必关心守护线程是否已结束。

如何创建守护线程呢？方法和普通线程一样，只是在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程：

```java
Thread t = new MyThread();
t.setDaemon(true);
t.start();
```

守护线程不能持有任何需要关闭的资源，例如打开文件等！！！！

## 5. 线程同步

如果多个线程同时读写共享变量，会出现数据不一致的问题。

多线程模型下，要保证逻辑正确，对共享变量进行读写时，必须保证一组指令以原子方式执行：即某一个线程执行时，其他线程必须等待：

![image-20210423102653271](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210423102653271.png)

Java程序使用`synchronized`关键字对一个对象进行加锁：

```java
synchronized(lock) {
    n = n + 1;
}
```

如何使用`synchronized`：

1. 找出修改共享变量的线程代码块；
2. 选择一个共享实例作为锁；
3. 使用`synchronized(lockObject) { ... }`。`lockObject`可以是任何一个对象。

例如：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        var add = new AddThread();
        var dec = new DecThread();
        add.start();
        dec.start();
        add.join();
        dec.join();
        System.out.println(Counter.count);
    }
}

class Counter {
    public static final Object lock = new Object();
    public static int count = 0;
}

class AddThread extends Thread {
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter.lock) {
                Counter.count += 1;
            }
        }
    }
}

class DecThread extends Thread {
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter.lock) {
                Counter.count -= 1;
            }
        }
    }
}
```

**不需要synchronized的操作**

JVM规范定义了几种原子操作：

- 基本类型（`long`和`double`除外）赋值，例如：`int n = m`；
- 引用类型赋值，例如：`List<String> list = anotherList`。

对JVM定义的单个原子操作不需要同步。

## 6. 同步方法

让线程自己选择锁对象往往会使得代码逻辑混乱，也不利于封装。更好的方法是把`synchronized`逻辑封装起来。

```java
public class Counter {
    private int count = 0;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }

    public void dec(int n) {
        synchronized(this) {
            count -= n;
        }
    }

    public int get() {
        return count;
    }
}
```

`synchronized`锁住的对象是`this`，即当前实例，这又使得创建多个`Counter`实例的时候，它们之间互不影响，可以并发执行：

```java
var c1 = Counter();
var c2 = Counter();

// 对c1进行操作的线程:
new Thread(() -> {
    c1.add();
}).start();
new Thread(() -> {
    c1.dec();
}).start();

// 对c2进行操作的线程:
new Thread(() -> {
    c2.add();
}).start();
new Thread(() -> {
    c2.dec();
}).start();
```

如果一个类被设计为允许多线程正确访问，我们就说这个类就是“线程安全”的（thread-safe），上面的`Counter`类就是线程安全的。Java标准库的`java.lang.StringBuffer`也是线程安全的。

还有一些不变类，例如`String`，`Integer`，`LocalDate`，它们的所有成员变量都是`final`，多线程同时访问时只能读不能写，这些不变类也是线程安全的。

最后，类似`Math`这些只提供静态方法，没有成员变量的类，也是线程安全的。

除了上述几种少数情况，大部分类，例如`ArrayList`，都是非线程安全的类，我们不能在多线程中修改它们。但是，如果所有线程都只读取，不写入，那么`ArrayList`是可以安全地在线程间共享的。

```java
public void add(int n) {
    synchronized(this) { // 锁住this
        count += n;
    } // 解锁
}
```

等同于：

```java
public synchronized void add(int n) { // 锁住this
    count += n;
} // 解锁
```

对于`static`方法，是没有`this`实例的，对`static`方法添加`synchronized`，锁住的是该类的`Class`实例。

```java
public static void test(int n) {
        synchronized(Counter.class) {
            ...
        }
 }
```

## 7. 死锁

Java的线程锁是可重入的锁。

能被同一个线程反复获取的锁，就叫做可重入锁。

由于Java的线程锁是可重入锁，所以，获取锁的时候，不但要判断是否是第一次获取，还要记录这是第几次获取。每获取一次锁，记录+1，每退出`synchronized`块，记录-1，减到0的时候，才会真正释放锁。

**死锁**

一个线程可以获取一个锁后，再继续获取另一个锁。

```java
public void add(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value += m;
        synchronized(lockB) { // 获得lockB的锁
            this.another += m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}

public void dec(int m) {
    synchronized(lockB) { // 获得lockB的锁
        this.another -= m;
        synchronized(lockA) { // 获得lockA的锁
            this.value -= m;
        } // 释放lockA的锁
    } // 释放lockB的锁
}
```

死锁发生后，没有任何机制能解除死锁，只能强制结束JVM进程。在编写多线程应用时，要特别注意防止死锁。

如何避免死锁呢？

线程获取锁的顺序要一致。即严格按照先获取`lockA`，再获取`lockB`的顺序，改写`dec()`方法如下：

```java
public void dec(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value -= m;
        synchronized(lockB) { // 获得lockB的锁
            this.another -= m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}
```

## 8. wait和notify

多线程协调运行的原则就是：当条件不满足时，线程进入等待状态；当条件满足时，线程被唤醒，继续执行任务。

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
    }

    public synchronized String getTask() {
        while (queue.isEmpty()) {
        }
        return queue.remove();
    }
}
```

`while()`循环永远不会退出。因为线程在执行`while()`循环时，已经在`getTask()`入口获取了`this`锁，其他线程根本无法调用`addTask()`，因为`addTask()`执行条件也是获取`this`锁。

改造：

```java
public synchronized String getTask() {
    while (queue.isEmpty()) {
        this.wait();
    }
    return queue.remove();
}
```

当一个线程执行到`getTask()`方法内部的`while`循环时，它必定已经获取到了`this`锁，此时，线程执行`while`条件判断，如果条件成立（队列为空），线程将执行`this.wait()`，进入等待状态。

这里的关键是：`wait()`方法必须在当前获取的锁对象上调用，这里获取的是`this`锁，因此调用`this.wait()`。

`wait()`方法调用时，会*释放*线程获得的锁，`wait()`方法返回后，线程又会重新试图获得锁。

如何让等待的线程被重新唤醒，然后从`wait()`方法返回？

在相同的锁对象上调用`notify()`方法。我们修改`addTask()`如下：

```java
public synchronized void addTask(String s) {
    this.queue.add(s);
    this.notify(); // 唤醒在this锁等待的线程
}
```

在往队列中添加了任务后，线程立刻对`this`锁对象调用`notify()`方法，这个方法会唤醒一个正在`this`锁等待的线程（就是在`getTask()`中位于`this.wait()`的线程），从而使得等待线程从`this.wait()`方法返回。

完整例子：

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        var q = new TaskQueue();
        var ts = new ArrayList<Thread>();
        for (int i=0; i<5; i++) {
            var t = new Thread() {
                public void run() {
                    // 执行task:
                    while (true) {
                        try {
                            String s = q.getTask();
                            System.out.println("execute task: " + s);
                        } catch (InterruptedException e) {
                            return;
                        }
                    }
                }
            };
            t.start();
            ts.add(t);
        }
        var add = new Thread(() -> {
            for (int i=0; i<10; i++) {
                // 放入task:
                String s = "t-" + Math.random();
                System.out.println("add task: " + s);
                q.addTask(s);
                try { Thread.sleep(100); } catch(InterruptedException e) {}
            }
        });
        add.start();
        add.join();
        Thread.sleep(100);
        for (var t : ts) {
            t.interrupt();
        }
    }
}

class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
        this.notifyAll();
    }

    public synchronized String getTask() throws InterruptedException {
        while (queue.isEmpty()) {
            this.wait();
        }
        return queue.remove();
    }
}
```

`this.notifyAll()`而不是`this.notify()`，使用`notifyAll()`将唤醒所有当前正在`this`锁等待的线程，而`notify()`只会唤醒其中一个，这是因为可能有多个线程正在`getTask()`方法内部的`wait()`中等待，使用`notifyAll()`将一次性全部唤醒。通常来说，`notifyAll()`更安全。

**`while()`循环中调用`wait()**

## 9. ReentrantLock

`java.util.concurrent.locks`包提供的`ReentrantLock`用于替代`synchronized`加锁

```java
public class Counter {
    private int count;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }
    }
}
```

等同于：

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```

`ReentrantLock`是Java代码实现的锁，我们就必须先获取锁，然后在`finally`中正确释放锁。

`ReentrantLock`是可重入锁，它和`synchronized`一样，一个线程可以多次获取同一个锁。

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

上述代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，`tryLock()`返回`false`，程序就可以做一些额外处理，而不是无限等待下去。

所以，使用`ReentrantLock`比直接使用`synchronized`更安全，线程在`tryLock()`失败的时候不会导致死锁。

## 10. Condition

使用`Condition`对象来实现`wait`和`notify`的功能。

把前面用`synchronized`实现的功能通过`ReentrantLock`和`Condition`来实现：

```java
class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```

使用`Condition`时，引用的`Condition`对象必须从`Lock`实例的`newCondition()`返回，这样才能获得一个绑定了`Lock`实例的`Condition`实例。

`Condition`提供的`await()`、`signal()`、`signalAll()`原理和`synchronized`锁对象的`wait()`、`notify()`、`notifyAll()`是一致的，并且其行为也是一样的：

- `await()`会释放当前锁，进入等待状态；
- `signal()`会唤醒某个等待线程；
- `signalAll()`会唤醒所有等待线程；
- 唤醒线程从`await()`返回后需要重新获得锁。

`await()`可以在等待指定时间后，如果还没有被其他线程通过`signal()`或`signalAll()`唤醒，可以自己醒来：

```java
if (condition.await(1, TimeUnit.SECOND)) {
    // 被其他线程唤醒
} else {
    // 指定时间内没有被其他线程唤醒
}
```

## 11. ReadWriteLock

`ReadWriteLock`可以解决这个问题，它保证：

- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。

```java
public class Counter {
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();
    private final Lock wlock = rwlock.writeLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        wlock.lock(); // 加写锁
        try {
            counts[index] += 1;
        } finally {
            wlock.unlock(); // 释放写锁
        }
    }

    public int[] get() {
        rlock.lock(); // 加读锁
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            rlock.unlock(); // 释放读锁
        }
    }
}	
```

## 12. StampedLock

新的读写锁：`StampedLock`。`StampedLock`和`ReadWriteLock`相比，改进之处在于：读的过程中也允许获取写锁后写入！

乐观锁的意思就是乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁。反过来，悲观锁则是读的过程中拒绝有写入，也就是写入必须等待。显然乐观锁的并发效率更高，但一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。

```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            stamp = stampedLock.readLock(); // 获取一个悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

## 13. 使用Concurrent集合

`BlockingQueue`：当一个线程调用这个`TaskQueue`的`getTask()`方法时，该方法内部可能会让线程变成等待状态，直到队列条件满足不为空，线程被唤醒后，`getTask()`方法才会返回。

Java标准库的`java.util.concurrent`包提供线程安全的集合：

| interface | non-thread-safe           | thread-safe                                |
| :-------- | :------------------------ | :----------------------------------------- |
| List      | `ArrayList`               | `CopyOnWriteArrayList`                     |
| Map       | `HashMap`                 | `ConcurrentHashMap`                        |
| Set       | `HashSet / TreeSet`       | `CopyOnWriteArraySet`                      |
| Queue     | `ArrayDeque / LinkedList` | `ArrayBlockingQueue / LinkedBlockingQueue` |
| Deque     | `ArrayDeque / LinkedList` | `LinkedBlockingDeque`                      |

使用方法完全相同。

`java.util.Collections`工具类还提供了一个旧的线程安全集合转换器，可以这么用：

```java
Map unsafeMap = new HashMap();
Map threadSafeMap = Collections.synchronizedMap(unsafeMap);
```

**但是不推荐，因为效率低**

## 14. Atomic

Java的`java.util.concurrent`包除了提供底层锁、并发集合外，还提供了一组原子操作的封装类，它们位于`java.util.concurrent.atomic`包。

我们以`AtomicInteger`为例，它提供的主要操作有：

- 增加值并返回新值：`int addAndGet(int delta)`
- 加1后返回新值：`int incrementAndGet()`
- 获取当前值：`int get()`
- 用CAS方式设置：`int compareAndSet(int expect, int update)`

CAS ：Compare and Set。

实现方法如下：

```java
public int incrementAndGet(AtomicInteger var) {
    int prev, next;
    do {
        prev = var.get();
        next = prev + 1;
    } while ( ! var.compareAndSet(prev, next));
    return next;
}
```

CAS是指，在这个操作中，如果`AtomicInteger`的当前值是`prev`，那么就更新为`next`，返回`true`。如果`AtomicInteger`的当前值不是`prev`，就什么也不干，返回`false`。通过CAS操作并配合`do ... while`循环，即使其他线程修改了`AtomicInteger`的值，最终的结果也是正确的。

我们并不需要直接用`do ... while`循环调用`compareAndSet`实现复杂的并发操作，而是用`incrementAndGet()`这样的封装好的方法，因此，使用起来非常简单。

在高度竞争的情况下，还可以使用Java 8提供的`LongAdder`和`LongAccumulator`。

## 15. 线程池

### 15.1 普通线程池

程池内部维护了若干个线程，没有任务的时候，这些线程都处于等待状态。如果有新任务，就分配一个空闲线程执行。如果所有线程都处于忙碌状态，新任务要么放入队列等待，要么增加一个新线程进行处理。

Java标准库提供了`ExecutorService`接口表示线程池，它的典型用法如下：

```java
// 创建固定大小的线程池:
ExecutorService executor = Executors.newFixedThreadPool(3);
// 提交任务:
executor.submit(task1);
executor.submit(task2);
executor.submit(task3);
executor.submit(task4);
executor.submit(task5);
```

因为`ExecutorService`只是接口，Java标准库提供的几个常用实现类有：

- `FixedThreadPool`：线程数固定的线程池；
- `CachedThreadPool`：线程数根据任务动态调整的线程池；
- `SingleThreadExecutor`：仅单线程执行的线程池。

执行逻辑：

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        // 创建一个固定大小的线程池:
        ExecutorService es = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 6; i++) {
            es.submit(new Task("" + i));
        }
        // 关闭线程池:
        es.shutdown();
    }
}

class Task implements Runnable {
    private final String name;

    public Task(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("start task " + name);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
        System.out.println("end task " + name);
    }
}
/*
start task 0
start task 1
start task 2
start task 3
end task 0
start task 4
end task 1
start task 5
end task 2
end task 3
end task 4
end task 5
*/
```

使用`shutdown()`方法关闭线程池的时候，它会等待正在执行的任务先完成，然后再关闭。`shutdownNow()`会立刻停止正在执行的任务，`awaitTermination()`则会等待指定的时间让线程池关闭。

动态范围的线程池：

```java
int min = 4;
int max = 10;
ExecutorService es = new ThreadPoolExecutor(min, max,
        60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```

### 15.2 反复执行线程池ScheduledThreadPool

创建一个`ScheduledThreadPool`仍然是通过`Executors`类：

```java
ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);
```

我们可以提交一次性任务，它会在指定延迟后只执行一次：

```java
// 1秒后执行一次性任务:
ses.schedule(new Task("one-time"), 1, TimeUnit.SECONDS);
```

如果任务以固定的每3秒执行，我们可以这样写：

```java
// 2秒后开始执行定时任务，每3秒执行:
ses.scheduleAtFixedRate(new Task("fixed-rate"), 2, 3, TimeUnit.SECONDS);
```

如果任务以固定的3秒为间隔执行，我们可以这样写：

```java
// 2秒后开始执行定时任务，以3秒为间隔执行:
ses.scheduleWithFixedDelay(new Task("fixed-delay"), 2, 3, TimeUnit.SECONDS);
```

`FixedRate`是指任务总是以固定时间间隔触发，不管任务执行多长时间：

![image-20210509224407824](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210509224407824.png)

`FixedDelay`是指，上一次任务执行完毕后，等待固定的时间间隔，再执行下一次任务：

![image-20210509224432989](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210509224432989.png)

## 16. Future

```
class Task implements Runnable {
    public String result;

    public void run() {
        this.result = longTimeCalculation(); 
    }
}
```

`Runnable`接口有个问题，它的方法没有返回值。

Java标准库还提供了一个`Callable`接口，和`Runnable`接口比，它多了一个返回值：

```java
class Task implements Callable<String> {
    public String call() throws Exception {
        return longTimeCalculation(); 
    }
}
```

`Callable`接口是一个泛型接口，可以返回指定类型的结果。

`ExecutorService.submit()`方法返回一个`Future`类型的实例，代表一个未来能获取结果的对象：

```java
ExecutorService executor = Executors.newFixedThreadPool(4); 
// 定义任务:
Callable<String> task = new Task();
// 提交任务并获得Future:
Future<String> future = executor.submit(task);
// 从Future获取异步执行返回的结果:
String result = future.get(); // 可能阻塞
```

一个`Future<V>`接口表示一个未来可能会返回的结果，它定义的方法有：

- `get()`：获取结果（可能会等待）
- `get(long timeout, TimeUnit unit)`：获取结果，但只等待指定的时间；
- `cancel(boolean mayInterruptIfRunning)`：取消当前任务；
- `isDone()`：判断任务是否已完成。

## 17. CompletableFuture

Java 8开始引入了`CompletableFuture`，它针对`Future`做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。

举例：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 创建异步执行任务:
        CompletableFuture<Double> cf = CompletableFuture.supplyAsync(Main::fetchPrice);
        // 如果执行成功:
        cf.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 如果执行异常:
        cf.exceptionally((e) -> {
            e.printStackTrace();
            return null;
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(200);
    }

    static Double fetchPrice() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        if (Math.random() < 0.3) {
            throw new RuntimeException("fetch price failed!");
        }
        return 5 + Math.random() * 20;
    }
}
```

创建一个`CompletableFuture`是通过`CompletableFuture.supplyAsync()`实现的，它需要一个实现了`Supplier`接口的对象：

```java
public interface Supplier<T> {
    T get();
}
```

用lambda语法简化了一下，直接传入`Main::fetchPrice`,`Main.fetchPrice()`静态方法的签名符合`Supplier`接口的定义。

紧接着，`CompletableFuture`已经被提交给默认的线程池执行了，我们需要定义的是`CompletableFuture`完成时和异常时需要回调的实例。完成时，`CompletableFuture`会调用`Consumer`对象：

```java
public interface Consumer<T> {
    void accept(T t);
}
```

异常时，`CompletableFuture`会调用`Function`对象：

```java
public interface Function<T, R> {
    R apply(T t);
}
```

`CompletableFuture`的优点是：

- 异步任务结束时，会自动回调某个对象的方法；
- 异步任务出错时，会自动回调某个对象的方法；
- 主线程设置好回调后，不再关心异步任务的执行。

直接看例子：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 第一个任务:
        CompletableFuture<String> cfQuery = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油");
        });
        // cfQuery成功后继续执行下一个任务:
        CompletableFuture<Double> cfFetch = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice(code);
        });
        // cfFetch成功后打印结果:
        cfFetch.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(2000);
    }

    static String queryCode(String name) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        return "601857";
    }

    static Double fetchPrice(String code) {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        return 5 + Math.random() * 20;
    }
}
/*
price: 18.552424162240882
*/
```

除了串行执行外，多个`CompletableFuture`还可以并行执行。例如，我们考虑这样的场景：

同时从新浪和网易查询证券代码，只要任意一个返回结果，就进行下一步查询价格，查询价格也同时从新浪和网易查询，只要任意一个返回结果，就完成操作：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 两个CompletableFuture执行异步查询:
        CompletableFuture<String> cfQueryFromSina = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油", "https://finance.sina.com.cn/code/");
        });
        CompletableFuture<String> cfQueryFrom163 = CompletableFuture.supplyAsync(() -> {
            return queryCode("中国石油", "https://money.163.com/code/");
        });

        // 用anyOf合并为一个新的CompletableFuture:
        CompletableFuture<Object> cfQuery = CompletableFuture.anyOf(cfQueryFromSina, cfQueryFrom163);

        // 两个CompletableFuture执行异步查询:
        CompletableFuture<Double> cfFetchFromSina = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice((String) code, "https://finance.sina.com.cn/price/");
        });
        CompletableFuture<Double> cfFetchFrom163 = cfQuery.thenApplyAsync((code) -> {
            return fetchPrice((String) code, "https://money.163.com/price/");
        });

        // 用anyOf合并为一个新的CompletableFuture:
        CompletableFuture<Object> cfFetch = CompletableFuture.anyOf(cfFetchFromSina, cfFetchFrom163);

        // 最终结果:
        cfFetch.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(200);
    }

    static String queryCode(String name, String url) {
        System.out.println("query code from " + url + "...");
        try {
            Thread.sleep((long) (Math.random() * 100));
        } catch (InterruptedException e) {
        }
        return "601857";
    }

    static Double fetchPrice(String code, String url) {
        System.out.println("query price from " + url + "...");
        try {
            Thread.sleep((long) (Math.random() * 100));
        } catch (InterruptedException e) {
        }
        return 5 + Math.random() * 20;
    }
}
/*
query code from https://finance.sina.com.cn/code/...
query code from https://money.163.com/code/...
query price from https://finance.sina.com.cn/price/...
query price from https://money.163.com/price/...
price: 16.397253746873602
*/
```

![image-20210509230825458](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210509230825458.png)

除了`anyOf()`可以实现“任意个`CompletableFuture`只要一个成功”，`allOf()`可以实现“所有`CompletableFuture`都必须成功”，这些组合操作可以实现非常复杂的异步流程控制。

最后我们注意`CompletableFuture`的命名规则：

- `xxx()`：表示该方法将继续在已有的线程中执行；
- `xxxAsync()`：表示将异步在线程池中执行。

## 18. ForkJoin

Fork/Join任务的原理：判断一个任务是否足够小，如果是，直接计算，否则，就分拆成几个小任务分别计算。这个过程可以反复“裂变”成一系列小任务。

使用Fork/Join对大数据进行并行求和：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 创建2000个随机数组成的数组:
        long[] array = new long[2000];
        long expectedSum = 0;
        for (int i = 0; i < array.length; i++) {
            array[i] = random();
            expectedSum += array[i];
        }
        System.out.println("Expected sum: " + expectedSum);
        // fork/join:
        ForkJoinTask<Long> task = new SumTask(array, 0, array.length);
        long startTime = System.currentTimeMillis();
        Long result = ForkJoinPool.commonPool().invoke(task);
        long endTime = System.currentTimeMillis();
        System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
    }

    static Random random = new Random(0);

    static long random() {
        return random.nextInt(10000);
    }
}

class SumTask extends RecursiveTask<Long> {
    static final int THRESHOLD = 500;
    long[] array;
    int start;
    int end;

    SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // 如果任务足够小,直接计算:
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += this.array[i];
                // 故意放慢计算速度:
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                }
            }
            return sum;
        }
        // 任务太大,一分为二:
        int middle = (end + start) / 2;
        System.out.println(String.format("split %d~%d ==> %d~%d, %d~%d", start, end, start, middle, middle, end));
        SumTask subtask1 = new SumTask(this.array, start, middle);
        SumTask subtask2 = new SumTask(this.array, middle, end);
        invokeAll(subtask1, subtask2);
        Long subresult1 = subtask1.join();
        Long subresult2 = subtask2.join();
        Long result = subresult1 + subresult2;
        System.out.println("result = " + subresult1 + " + " + subresult2 + " ==> " + result);
        return result;
    }
}
/*
Expected sum: 9788366
split 0~2000 ==> 0~1000, 1000~2000
split 1000~2000 ==> 1000~1500, 1500~2000
split 0~1000 ==> 0~500, 500~1000
result = 2485485 + 2491717 ==> 4977202
result = 2391591 + 2419573 ==> 4811164
result = 4811164 + 4977202 ==> 9788366
Fork/join sum: 9788366 in 1084 ms.
*/
```

关键式分裂，代码结构如下：

```java
class SumTask extends RecursiveTask<Long> {
    protected Long compute() {
        // “分裂”子任务:
        SumTask subtask1 = new SumTask(...);
        SumTask subtask2 = new SumTask(...);
        // invokeAll会并行运行两个子任务:
        invokeAll(subtask1, subtask2);
        // 获得子任务的结果:
        Long subresult1 = subtask1.join();
        Long subresult2 = subtask2.join();
        // 汇总结果:
        return subresult1 + subresult2;
    }
}
```

Java标准库提供的`java.util.Arrays.parallelSort(array)`可以进行并行排序，它的原理就是内部通过Fork/Join对大数组分拆进行并行排序，在多核CPU上就可以大大提高排序的速度。

## 19. ThreadLocal

`Thread`对象代表一个线程，我们可以在代码中调用`Thread.currentThread()`获取当前线程。例如，打印日志时，可以同时打印出当前线程的名字：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        log("start main...");
        new Thread(() -> {
            log("run task...");
        }).start();
        new Thread(() -> {
            log("print...");
        }).start();
        log("end main.");
    }

    static void log(String s) {
        System.out.println(Thread.currentThread().getName() + ": " + s);
    }
}
/*
main: start main...
Thread-0: run task...
main: end main.
Thread-1: print...
*/
```

对于多任务，Java标准库提供的线程池可以方便地执行这些任务，同时复用线程。Web应用程序就是典型的多任务应用，每个用户请求页面时，我们都会创建一个任务，类似：

```java
public void process(User user) {
    checkPermission();
    doWork();
    saveStatus();
    sendResponse();
}
```

然后，通过线程池去执行这些任务。

Java标准库提供了一个特殊的`ThreadLocal`，它可以在一个线程中传递同一个对象。

`ThreadLocal`实例通常总是以静态字段初始化如下：

```java
static ThreadLocal<User> threadLocalUser = new ThreadLocal<>();
```

使用方式如下：

```java
void processUser(user) {
    try {
        threadLocalUser.set(user);
        step1();
        step2();
    } finally {
        threadLocalUser.remove();
    }
}
```

通过设置一个`User`实例关联到`ThreadLocal`中，在移除之前，所有方法都可以随时获取到该`User`实例：

```java
void step1() {
    User u = threadLocalUser.get();
    log();
    printUser();
}

void log() {
    User u = threadLocalUser.get();
    println(u.name);
}

void step2() {
    User u = threadLocalUser.get();
    checkUser(u.id);
}
```

可以把`ThreadLocal`看成一个全局`Map<Thread, Object>`：每个线程获取`ThreadLocal`变量时，总是使用`Thread`自身作为key：

```java
Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
```

因此，`ThreadLocal`相当于给每个线程都开辟了一个独立的存储空间，各个线程的`ThreadLocal`关联的实例互不干扰。

最后，特别注意`ThreadLocal`一定要在`finally`中清除：

```java
try {
    threadLocalUser.set(user);
    ...
} finally {
    threadLocalUser.remove();
}
```

为了保证能释放`ThreadLocal`关联的实例，我们可以通过`AutoCloseable`接口配合`try (resource) {...}`结构，让编译器自动为我们关闭。例如，一个保存了当前用户名的`ThreadLocal`可以封装为一个`UserContext`对象：

```java
public class UserContext implements AutoCloseable {

    static final ThreadLocal<String> ctx = new ThreadLocal<>();

    public UserContext(String user) {
        ctx.set(user);
    }

    public static String currentUser() {
        return ctx.get();
    }

    @Override
    public void close() {
        ctx.remove();
    }
}
```

使用的时候，我们借助`try (resource) {...}`结构，可以这么写：

```java
try (var ctx = new UserContext("Bob")) {
    // 可任意调用UserContext.currentUser():
    String currentUser = UserContext.currentUser();
} // 在此自动调用UserContext.close()方法释放ThreadLocal关联对象
```

这样就在`UserContext`中完全封装了`ThreadLocal`，外部代码在`try (resource) {...}`内部可以随时调用`UserContext.currentUser()`获取当前线程绑定的用户名。