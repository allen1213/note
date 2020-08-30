### Spring Cloud Sleuth：分布式请求链路跟踪  [来源](https://mp.weixin.qq.com/s?__biz=MzU1Nzg4NjgyMw==&mid=2247484087&idx=1&sn=817da72f117087fe7fa3061cb0bace84&scene=21#wechat_redirect)



Spring Cloud Sleuth 是分布式系统中跟踪服务间调用的工具，它可以直观地展示出一次请求的调用过程



随着系统越来越庞大，各个服务间的调用关系也变得越来越复杂，当客户端发起一个请求时，这个请求经过多个服务后，最终返回了结果，经过的每一个服务都有可能发生延迟或错误，从而导致请求失败，Spring Cloud Sleuth就可以跟踪请求链路，理清请求调用的服务链路，解决问题





### 给服务添加请求链路跟踪



下面将通过user-service和ribbon-service之间的服务调用来演示该功能，当调用ribbon-service的接口时，ribbon-service会通过RestTemplate来调用user-service提供的接口



首先给user-service和ribbon-service添加请求链路跟踪功能的支持，在user-service和ribbon-service中添加相关依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```



修改application.yml文件，配置收集日志的zipkin-server访问地址：

```yml
spring:
  zipkin:
    base-url:http://localhost:9411
  sleuth:
    sampler:
      probability:0.1#设置Sleuth的抽样收集概率
```





### 整合Zipkin获取及分析日志

Zipkin是Twitter的一个开源项目，可以用来获取和分析Spring Cloud Sleuth 中产生的请求链路跟踪日志，它提供的Web界面可以直观地查看请求链路跟踪信息



SpringBoot 2.0以上版本已经不需要自行搭建zipkin-server，我们可以从该地址  [下载 zipkin-server](https://repo1.maven.org/maven2/io/zipkin/java/zipkin-server/2.12.9/zipkin-server-2.12.9-exec.jar)，下载完成后使用以下命令运行zipkin-server：

```shell
java -jar zipkin-server-2.12.9-exec.jar
```



查看Zipkin页面访问地址：http://localhost:9411



启动eureka-sever，ribbon-service，user-service，多次调用（Sleuth为抽样收集）ribbon-service的接口http://localhost:8301/user/1 ，调用完后查看Zipkin首页会显示请求的链路跟踪信息

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwlPEib8Yic8ApWia3FW3NtiapnASS09znqNUQZbt3d8KUbvS0I1ES0xCnribqLBOlpib2EGibq1xwmkmBj4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





点击查看详情可以直观地看到请求调用链路和通过每个服务的耗时

![](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwlPEib8Yic8ApWia3FW3NtiapnAjpBPwcRzrGOyjveibadicoVU78COWEm6eIt93pJIgQCGViaamotxqm6Aw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





### 持久化跟踪信息



跟踪信息是存储在内存中的，如果把zipkin-server重启存储的跟踪信息全部丢失，若需要将所有信息存储下来，可以使用Elasticsearch等工具



安装了ES之后，使用以下命令运行，就可以把跟踪信息存储到Elasticsearch里面去了，重新启动也不会丢失

```shell
# STORAGE_TYPE：表示存储类型 ES_HOSTS：表示ES的访问地址
java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=elasticsearch --ES_HOSTS=localhost:9200
```



之后需要重新启动user-service和ribbon-service才能生效，重启后多次调用ribbon-service的接口http://localhost:8301/user/1，如果安装了Elasticsearch的可视化工具Kibana，也可以看到里面的跟踪信息



更多启动参数 [参考](https://github.com/openzipkin/zipkin/tree/master/zipkin-server#elasticsearch-storage)  