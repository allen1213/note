

[参考](https://www.javazhiyin.com/59524.html)

```xml
<dependencies>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.61</version>
        </dependency>
        <!--elasticsearch-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>6.5.4</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>6.5.4</version>
        </dependency>
    </dependencies>
```





```yml
#base
server:
  port: 8080
#spring
spring:
  application:
    name: springboot-elasticsearch-example
#elasticsearch
elasticsearch:
  schema: http
  address: 127.0.0.1:9200
  connectTimeout: 5000
  socketTimeout: 5000
  connectionRequestTimeout: 5000
  maxConnectNum: 100
  maxConnectPerRoute: 100
```







### 使用 ElasticSearch搭建搜索系统  [项目地址](https://github.com/Motianshi/alldemo/tree/master/demo-search)  [参考](https://www.javazhiyin.com/62141.html)



##### maven依赖

```xml
<properties>
        <es.version>7.3.2</es.version>
    </properties>
        <!-- high client-->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>${es.version}</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch.client</groupId>
                <artifactId>elasticsearch-rest-client</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.elasticsearch</groupId>
                <artifactId>elasticsearch</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>${es.version}</version>
    </dependency>

    <!--rest low client high client以来低版本client所以需要引入-->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-client</artifactId>
        <version>${es.version}</version>
    </dependency>
```



##### ES配置文件`es-config.properties`

```properties
es.host=localhost
es.port=9200
es.token=es-token
es.charset=UTF-8
es.scheme=http

es.client.connectTimeOut=5000
es.client.socketTimeout=15000
```





##### RestHighLevelClient.java

```java
@Configuration
@PropertySource("classpath:es-config.properties")
public class RestHighLevelClientConfig {

    @Value("${es.host}")
    private String host;
    @Value("${es.port}")
    private int port;
    @Value("${es.scheme}")
    private String scheme;
    @Value("${es.token}")
    private String token;
    @Value("${es.charset}")
    private String charSet;
    @Value("${es.client.connectTimeOut}")
    private int connectTimeOut;
    @Value("${es.client.socketTimeout}")
    private int socketTimeout;

    @Bean
    public RestClientBuilder restClientBuilder() {
        RestClientBuilder restClientBuilder = RestClient.builder(
                new HttpHost(host, port, scheme)
        );

        Header[] defaultHeaders = new Header[]{
                new BasicHeader("Accept", "*/*"),
                new BasicHeader("Charset", charSet),
                //设置token 是为了安全 网关可以验证token来决定是否发起请求 我们这里只做象征性配置
                new BasicHeader("E_TOKEN", token)
        };
        restClientBuilder.setDefaultHeaders(defaultHeaders);
        restClientBuilder.setFailureListener(new RestClient.FailureListener(){
            @Override
            public void onFailure(Node node) {
                System.out.println("监听某个es节点失败");
            }
        });
        restClientBuilder.setRequestConfigCallback(builder ->
                builder.setConnectTimeout(connectTimeOut).setSocketTimeout(socketTimeout));
        return restClientBuilder;
    }

    @Bean
    public RestHighLevelClient restHighLevelClient(RestClientBuilder restClientBuilder) {
        return new RestHighLevelClient(restClientBuilder);
    }
}
```





##### 封装ES常用操作

```java
@Service
public class RestHighLevelClientService {

    @Autowired
    private RestHighLevelClient client;

    @Autowired
    private ObjectMapper mapper;

    /**
     * 创建索引
     * @param indexName
     * @param settings
     * @param mapping
     * @return
     * @throws IOException
     */
    public CreateIndexResponse createIndex(String indexName, String settings, String mapping) throws IOException {
        CreateIndexRequest request = new CreateIndexRequest(indexName);
        if (null != settings && !"".equals(settings)) {
            request.settings(settings, XContentType.JSON);
        }
        if (null != mapping && !"".equals(mapping)) {
            request.mapping(mapping, XContentType.JSON);
        }
        return client.indices().create(request, RequestOptions.DEFAULT);
    }

    /**
     * 判断 index 是否存在
     */
    public boolean indexExists(String indexName) throws IOException {
        GetIndexRequest request = new GetIndexRequest(indexName);
        return client.indices().exists(request, RequestOptions.DEFAULT);
    }

    /**
     * 搜索
    */
    public SearchResponse search(String field, String key, String rangeField, String 
                                 from, String to,String termField, String termVal, 
                                 String ... indexNames) throws IOException{
        SearchRequest request = new SearchRequest(indexNames);

        SearchSourceBuilder builder = new SearchSourceBuilder();
        BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
        boolQueryBuilder.must(new MatchQueryBuilder(field, key)).must(new RangeQueryBuilder(rangeField).from(from).to(to)).must(new TermQueryBuilder(termField, termVal));
        builder.query(boolQueryBuilder);
        request.source(builder);
        log.info("[搜索语句为:{}]",request.source().toString());
        return client.search(request, RequestOptions.DEFAULT);
    }

    /**
     * 批量导入
     * @param indexName
     * @param isAutoId 使用自动id 还是使用传入对象的id
     * @param source
     * @return
     * @throws IOException
     */
    public BulkResponse importAll(String indexName, boolean isAutoId, String  source) throws IOException{
        if (0 == source.length()){
            //todo 抛出异常 导入数据为空
        }
        BulkRequest request = new BulkRequest();
        JsonNode jsonNode = mapper.readTree(source);

        if (jsonNode.isArray()) {
            for (JsonNode node : jsonNode) {
                if (isAutoId) {
                    request.add(new IndexRequest(indexName).source(node.asText(), XContentType.JSON));
                } else {
                    request.add(new IndexRequest(indexName)
                            .id(node.get("id").asText())
                            .source(node.asText(), XContentType.JSON));
                }
            }
        }
        return client.bulk(request, RequestOptions.DEFAULT);
    }
```

