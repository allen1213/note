

**Spring并没有保证bean的线程安全，需要自己编写解决线程安全问题的代码**，Spring对每个bean提供了一个scope属性来表示该bean的作用域：

- [ ] **singleton：**默认的scope，每个scope为singleton的bean都会被定义为一个单例对象，该对象的生命周期是与Spring IOC容器一致的
- [ ] **prototype：**bean被定义为在每次注入时都会创建一个新的对象
- [ ] **request**：bean被定义为在每个HTTP请求中创建一个单例对象，在单个请求中都会复用这一个单例对象
- [ ] **session：**bean被定义为在一个session的生命周期内创建一个单例对象
- [ ] **application：**bean被定义为在ServletContext的生命周期中复用一个单例对象
- [ ] **websocket：**bean被定义为在websocket的生命周期中复用一个单例对象



交由Spring管理的大多数对象其实都是一些无状态的对象，**每个单例的无状态对象都是线程安全的**，无状态对象包括常用的DO、DTO、VO、Service、DAO和Controller



**Spring没有对bean的多线程安全问题做出保证与措施，对于每个bean的线程安全问题，根本原因是每个bean自身的设计，不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，就使用`ThreadLocal`把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，就只能使用synchronized、lock、CAS等实现线程同步的方法**





### ThreadLocal

`ThreadLocal`是一个为线程提供线程局部变量的工具类，为线程提供一个线程私有的变量副本，这样多个线程都可以随意更改自己线程局部的变量，不会影响到其他线程

**ThreadLocal提供的只是一个浅拷贝，深拷贝可以通过重写`ThreadLocal`的`initialValue()`函数来自己实现**

`ThreadLocal`与`synchronized`这样的锁机制是不同的，锁更强调的是如何同步多个线程去正确地共享一个变量，`ThreadLocal`则是为了解决同一个变量如何不被多个线程共享，如果锁机制是用时间换空间的话，那`ThreadLocal`就是用空间换时间

ThreadLocal中有一个内部类ThreadLocalMap用来存储变量副本，它的key为ThreadLocal对象