认证，授权



```xml
<!--spring boot security-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



设置认证账号和密码的三种方式：

1. 配置文件中配置账号密码
2. 配置类中配置
3. 自定义实现类从数据库中查询



Remember me

1. 创建数据库表（可以不创建，配置类中配置启动时自动创建）
2. 配置类添加数据源