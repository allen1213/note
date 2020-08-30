### Spring Cloud Bus：消息总线  [来源](https://mp.weixin.qq.com/s?__biz=MzU1Nzg4NjgyMw==&mid=2247484080&idx=1&sn=2f55b7e298028e3e69e71ed8418d3ef6&scene=21#wechat_redirect)



Spring Cloud Bus 使用轻量级的消息代理来连接微服务架构中的各个服务，可以将其用于广播状态更改，例如配置中心配置更改或其他管理指令



我们通常会使用消息代理来构建一个主题，然后把微服务架构中的所有服务都连接到这个主题上，当我们向该主题发送消息时，所有订阅该主题的服务都会收到消息并进行消费



使用 Spring Cloud Bus 可以方便地构建起这套机制，所以 Spring Cloud Bus 又被称为消息总线。Spring Cloud Bus 配合 Spring Cloud Config 使用可以实现配置的动态刷新



目前  Spring Cloud Bus 支持两种消息代理：RabbitMQ 和 Kafka，下面以 RabbitMQ 为例来演示下使用Spring Cloud Bus 动态刷新配置的功能











查看docker容器的pid：docker inspect -f '{{.State.Pid}}' gitlab









### 动态刷新配置

使用 Spring Cloud Bus 动态刷新配置需要配合 Spring Cloud Config 一起使用，使用config-server、config-client模块





##### config-server 模块中添加消息总线支持

在pom.xml中添加相关依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



添加配置文件application-amqp.yml，主要是添加RabbitMQ的配置及暴露了刷新配置的Actuator端点

```yml
server:
  port:8904
spring:
  application:
    name:config-server
  cloud:
    config:
      server:
        git:
          uri:https://gitee.com/macrozheng/springcloud-config.git
          username:macro
          password:123456
          clone-on-start:true# 开启启动时直接从git获取配置
  rabbitmq:#rabbitmq相关配置
    host:localhost
    port:5672
    username:guest
    password:guest
eureka:
  client:
    service-url:
      defaultZone:http://localhost:8001/eureka/
management:
  endpoints:#暴露bus刷新配置的端点
    web:
      exposure:
        include:'bus-refresh'
```





##### config-client 模块中添加消息总线支持

在pom.xml中添加相关依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```



添加配置文件bootstrap-amqp1.yml及bootstrap-amqp2.yml用于启动两个不同的config-client，两个配置文件只有端口号不同

```yml
server:
  port:9004
spring:
  application:
    name:config-client
  cloud:
    config:
      profile:dev#启用环境名称
      label:dev#分支名称
      name:config#配置文件名称
      discovery:
        enabled:true
        service-id:config-server
  rabbitmq:#rabbitmq相关配置
    host:localhost
    port:5672
    username:guest
    password:guest
eureka:
  client:
    service-url:
      defaultZone:http://localhost:8001/eureka/
management:
  endpoints:
    web:
      exposure:
        include:'refresh'
```



启动eureka-server，以application-amqp.yml为配置启动config-server，以bootstrap-amqp1.yml为配置启动config-client，以bootstrap-amqp2.yml为配置再启动一个config-clien



登录RabbitMQ的控制台可以发现Spring Cloud Bus 创建了一个叫springCloudBus的交换机及三个以 springCloudBus.anonymous开头的队列



修改Git仓库中dev分支下的config-dev.yml配置文件：

```
# 修改前信息
config:
  info:"config info for dev(dev)"
# 修改后信息
config:
  info:"update config info for dev(dev)"
```



调用注册中心的接口刷新所有配置：`http://localhost:8904/actuator/bus-refresh`，刷新后再分别调用`http://localhost:9004/configInfo` 和 `http://localhost:9005/configInfo` 获取配置信息，发现都已经刷新了



如果只需要刷新指定实例的配置可以使用以下格式进行刷新：`http://localhost:8904/actuator/bus-refresh/{destination}`，我们这里以刷新运行在9004端口上的config-client为例`http://localhost:8904/actuator/bus-refresh/config-client:9004`





### 配合WebHooks使用



WebHooks相当于是一个钩子函数，可以配置当向Git仓库push代码时触发这个钩子函数

