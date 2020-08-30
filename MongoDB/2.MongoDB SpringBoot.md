

#### Simple Demo

maven 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```



`application.yml`

```yml
spring:
  data:
    mongodb:
      uri:"mongodb://localhost:27017/test"
```



`User.java`

```java
public class User {

    private String id;
    private String name;
    private Integer age;
    private Integer grader;
    
    //getter and setter
}
```



在MongoDB数据库中新建一个collection：`user`

```js
for (var i = 0; i < 5; i ++) {
    db.user.insert({
        name: 'bobo' + i,
        age: i,
        gender: i
    })
}
```



测试类

```java
@SpringBootTest
public class MongoTest {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Test
    public void test1() {
        List<User> userList = mongoTemplate.findAll(User.class);
        if (userList != null && userList.size() > 0) {
            userList.forEach(user -> {
                System.out.println(user.toString());
            });
        }
    }

}
```



#### Web CRUD Demo

maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.60</version>
</dependency>
```



可视化工具 [admin-mongo](https://github.com/mrvautin/adminMongo)，新建 `user.java`

```java
@Document("user")
public class User {

    @Id
    private String id;

    @Field("name")
    private String name;

    @Field("password")
    private String password;

    @Field("address")
    private String address;

    @Field("create_time")
    private Date createTime;

    @Field("last_update_time")
    private Date lastUpdateTime;
    
    //getter and setter
    
    @Override
    public String toString() {
        return JSONObject.toJSONString(this);
    }
}
```



`JsonResult.java` 用于封装接口返回数据

```java
public class JsonResult {

    //未登录
    public static final int NO_LOGIN = 400;

    //登录失败
    public static final int LOGIN_FAILED = 401;

    //TOKEN过期
    public static final int TOKEN_EXPIRED = 402;

    //无权限
    public static final int NO_PERMISSION = 403;


    private Boolean success;
    private Integer code;
    private String msg;
    private Object data;

    public JsonResult(Boolean success) {
        this.success = success;
    }

    public JsonResult(Boolean success, String msg) {
        this.success = success;
        this.msg = msg;
    }

    public JsonResult(Integer code, Boolean success, String msg) {
        this.code = code;
        this.success = success;
        this.msg = msg;
    }

    public JsonResult(Boolean success, Object data) {
        this.success = success;
        this.data = data;
    }

    public JsonResult(Boolean success, Integer code, String msg, Object data) {
        this.success = success;
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    // ... ignore getter and setter methods

    public void put(String key, Object value) {
        if (data == null) {
            data = new HashMap<>();
        }
        ((Map) data).put(key, value);
    }

    public void putAll(Map<String, Object> map) {
        if (data == null) {
            data = new HashMap<>();
        }
        ((Map) data).putAll(map);
    }

    @Override
    public String toString() {
        return JSONObject.toJSONString(this);
    }
}
```



`UserController.java`

```java
@RestController
@RequestMapping(value = "/user")
public class UserController {

    @Autowired
    private MongoTemplate mongoTemplate;

    @GetMapping(value = "/select")
    public JsonResult list() {
        List<User> userList = mongoTemplate.findAll(User.class, "user");
        return new JsonResult(true, userList);
    }
}
```



更新数据

```java
@PostMapping(value = "/update")
public JsonResult update(User user) {

    if (user.getId() == null) {
        user.setCreateTime(new Date());
        user.setLastUpdateTime(new Date());
        User newUser = mongoTemplate.insert(user, "user");
        return new JsonResult(true, newUser);
    } else {
        Query query = new Query();
        query.addCriteria(Criteria.where("_id").is(user.getId()));

        Update update = new Update();
        update.set("name", user.getName());
        update.set("password", user.getPassword());
        update.set("address", user.getAddress());
        update.set("last_update_time", new Date());

        UpdateResult updateResult = mongoTemplate.updateFirst(query, update, "user");
        return new JsonResult(true, updateResult);
    }
}
```





删除数据

```java
@DeleteMapping(value = "{id}")
public JsonResult delete(@PathVariable String id) {
    Query query = new Query();
    query.addCriteria(Criteria.where("_id").is(id));
    DeleteResult deleteResult = mongoTemplate.remove(query, User.class, "user");
    return new JsonResult(true, deleteResult);
}
```

