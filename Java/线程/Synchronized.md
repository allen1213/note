### Synchronized

关键字`synchronized`可以修饰方法或者以同步块的形式使用，确保多个线程在同一个时刻，只能有一个线程处于方法或者同步块中，**保证了线程对变量访问的可见性和排他性**

- [ ] 每个对象都可以做为锁，但一个对象做为锁时，应该被多个线程共享，在并发环境下，一个没有共享的对象作为锁是没有意义的

- [ ] 无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或函数当作锁
- [ ] 如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法 
- [ ] 但不同的对象实例的 synchronized方法是不相干扰的，也就是说，其它线程可以同时访问相同类的另一个对象实例中的synchronized方法



### Synchronized 的可重入性

线程会成功获取它自己已经持有的锁，这样的锁是可重入的

```java
class A {
    public synchronized void hello() {
        System.out.println("hello");
    }
}

class B extends A {
    public synchronized void doSomething() {
        System.out.println("doSomething...");
        super.hello();
        System.out.println("finish");
    }
}

// 输出的结果：
// doSomething...
// hello
// over
```

常见场景：

- [ ] 重排序，在没有同步的情况下，编译器、处理器以及运行时等都有可能对操作的执行顺序进行调整，即写的代码顺序和真正的执行顺序不一样, 导致读到的是一个失效的值
- [ ] 读取` long、double `等类型的变量，JVM 允许将一个 64 位的操作分解成两个 32 位的操作，读写在不同的线程中时，可能读到错误的高低位组合



常见方案：

加锁或使用 `Volatile` 关键字声明变量，但`Volatile `并不能保证操作的原子性，使用的好处在于访问 Volatile 变量并不会执行加锁操作，也就不会阻塞线程