

### 策略模式

大部分都会讲到用策略模式去代替`if-else`，主要是定义统一行为`接口或抽象类`，并实现不同策略下的处理逻辑`对应实现类`，客户端使用时自己选择相应的处理类，利用工厂或其他方式



### 使用自定义注解实现  [来源](https://mp.weixin.qq.com/s/jBHbiD_e4dK8JzWrK72eyw)



订单实体类：

```java
@Data
public class Order {
    //订单来源
    private String source;
    
    //支付方式
    private String payMethod;
    
    //订单编号
    private String code;
    
    //订单金额
    private BigDecimal amount;
    
    // ...
}
```



对于不同来源pc端、移动端的订单需要不同的逻辑处理：

```java
@Service
public class OrderService {

    public void orderService(Order order) {
        // 处理pc端订单的逻辑
        if(order.getSource().equals("pc")){
        
        // 处理移动端订单的逻辑
        }else if(order.getSource().equals("mobile")){
        
        // 其他逻辑
        }else {
            
        }
    }
}
```



策略模式就是要去掉上面的一坨if-else，先总览下结构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WLIGprPy3z6M0Ej0dInjLDGzicwohlmN0JsmuwAnFOYJaYbPsPG38qkMf9jB8ibLHCapmvz7qT8OdcQdZpIMPwRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



首先定义`OrderHandler`接口，此接口规定了处理订单的方法

```java
public interface OrderHandler {
    void handle(Order order);
}
```



定义`OrderHandlerType`注解，表示某个类是用来处理何种来源的订单：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Service
public @interface OrderHandlerType {
    String source();
}
```



实现pc端和移动端订单处理各自的`handler`，并加上`OrderHandlerType`注解：

```java
@OrderHandlerType(source = "mobile")
public class MobileOrderHandler implements OrderHandler {
    @Override
    public void handle(Order order) {
        System.out.println("处理移动端订单");
    }
}

@OrderHandlerType(source = "pc")
public class PCOrderHandler implements OrderHandler {
    @Override
    public void handle(Order order) {
        System.out.println("处理PC端订单");
    }
}
```



向spring容器中注入各种订单处理的handler，并在`OrderService.orderService`方法中，通过策略/订单来源决定选择哪一个`OrderHandler`处理订单：

```java
@Service
public class OrderService {

    private Map<String, OrderHandler> orderHandleMap;

    @Autowired
    public void setOrderHandleMap(List<OrderHandler> orderHandlers) {
        // 注入各种类型的订单处理类
        orderHandleMap = orderHandlers.stream().collect(
                Collectors.toMap(orderHandler -> AnnotationUtils.findAnnotation(orderHandler.getClass(), OrderHandlerType.class).source(),
                        v -> v, (v1, v2) -> v1));
    }

    public void orderService(Order order) {
        // ...

        // 通过订单来源确定对应的handler
        OrderHandler orderHandler = orderHandleMap.get(order.getSource());
        orderHandler.handle(order);

        // ...
    }
}
```



`OrderService`中，维护了一个`orderHandleMap`，`key`为订单来源，`value`为对应的订单处理器`Handler`，通过`@Autowired`去初始化`orderHandleMap`，`if-else`就没有了，取而代之的是`orderHandleMap`根据订单来源获取对应的`OrderHandler`，然后执行`OrderHandler.handle`方法





`orderHandleMap`的`key`是订单来源，若想通过订单来源+订单支付，虽然可以使用两个字段组成的字符串，但是不推荐这种方法，因为key每添加内容都需要修改`service`，推荐的方法为把`key`的值改成自定义注解类型`OrderHandlerType`：

```java
public class OrderService {
    private Map<OrderHandlerType, OrderHandler> orderHandleMap;

    @Autowired
    public void setOrderHandleMap(List<OrderHandler> orderHandlers) {
        // 注入各种类型的订单处理类
        orderHandleMap = orderHandlers.stream().collect(
            Collectors.toMap(orderHandler -> AnnotationUtils.findAnnotation(orderHandler.getClass(), OrderHandlerType.class),
                    v -> v, (v1, v2) -> v1));
    }
    // ...
}
```



如何根据`order`的来源和支付方式去`orderHandleMap`里获取对应的`OrderHandler`呢，其实注解也是一个接口，只要定义一个类实现这个接口就能关联起来：

```java
public class OrderHandlerTypeImpl implements OrderHandlerType {

    private String source;
    private String payMethod;

    OrderHandlerTypeImpl(String source, String payMethod) {
        this.source = source;
        this.payMethod = payMethod;
    }

    @Override
    public String source() {
        return source;
    }

    @Override
    public String payMethod() {
        return payMethod;
    }

    @Override
    public Class<? extends Annotation> annotationType() {
        return OrderHandlerType.class;
    }
    
    // 需要重写 hashCode 和 equals 方法，否则会报空指针异常
    @Override
    public int hashCode() {
        int hashCode = 0;
        hashCode += (127 * "source".hashCode()) ^ source.hashCode();
        hashCode += (127 * "payMethod".hashCode()) ^ payMethod.hashCode();
        return hashCode;
    }


    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof OrderHandlerType)) {
            return false;
        }
        OrderHandlerType other = (OrderHandlerType) obj;
        return source.equals(other.source()) && payMethod.equals(other.payMethod());
    }

}
```



`OrderService.orderService()` ：

```java
public void orderService(Order order) {
    // ...

    // 通过订单来源确以及支付方式获取对应的handler
    OrderHandlerType orderHandlerType = new OrderHandlerTypeImpl(order.getSource(), order.getPayMethod());
    OrderHandler orderHandler = orderHandleMap.get(orderHandlerType);
    orderHandler.handle(order);

    // ...
}
```



这样一来，不管以后业务怎么发展，`OrderService`核心逻辑不会改变，只需要扩展`OrderHandler`即可





































