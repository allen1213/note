#### 将应用打包成一个可执行的jar包
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

`@Import(AutoConfigurationPackages.Registrar.class)`  是将主配置类，也就是标注有 `@SpringBootApplication` 的所在包及其所有子包的所有组件扫描到 spring 容器中

`@Import(AutoConfigurationImportSelector.class)` 将类路径下 `META-INF/spring.factories` 所有需要导入的组件以全类名的方式返回，添加到容器中

#### Yaml 语法
- [ ] 双引号会将特殊字符转译，单引号会将 `\n` 当成一个字符串输出，双引号则会换行
- [ ] 数组可以写成 `array: - ... - ...`  ，也可以写成  `array: [...]`
- [ ] 使用 `@ConfigurationProperties(prefix = "")` 和 `@Component`  将配置文件中的值，映射到Entity实体类中，导入相应的依赖可以在写配置文件时出现提示
- [ ] 使用 `@Value("${name}")` 也可以将配置文件中的值映射到实体类中，该注解中可以填 `${name}` `#{10*2}` 或者字面量 `true`


#### @ConfigurationProperties/@Value
|@ConfigurationProperties|@Value
---|---|---
功能|只需在注解中写上前缀，批量注入配置文件中的属性|一个个指定
松散绑定|支持驼峰命名或者 `user-name` 的方式取值|不支持
SpEL|不支持`#{10*2}`|支持`#{10*2}`
JSR303数据校验|支持`@Validated @Eamil`等校验注解|不支持
复杂类型|支持|不支持


#### @PropertySource/@ImportResource
`@PropertySource` 加载配置文件
`@ImportResource(locations={})` 导入Spring的配置文件，让自定义的配置文件生效，一般加在启动类中，**但SpringBoot不推荐这种方式**，而是使用 `@Confinguration` 全注解方式


#### 配置文件占位符
```yml
person: 
  last-name: allen${random.uid}
  # 使用 : 可以指定默认值，若 ${person.hello} 没有定义，则使用 hello 作为值
  first-name: bobo${person.hello:hello}
```


#### Profile 多环境
> 多Profile 文件

在存在多个 Profile 配置文件中，默认使用  `application.properties`，若要激活其他配置文件，可在默认的配置文件中添加 `spring.profiles.active=dev`

> yml 文档快
```yml
# 默认使用第一个文档快
server:
  port: 8080
spring: 
  profiles: 
    # 激活指定的文档快
    active: dev
---
server:
  port: 8081
spring: 
  # 指定文档快的名称
  profiles: dev
---
server:
  port: 8082
spring: 
  profiles: prod
```

或者在运行时用命令的方式指定激活的环境 `java -jar app.jar --spring.profiles.active=dev`

#### 配置文件加载位置
- [ ] 项目根目录下的 `config`
- [ ] 项目根目录
- [ ] classpath:/config/
- [ ] classpath:/

以上顺序优先级从高到低，高优先级会覆盖低优先级的配置

或者在运行时，使用命令指定配置文件 `--spring.confing.location=D:\\....`


#### 外部配置加载顺序

以下顺序优先级从高到低

- [ ] 命令行 `java -jar app.jar --server.port=8000`
- [ ] 来自java:comp/env的JNDI属性
- [ ] Java系统属性，`System.getProperties()`
- [ ] 操作系统环境变量
- [ ] RandomValuePropertySource 配置的 `random.*`属性值
- [ ] jar包外部的 `application-{profile}.properties` 或带有 `spring.profile` 的  `application.yml`
- [ ] jar包内部的 `application-{profile}.properties` 或带有 `spring.profile` 的  `application.yml`
- [ ] jar包外部的 `application.properties` 或 不带 `spring.profile` 的  `application.yml`
- [ ] jar包内部的 `application.properties` 或 不带 `spring.profile` 的  `application.yml`
- [ ] 使用 `@Configuration` 注解类上的 `@PropertySoruce`
- [ ] 通过 SpringApplication.setDefaultProperties 指定的默认属性


#### 自动配置原理


`@EnableAutoConfiguration` 开启了自动配置功能


####  日志



#### 国际化



#### 切换Servlet容器


#### Servlet自动配置原理


#### 使用外部Servlet容器


#### @EnableWebMvc



#### JDBC
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```




#### 自定义starter

`@Configuration`：指定配置类
`@ConditionalOn`：指定在一定条件下才能自动装配
`@AutoConfigureAfter`：指定自动配置类的顺序
`@Bean`：添加到Spring容器中
`@ConfigurationProperties`：绑定相关的配置
`@EnableConfigurationProperties`：让配置文件生效加入容器中

将需要启动的自动配置类配置在 `META-INF/spring.factories`,启动器只用来做依赖导入，真正做处理的是 `AutoConfigurer`

- [ ] 新建项目，在项目中新建两个 model `my-spring-boot-starter` 和 `my-spring-boot-starter-autoconfigurer`，`autoconfigurer` 模块中只保留 `spring-boot-starter` 依赖， `my-spring-boot-starter` 模块中导入 `autoconfigurer` 模块 依赖，代码全部写在`autoconfigurer` 模块中

  

- [ ] `MyStarter.java`
```java
public class MyStarter {

    MyProperties properties;

    public MyProperties getProperties() {
        return properties;
    }

    public void setProperties(MyProperties properties) {
        this.properties = properties;
    }

    public String getProperties(String msg) {

        return properties.getPrefix() + " [ " + msg + " ] " + properties.getSuffix();
    }

}
```


- [ ] `MyProperties.java`
```java
//指定加载配置文件中前缀为 my 的值
@ConfigurationProperties(prefix = "my")
public class MyProperties {

    //定义了两个属性，这些属性可以在配置文件中通过 . 赋值，如：my.prefix=bobo
    private String prefix;

    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

- [ ] 配置类 `MyStarterAutoConfiguration.java`
```java
@Configuration
@ConditionalOnNotWebApplication
@EnableConfigurationProperties(MyProperties.class)
public class MyStarterAutoConfiguration {

    @Autowired
    MyProperties properties;

    @Bean
    public MyStarter myStarter() {
        MyStarter starter = new MyStarter();
        starter.setProperties(properties);
        return starter;
    }
}
```

- [ ] `resources/META-INF/spring.factories`
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.starter.MyStarterAutoConfiguration
```


























