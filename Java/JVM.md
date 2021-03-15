# JVM



## JVM 结构图 [来源](https://mrbird.cc/JVM-Learn.html) 

灰色部分是线程私有，不存在线程安全问题，橙色部分为线程共享区

![](https://mrbird.cc/img/QQ20200303-134536@2x.png)



## 类加载器

**类加载器** Class Loader 负责加载class文件，class文件在文件开头有特定的标识，这个特定的标识是十六进制字符**cafe babe**。类加载器将class文件字节码内容加载到内存中，并将这些内容转换成**方法区**中的运行时数据结构



ClassLoader只负责class文件的加载，至于它是否可以运行，则由执行引擎Execution Engine决定。类加载示意图：

![](https://mrbird.cc/img/QQ20200303-155915@2x.png)



### 类加载器分类

类加载器分为4种：

#### 启动类加载器

启动类加载器BootstrapClassLoader也叫根加载器，是虚拟机自带的加载器，底层由C++实现，用于加载`$JAVA_HOME/jre/lib/rt.jar`包内的class文件。rt.jar是Java基础类库，包含Java运行环境所需的基础类：



```java
public class Test {
    public static void main(String[] args) {
        Object object = new Object();
        System.out.println(object.getClass().getClassLoader());
    }
}
```



`Object`为Java自带的类，运行结果如下：

```
null
```



并没有返回预期的BootstrapClassLoader，这是因为BootstrapClassLoader底层是由C++实现的，并非Java实现





#### 拓展类加载器

拓展类加载器ExtClassLoader是虚拟机自带的加载器，由Java语言实现，用于加载`$JAVA_HOME/jre/lib/ext/**.jar`目录下的class文件，主要是Java在迭代过程中，一些拓展的功能：

```java
public class Test {
    public static void main(String[] args) {
        ZipInfo zipInfo = new ZipInfo();
        System.out.println(zipInfo.getClass().getClassLoader());
    }
}
```



`ZipInfo`是`$JAVA_HOME/jre/lib/ext/zipfs.jar`包里的一个类，程序运行结果如下：

```java
sun.misc.Launcher$ExtClassLoader@5e2de80c
```





#### 应用程序类加载器

应用程序类加载器AppClassLoader是虚拟机自带的加载器，用于加载当前应用的classpath的所有类，也就是平常写的那些Java代码，比如：

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Test.class.getClassLoader());
    }
}
```



程序运行结果：

```
sun.misc.Launcher$AppClassLoader@18b4aac2
```





#### 用户自定义加载器

除了使用上面三种JVM自带的类加载器外，也可以通过继承`Java.lang.ClassLoader`抽象类自定义一个类加载器



这四种类加载器是继承关系：

![](https://mrbird.cc/img/QQ20200303-165603@2x.png)





双亲委派的一个好处是：不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个String对象，所以自定义的Java类并不会污染JDK自带的那些类，即使全类名一样，这种保护机制也叫沙箱安全机制





## 程序计数器

程序计数器 Program Counter Register 又叫PC寄存器。每个线程都有一个程序计数器，是线程私有的



它是一个指针，指向方法区中的方法字节码，用来存储指向下一条指令的地址，也即将要执行的指令代码，由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不记：

![](https://mrbird.cc/img/QQ20200615-183140@2x.png)



如果执行的是一个Native方法，那这个计数器的值为undefied，CPU需要不停地切换各个线程，有了程序计数器后，当CPU切换回来后，就可以知道接着从哪开始继续执行程序





## 虚拟机栈 / Java栈

虚拟机栈也称为Java栈，每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个栈帧 Stack Frame，对应着一次次的Java方法调用



和PC寄存器一样，虚拟机栈的生命周期和线程一致。虚拟机栈主管Java程序的运行，它保存方法的局部变量（8种基本数据类型，对象引用地址）、部分结果，并参与方法的调用和返回

![](https://mrbird.cc/img/QQ20200618-091650@2x.png)



在一个活动线程中，一个时间点只会有一个活动的栈帧，即当前正在执行方法对应的栈帧（当前栈帧）；如果一个方法调用了另一个方法，那么对应的新的栈帧将会被创建出来，放在栈顶，成为新的当前栈帧



### 虚拟机栈大小调整

Java虚拟机规范允许虚拟机栈的大小固定不变或者动态扩展

- 固定情况下：如果线程请求分配的栈容量超过Java虚拟机允许的最大容量，则抛出StackOverflowError异常；
- 可动态扩展情况下：尝试扩展的时候无法申请到足够的内存；或者在创建新的线程的时候没有足够的内存去创建对应的虚拟机栈，则会抛出OutOfMemoryError异常。

不同平台的虚拟机栈默认大小不同：

- Linux/x64 (64-bit): 1024 KB
- macOS (64-bit): 1024 KB
- Oracle Solaris/x64 (64-bit): 1024 KB
- Windows: 默认值取决于虚拟内存



可以通过`-Xss`（`-XX:ThreadStackSize`简写）设置虚拟机栈大小，默认单位为字节。也可以通过k或者K指定单位为KB，m或M指定单位为MB，g或G指定单位为GB。下面这组配置都是将虚拟机栈大小设置为1024KB：

```
-Xss1m
-Xss1024k
-Xss1048576
```



### 栈帧内部结构

每个栈帧包含5个组成部分：局部变量表 Local Variables 、操作数栈 Operand Stack 、动态链接 Dynamic Linking 、方法返回地址 Return Address 和一些附加信息：

![](https://mrbird.cc/img/QQ20200618-140408@2x.png)



局部变量表是一个数字数组，用于存储方法参数和方法体内的局部变量，在局部变量表中，非静态方法的局部变量表和静态方法相比，多了个this对象即当前类，这也是静态方法内无法使用this的原因，因为静态方法的局部变量表中没有this对象



每一个独立的栈帧中除了包含局部变量表外，还包含一个FILO的操作数栈，用于保存计算过程中的中间结果，同时作为计算过程中变量临时的存储空间



在Java源文件被编译成字节码文件时，所有的变量和方法引用都作为符号引用保存在class文件的常量池里，动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用



存放调用该方法的pc寄存器的值。一个方法的结束分为以下两种方式：

- 正常执行结束；
- 出现未处理异常，非正常退出。

无论是哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，调用者的pc寄存器的值作为返回地址，即调用该方法的指令的下一条指令地址；异常退出时，返回地址需要通过异常表来确定





| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |





































































































































































