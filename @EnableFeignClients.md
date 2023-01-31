nc -vz -w 2 10.0.100.117 6379



# @EnableFeignClients



### 使用

启动类添加 `@EnableFeignClients`

```java
@EnableFeignClients
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



使用注解`@FeignClient` 定义`feign`客户端，将远程服务`http://test-service/test/echo`映射为一个本地`Java`方法调用：

```java
@FeignClient(name = "test-service", path = "/test")
public interface TestServiceFeign {
    @RequestMapping(value = "/echo", method = RequestMethod.GET)
    Test msg(@RequestParam("parameter") String parameter);
}
```



使用

```java
@Autowired   
TestServiceFeign testServiceFeign;

public void test() {
    Test test = testServiceFeign.msg("Hello");
    log.info("echo : {}", test);
 }
```







### @EnableFeignClients

注解`@EnableFeignClients`告诉框架扫描所有使用注解`@FeignClient`定义的`feign`客户端，又通过注解`@Import`导入了`feign`客户端注册器 `FeignClientsRegistrar` 类：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
	// ...
}
```





### FeignClientsRegistrar

`feign`客户端注册器，`FeignClientsRegistrar`实现了接口 `ImportBeanDefinitionRegistrar`，而`ImportBeanDefinitionRegistrar`的设计目的，就是被某个实现类实现，配合使用`@Configuration`注解的使用者配置类，在配置类被处理时，用于额外注册一部分`bean`定义，对于上面的例子，使用者配置类就是 `Application`：

```java
public interface ImportBeanDefinitionRegistrar {

   /**
    * Register bean definitions as necessary based on the given annotation metadata of
    * the importing @Configuration class.
    * 根据使用者配置类的注解元数据注册bean定义
    * @param importingClassMetadata 使用者配置类的注解元数据
    * @param registry 当前bean定义注册表，一般指当前Spring应用上下文对象，当前Spring容器
    */
   public void registerBeanDefinitions(
     AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```



而方法`FeignClientsRegistrar` 中 ` registerBeanDefinitions`实现如下：

```java
@Override
   public void registerBeanDefinitions(AnnotationMetadata metadata,
                                       BeanDefinitionRegistry registry) {
    // 注册缺省配置到容器 registry
    registerDefaultConfiguration(metadata, registry);
    // 注册所发现的各个 feign 客户端到到容器 registry
    registerFeignClients(metadata, registry);
   }
```











```shell
docker run -p 8080:8080 \
-e PARAMS="--spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" \ -v /tmp:/data/applogs --name xxl-job-admin \
-d xuxueli/xxl-job-admin:2.3.1


docker run -p 10001:8080 -e JAVA_OPTS="-Xms512m -Xmx512m" -e PARAMS="--spring.datasource.url=jdbc:mysql://10.251.104.180:3306/xxl_job?Unicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=Family@ssw0rd"  -v /data/db/volumes/xxl-job/log:/data/applogs --name xxl-job-admin -d xuxueli/xxl-job-admin:2.3.0

```





```
docker run -p 9000:9000 --name minio \
  -e "MINIO_ACCESS_KEY=allen" \
  -e "MINIO_SECRET_KEY=allen" \
  -v /docker/minio/data:/data \
  -v /docker/minio/config:/root/.minio \
  -d minio/minio server /data
```





https://twitter.com/i/status/1603669266284179458

https://twitter.com/i/status/1602506528103096320



https://twitter.com/i/status/1602506528103096320

