# Filter



有如下Controller，通过配置过滤器和拦截器来实现对`get`方法执行时间计算的功能：

```java
@RestController
@RequestMapping("user")
public class UserController {

    @GetMapping("/{id:\\d+}")
    public void get(@PathVariable String id) {
        System.out.println(id);
    }
}
```



`TimeFilter`类，实现`javax.servlet.Filter`：

```java
public class TimeFilter implements Filter{
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器初始化");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("开始执行过滤器");
        Long start = new Date().getTime();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("过滤器耗时 " + (new Date().getTime() - start));
        System.out.println("结束执行过滤器");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }
}
```



要使该过滤器在Spring Boot中生效，可通过在`TimeFilter`上加上 `@WebFilter`注解，`urlPatterns`属性配置了哪些请求可以进入该过滤器，`/*`表示所有请求：

```java
@Component
@WebFilter(urlPatterns = {"/*"})
public class TimeFilter implements Filter {
   // ...
}
```



除了在过滤器类上加注解外，也可以通过配置`FilterRegistrationBean`来注册过滤器，定义一个`WebConfig`类，加上`@Configuration`注解表明其为配置类，然后通过`FilterRegistrationBean`来注册过滤器:

```java
@Configuration
public class WebConfig {
    @Bean
    public FilterRegistrationBean timeFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        TimeFilter timeFilter = new TimeFilter();
        filterRegistrationBean.setFilter(timeFilter);

        List<String> urlList = new ArrayList<>();
        urlList.add("/*");

        filterRegistrationBean.setUrlPatterns(urlList);
        return filterRegistrationBean;
    }
}
```









# Interceptor



过滤器中只可以获取到servletRequest对象，并不能获取到方法的名称，所属类，参数等额外的信息，拦截器多了Object和Exception对象，可以获取的信息比过滤器要多。但过滤器仍无法获取到方法的参数等信息，可以通过切面编程来实现这个目的，[参考](https://mrbird.cc/Spring-Boot-AOP log.html)

 

定义一个`TimeInterceptor`类，实现`org.springframework.web.servlet.HandlerInterceptor`接口，`preHandle`方法在处理拦截之前执行，`postHandle`只有当被拦截的方法没有抛出异常成功时才会处理，`afterCompletion`方法无论被拦截的方法抛出异常与否都会执行：

```java
public class TimeInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("处理拦截之前");
        httpServletRequest.setAttribute("startTime", new Date().getTime());
        System.out.println(((HandlerMethod) o).getBean().getClass().getName());
        System.out.println(((HandlerMethod) o).getMethod().getName());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("开始处理拦截");
        Long start = (Long) httpServletRequest.getAttribute("startTime");
        System.out.println("拦截器耗时 " + (new Date().getTime() - start));
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("处理拦截之后");
        Long start = (Long) httpServletRequest.getAttribute("startTime");
        System.out.println("拦截器耗时 " + (new Date().getTime() - start));
        System.out.println("异常信息 " + e);
    }
}
```





要使拦截器在Spring Boot中生效，还需在`WebConfig`中通过`InterceptorRegistry`注册过滤器:

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
    @Autowired
    private TimeInterceptor timeInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(timeInterceptor);
    }
}
```





过滤器要先于拦截器执行，各层的执行顺序为：`filter -> Interceptor -> ControllerAdvice -> Aspect -> Controller` 