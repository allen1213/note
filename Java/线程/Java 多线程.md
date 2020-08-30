





### 线程栈空间

每个线程都有一个自己的本地内存空间--**线程栈空间**，线程执行时，先把变量从主内存读取到线程自己的本地内存空间，然后再对该变量进行操作，对该变量操作完后，**在某个时间再把变量刷新回主内存**     [来源](https://www.javazhiyin.com/63460.html)



### Volatile 简介

`volatile`是`Java`提供的一种轻量级的同步机制

`Java `语言包含两种内在的同步机制：同步块或方法和 `volatile `变量，相比于`synchronized`重量级锁，`volatile`更轻量，有时它更简单并且开销更低，不会引起线程上下文的切换和调度，但是`volatile `变量的同步性较差，而且其使用也更**容易出错**



### Volatile 的可见性



以下程序想通过改变 isRunning 的值，从而停止 run 方法中的 while 循环：

```java
public class RunThread extends Thread {

    //共享变量
    private boolean isRunning = true;
	// getter and setter...
    @Override
    public void run() {
        System.out.println("进入到run方法中了");
        while (isRunning) {
        }
        System.out.println("线程执行完成了");
    }
}

public class Run {
    public static void main(String[] args) {
        try {
            RunThread thread = new RunThread();
            thread.start();
            Thread.sleep(1000);
            // 试图将 isRunning 改成 false，以停止 run 方法中的 while 循环
            thread.setRunning(false);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



程序结果：`RunThread` 线程并不会终止，从而出现了死循环

原因分析：

现在有两个线程，一个是`main`线程，另一个是`RunThread`。它们都试图修改 `isRunning`变量。按照JVM内存模型，`main`线程将`isRunning`读取到本地线程内存空间，修改后再刷新回主内存

而在`JVM `设置成 `-server`模式运行程序时，线程会一直在私有堆栈中读取`isRunning`变量。因此，`RunThread`线程无法读到main线程改变的`isRunning`变量，从而出现了死循环，导致`RunThread`无法终止

解决方法：

 变量使用`volatile `关键字修饰，它强制线程从主内存中取 `volatile`修饰的变量

```java
volatile private boolean isRunning = true;
```



**volatile的作用是：使变量在多个线程间可见，具有可见性**



### Volatile 的非原子性

变量的自增操作 `i++`，分三个步骤：

- [ ] 从内存中读取出变量 i 的值
- [ ] 将 i 的值加1
- [ ] 将 加1 后的值写回内存

这说明 `i++ `并不是一个原子操作，因为，它分成了三步，有可能当某个线程执行到了第②时被中断了，就意味着只执行了其中的两个步骤，没有全部执行



volatile 的非原子性示例：

```java
public class MyThread extends Thread {
    // 使用 volatile 关键字修饰
    public volatile static int count;

    private static void addCount() {
        for (int i = 0; i < 100; i++) {
            count++;
        }
        System.out.println("count=" + count);
    }

    @Override
    public void run() {
        addCount();
    }
}

public class Run {
    public static void main(String[] args) {
        MyThread[] mythreadArray = new MyThread[100];
        for (int i = 0; i < 100; i++) {
            mythreadArray[i] = new MyThread();
        }

        for (int i = 0; i < 100; i++) {
            mythreadArray[i].start();
        }
    }
}
```



程序结果：`main `方法中创建了100个线程，这100个线程启动后去执行 `addCount()`，每个线程执行100次加1，正确的结果应该是 `100*100=10000`，但是实际`count`并没有达到`10000`

原因分析：

`volatile`修饰的变量并不保证对它的操作（自增）具有原子性，对于自增操作，可以使用JAVA的原子类`AutoicInteger`类保证原子自增

假设 i 自增到 5，线程A从主内存中读取i，值为5，将它存储到自己的线程空间中，执行加1操作，值为6。此时，CPU切换到线程B执行，从主从内存中读取变量i的值。由于线程A还没有来得及将加1后的结果写回到主内存，线程B就已经从主内存中读取了i，因此，线程B读到的变量 i 值还是5，相当于线程B读取的是已经过时的数据了，从而导致线程不安全性



**`volatile`不能保证线程的安全性，不具有原子性**



### Volatile 的有序性



`volatile`关键字修饰的变量不会被指令重排序优化，从而保证有序性



线程A执行的操作如下：

```java
Map configOptions ;
char[] configText;

volatile boolean initialized = false;

//线程A首先从文件中读取配置信息,调用process...处理配置信息,处理完成了将initialized 设置为true
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfig(configText, configOptions);//负责将配置信息configOptions 成功初始化
initialized = true;
```



线程B等待线程A把配置信息初始化成功后，使用配置信息去干活…..线程B执行的操作如下：

```java
while(!initialized){
    sleep();
}

//使用配置信息干活
doSomethingWithConfig();
```



如果`initialized`变量不用 `volatile `修饰，在线程A执行的代码中就有可能指令重排序，即：线程A执行的代码中的最后一行：`initialized = true `重排序到了 `processConfig`方法调用的前面执行了，这就意味着：配置信息还未成功初始化，但是`initialized`变量已经被设置成`true`了，那么就导致 线程B的`while`循环“提前”跳出，拿着一个还未成功初始化的配置信息去执行`doSomethingWithConfig`方法

因此，`initialized `变量就必须得用 `volatile`修饰。这样，就不会发生指令重排序，也即：只有当配置信息被线程A成功初始化之后，`initialized `变量才会初始化为`true`



**`volatile `修饰的变量会禁止指令重排序，保证有序性**





### Volatile / Synchronized  比较

`volatile`主要用在多个线程感知实例变量被更改了场合，从而使得各个线程获得最新的值，它强制线程每次从主内存中讲到变量，而不是从线程的私有内存中读取变量，从而**保证了数据的可见性**



比较：

- [ ] `volatile`轻量级，只能修饰变量，`synchronized`重量级，还可修饰方法

- [ ] `volatile`只能保证数据的可见性，不能用来同步，因为多个线程并发访问`volatile`修饰的变量不会阻塞



`synchronized`不仅保证可见性，而且还保证原子性，因为，只有获得了锁的线程才能进入临界区，从而保证临界区中的所有语句都全部执行，多个线程争抢`synchronized`锁对象时，会出现阻塞





### 线程安全性

线程安全性包括两个方面：可见性，原子性

从上面自增的例子中可以看出：仅仅使用`volatile`并不能保证线程安全性，而`synchronized`则可实现线程的安全性







