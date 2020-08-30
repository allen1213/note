MongoDB 是一款跨平台、面向文档的数据库，运行方式主要基于两个概念：集合 `collection` 与 文档 `document`

集合就是一组 MongoDB 文档，相当于关系型数据库中的表，集合位于单独的一个数据
库中，不能执行模式 schema，一个集合内的多个文档可以有多个不同的字段

文档是一组键-值对，文档有着动态的模式，同一集合内的文档不需要具有同样的字段或结构

MongoDB|关系型数据库
---|---
集合|表
文档|行
字段|列
内嵌文档|表 Join
由MongoDB提供的默认Key_id|主键



#### 范例文档

```json
{
    _id: ObjectId(7df78ad8902c)
    title: 'MongoDB Overview',
    description: 'MongoDB is no sql database',
    by: 'tutorials point',
    url: 'http://www.tutorialspoint.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100,
    comments: [
        {
            user:'user1',
            message: 'My first comment',
            dateCreated: new Date(2011,1,20,2,15),
            like: 0
        },
        {
            user:'user2',
            message: 'My second comments',
            dateCreated: new Date(2011,1,25,7,45),
            like: 5
        }
    ]
}
```

`_id` 是一个 12 字节长的十六进制数，它保证了每一个文档的唯一性，在插入文档时，需要提供 _id ，若不提供，那么 MongoDB 就会为每一文档提供一个唯一的 id， _id 的头 4 个字节代表的是当前的时间戳，接着的后 3 个字节表示的是机器 id 号，接着的 2 个字节表示 MongoDB 服务器进程 id，最后的 3 个字节代表递增值





#### 关系型数据库与MongoDB范例

假设一个客户需要为他的博客站点设计一个数据库，网站需求如下所示：
- [ ]  每篇博客都具有唯一的标题、描述以及 URL
- [ ]  每篇博客都具有一个或多个标签
- [ ]  每篇博客都具有发表者的名称，以及喜欢
- [ ]  每篇博客都有用户的评论，用户名、消息、日期时间以及评论的喜欢度
- [ ]  每篇博客都可以有 0 个或多个评论

在 RDBMS 中，设计一个能够满足上述需求的数据库模式至少需要 3 个表： `comments`评论表 ， `post`博客表，`tag_list` 博客标签表

在 MongoDB 中，设计出来的模式只有一个集合 post：
```json
{
    _id: POST_ID
    title: TITLE_OF_POST,
    description: POST_DESCRIPTION,
    by: POST_BY,
    url: URL_OF_POST,
    tags: [TAG1, TAG2, TAG3],
    likes: TOTAL_LIKES,
    comments: [
        {
            user:'COMMENT_BY',
            message: TEXT,
            dateCreated: DATE_TIME,
            like: LIKES
        },
        {
            user:'COMMENT_BY',
            message: TEXT,
            dateCreated: DATE_TIME,
            like: LIKES
        }
    ]
}

```





#### MongoDB数据类型

MongoDB 支持如下数据类型：
- [ ]  String：字符串
- [ ]  Integer：整型数值
- [ ]  Boolean：布尔值
- [ ]  Double：双精度浮点值
- [ ]  Min/Max keys：将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比
- [ ]  Arrays：用于将数组或列表或多个值存储为一个键
- [ ]  Timestamp：时间戳
- [ ]  Object：用于内嵌文档
- [ ]  Null：用于创建空值
- [ ]  Symbol：该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言
- [ ]  Date：日期时间
- [ ]  Object ID：对象 ID，用于创建文档的 ID
- [ ]  Binary Data：二进制数据
- [ ]  Code：代码类型，用于在文档中存储 JavaScript 代码
- [ ]  Regular expression：正则表达式类型





#### MongoDB常用命令

> #### 增删改

`use database_name`：创建数据库,在 MongoDB 中，默认的数据库是 test，若没有创建任何数据库，那么集合就会保存在 test 数据库中

`show dbs`：查看数据库列表，若数据库中没有文档使用该命令不会显示出来
`db`：查看当前选定的数据库

`db.dropDatabase()`：删除数据库，删除前先使用 `use` 进入该数据库

`db.createCollection(name, options[])`：创建集合，`options` 是可选的，用于指定有关内存大小及索引的选型

不带参数的创建集合：
```shell
db.createCollection("mycollection")
```

带参数创建集合：
**capped**：布尔型，用于指定固定大小的集合，当达到最大值时会自动覆盖最早的文档，值为 `true` 时，必须指定 size参数

**autoIndexID**：布尔型，如为 true，自动在 _id 字段创建索引，默认为 false

**size**：为固定集合指定一个最大值（以字节计），如果 capped 为 true，也需要指定该字段

**max**：指定固定集合中包含文档的最大数量
```shell
db.createCollection("mycol", { capped : true, autoIndexID : true, size : 6142800, max : 10000 } )
```


`show collections`：查看创建的集合

在 MongoDB 中不需要创建集合，当插入一些文档时，MongoDB 会自动创建集合：
```shell
#创建 tutorialspoint 集合，并且插入了数据
db.tutorialspoint.insert({"name" : "tutorialspoint"})
```

删除集合
```shell
use mydb
show collections
db.collectionName.drop()
```

`insert()` `save()` ：插入文档
```shell
#db.COLLECTION_NAME.insert(document)

db.mycol.insert({
    _id: ObjectId(7df78ad8902c),
    title: 'MongoDB Overview',
    description: 'MongoDB is no sql database',
    by: 'tutorials point',
    url: 'http://www.tutorialspoint.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})

```
若没有指定文档的 _id ，`save() ` 和 `insert()` 的功能一样， 如果指定了文档的 _id ，那么它会覆盖掉含有 save() 方法中指定的 _id 的文档的全部数据



`remove()` 删除文档
MongoDB 利用 remove() 方法 清除集合中的文档，它有 2 个可选参数：
- [ ] deletion criteria：（可选）删除文档的标准
- [ ] justOne：（可选）如果设为 true 或 1，则只删除一个文档

```shell
#删除其中所有标题为 'MongoDB Overview' 的文档
db.mycol.remove({'title':'MongoDB Overview'})

#只删除一条记录
db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)

#若没有指定删除标准，则会删除集合中所有文档，等同于 SQL 中的 truncate 命令
db.mycol.remove()
```



> #### 查询

`db.COLLECTION_NAME.find()`:  会以非结构化的方式来显示所有文档，在 MongoDB 中执行 `find()` 方法时，显示的是一个文档的所有字段,要想限制，可以利用 0 或 1 来设置字段列表,1 用于显示字段，0 用于隐藏字段

```shell
#显示 title，隐藏 _id
db.mycol.find({},{"title":1,_id:0})
```

`db.mycol.find().pretty()` : 格式化显示结果


`db.<collectionName>.count()`：文档记录数



**条件查询**

操作|格式|范例|RDBMS类似语句
---|---|---|---
=|{key:value}|db.name.find({"name":"bobo"}).pretty()|where name = 'bobo'
<|{key: {$lt:<value}}|db.name.find({"name":{$lt:50}}).pretty()|where name < 50
<=|{key:{$lte:<value}}|db.name.find({"name":{$lte:50}}).pretty()|where name <= 50
`>`|{key:{$gt:value}}|db.name.find({"name":{$gt:50}}).pretty()|where name > 50
`>=`|{key:{$gte:value}}|db.name.find({"name":{$gte:50}}).pretty()|where name >= 50
!=|{key:{$ne:value}}|db.name.find({"name":{$ne:50}}).pretty()|where name != 'bobo'


在 `find() `方法中，如果传入多个键，并用逗号分隔，那么 MongoDB 会把它看成是 AND 条件：
```shell
db.mycol.find({key1:value1, key2:value2}).pretty()
```

若基于 OR 条件来查询文档，可以使用关键字 `$or`, OR 条件的基本语法格式为：
```shell
db.mycol.find({
    $or: [
    	{key1: value1}, {key2:value2}
    ]
}).pretty()

#示例
db.mycol.find({"likes": {$gt:10}, $or: [{"by": "tutorials point"},{"title": "MongoDB Overview"}]}).pretty()
```

MongoDB 中的 `update()` 与 `save()` 方法都能用于更新集合中的文档，`update()` 方法更新已有文档中的值，而 `save() ` 方法则是用传入该方法的文档来替换已有文档
```shell
db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}})
```
MongoDB 默认只更新单个文档，要想更新多个文档，需要把参数 `multi` 设为 `true` 
```shell
db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}},{multi:true})
```


`db.COLLECTION_NAME.find().limit(NUMBER)`：显示Number条数据，若未指定 `limit()` 方法中的数值参数，则显示该集合内的所有文档

`db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)`：


`db.COLLECTION_NAME.find().sort({KEY:1})`：根据 KEY 字段排序，1 表示升序， -1 表示降序



> 其他

登陆MongoDB

```shell
mongo --host <hostName> --port <port> -u <username> -p <password> --authenticationDatabase <dbname>

mongo --host 192.168.140.11 -u test -p 123456 --authenticationDatabase test_db
```



创建用户

```bash
use <databaseName>
db.createUser({ user: '<username>', pwd: '<password>', roles: [ { role: "readWrite", db: "<databaseName>" } ] });

#例子:
use admin
db.createUser({ user: 'admin', pwd: '123456', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
```



列出所有集合，用户，角色

```bash
# 列出当前database下所有的集合:
show collections;
db.getCollectionNames();

# 列出当前database下所有的用户
show users;
db.getUsers();

# 列出当前dababase下所有角色
show roles;
```



管理命令

```bash
# 获取集合的统计信息,比如空间占用，总大小，引擎信息等
db.<collectionName>.stats()
db.printCollectionStats()

# 获取集合的延迟统计信息,比如读写的次数,时间等等
db.<collectionName>.latencyStats()

# 获取数据和索引的集合大小
# 集合的大小
db.<collectionName>.dataSize()

# 集合中存储文档的总大小
db.<collectionName>.storageSize()

# 集合数据和索引的总大小(以字节为单位)
db.<collectionName>.totalSize()

# 集合中所有索引的总大小
db.<collectionName>.totalIndexSize()
```







#### 索引

索引是一种特殊的数据结构，将一小块数据集保存为容易遍历的形式，索引能够存储某种特殊字段或字段集的值，并按照索引指定的方式将字段值进行排序


MongoDB 中使用 `ensureIndex()`  方法，创建索引：
key 是想创建索引的字段名称，1 代表按升序排列字段值,-1 代表按降序排列
```shell
db.COLLECTION_NAME.ensureIndex({KEY:1})

#示例，title 和 description 组合索引
db.mycol.ensureIndex({"title":1,"description":-1})
```



#### 定时索引

MongoDB中存在一种索引,叫做 `TTL` 索引，这种索引允许为每一个文档设置一个超时时间，一个文档达到预设置的老化程度后就会被删除

MongoDB的TTL功能依赖于mongodb中的后台线程，该线程读取索引中的日期类型值并从集合中删除过期的文档

MongoDB每分钟对TTL索引进行一次清理，TTL索引不保证在到期时立即删除过期数据,由于删除过期文档的后台任务每60秒运行一次，所以文档可能在文档到期和后台任务运行之间的期间保留在集合中



创建 TTL 索引：

```bash
# 超时时间为24小时,默认是前台运行，可以通过background:true设置为后台模式
db.user_session.createIndex({"updated":1},{expireAfterSeconds:60*60*24});
```

这样在`updated`字段上创建了一个TTL索引，如果一个文档的`updated`字段存在并且它的值是日期类型，当服务器时间比文档的updated字段的时间晚expireAfterSeconds秒时，文档就会被删除



MongoDB保存时间使用的UTC时间，在查询出来的结果的时候会转换为GMT时间，所以保存的时间和电脑时间相差8个小时(GMT+8)，在查询的时候可以使用`new Date()`直接进行时间的比较，new Date传入的参数是GMT时间：
```bash
db.getCollection('user_session').find({updated:{$gt: new Date("2019-07-12 14:00:00")}}) 
```



MongoDB不支持使用createIndex来重新设置过期时间，只可以使用`collMod`命令修改expireAfterSeconds的值：

```bash
db.runCommand({collMod:"user_session",index: {name:"updated_1",expireAfterSeconds: 120}});
```

修改成功后，会收到这样的消息(之前的过期时间是一分钟,现在修改为2分钟)

```json
{
    "expireAfterSeconds_old" : 60.0,
    "expireAfterSeconds_new" : 120.0,
    "ok" : 1.0
}
```



在一个给定的集合上可以有多个TTL索引，可以在created和updated字段分别建立ttl索引,但是**不能同时使用两个字段建立复合ttl索引**,也**不能在同一个字段上同时创建TTL索引和普通索引**，但是可以像“普通索引”一样用来优化排序和查询





#### 聚合

聚合操作能够处理数据记录并返回计算结果，相当于 SQL 中的 count(*) 组合 group by

```shell
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])

#上面的语句等同于
select by_user, count(*) from mycol group by by_user
```








#### 关系

MongoDB 中的关系表示文档之间的逻辑相关方式，可以通过内嵌`Embedded`或引用`Referenced`两种方式建模，引入分为手动引入和数据库引入，这关系可能是 1：1、1：N、N：1， N：N

例子：一个用户可能有多个地址，这是一个 1：N 的关系
下面是一个结构非常简单的 user 文档：

```json
{
    "_id":ObjectId("52ffc33cd85242f436000001"),
    "name": "Tom Hanks",
    "contact": "987654321",
    "dob": "01-01-1991"
}
```
下面是 address 文档的结构：
```json
{
    "_id":ObjectId("52ffc4a5d85242602e000000"),
    "building": "22 A, Indiana Apt",
    "pincode": 123456,
    "city": "Los Angeles",
    "state": "California"
}
```

使用内嵌将地址文档内嵌到 user 文档中：
```json
{
    "_id":ObjectId("52ffc33cd85242f436000001"),
    "contact": "987654321",
    "dob": "01-01-1991",
    "name": "Tom Benzamin",
    "address": [{...},{...}...]
}
```
该方法会将所有相关数据都保存在一个文档中，从而易于检索和维护，使用 `db.users.findOne({"name":"Tom Benzamin"},{"address":1})` 查找数据，这种方法的缺点是，如果内嵌文档不断增长，会对读写性能造成影响

**手动引用方式**：的用户和地址文档都将分别存放，而用户文档会包含一个字段，用来引用地址文档 id 字段

```json
{
    "_id":ObjectId("52ffc33cd85242f436000001"),
    "contact": "987654321",
    "dob": "01-01-1991",
    "name": "Tom Benzamin",
    "address_ids": [
        ObjectId("52ffc4a5d85242602e000000"),
        ObjectId("52ffc4a5d85242602e000001")
    ]
}
```
利用这种方法时，需要进行两种查询：首先从 user 文档处获取 address_ids，其次从 address 集合中获取这些地址
```shell
var result = db.users.findOne({"name":"Tom Benzamin"},{"address_ids":1})
var addresses = db.address.find({"_id":{"$in":result["address_ids"]}})
```



**数据库引入方式**
在 DBRef 中有三个字段：`$ref` 指定所引用文档的集合名称，`$id `指定引用文档的 `-id` 字段 ,`$db` 字段是可选的，包含引用文档所在数据库的名称

```json
{
    "_id":ObjectId("53402597d852426020000002"),
    "address": {
        "$ref": "address_home",
        "$id": ObjectId("534009e4d852427820000002"),
        "$db": "tutorialspoint"
    },
    "contact": "987654321",
    "dob": "01-01-1991",
    "name": "Tom Benzamin"
}
```
数据库引用字段 address 指定出，引用地址文档位于 tutorialspoint 数据库的 address_home 集合中，并且它的 id 为 534009e4d852427820000002

查找：
```json
var user = db.users.findOne({"name":"Tom Benzamin"})
var dbRef = user.address
db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
```





#### 覆盖索引查询

覆盖查询都具有以下两个特点：
- [ ] 查询中的所有字段都属于一个索引
- [ ]  查询所返回的所有字段也都属于同一索引内

为了测试覆盖查询，假设在一个 users 集合中包含下列文档：
```json
{
    "_id": ObjectId("53402597d852426020000002"),
    "contact": "987654321",
    "dob": "01-01-1991",
    "gender": "M",
    "name": "Tom Benzamin",
    "user_name": "tombenzamin"
}
```

`db.users.ensureIndex({gender:1,user_name:1})`创建users 集合中的 gender 和 user_name 字段符合索引，这一索引将覆盖下列查询：
```shell
db.users.find({gender:"M"},{user_name:1,_id:0})
```
也就是说，对于上面的查询，MongoDB 不会去查看文档，转而从索引数据中获取所需的数据

**索引字段是数组或子文档时，索引不能覆盖查询**





#### 查询分析

使用 `$explain` 和 `$hint` 获取查询的效率

**`$explain`** 操作提供的消息包括：查询消息、查询所使用的索引以及其他的统计信息，如 
```
db.users.find({gender:"M"},{user_name:1,_id:0}).explain()
```

分析结果：
```json
{
    "cursor" : "BtreeCursor gender_1_user_name_1",
    "isMultiKey" : false,
    "n" : 1,
    "nscannedObjects" : 0,
    "nscanned" : 1,
    "nscannedObjectsAllPlans" : 0,
    "nscannedAllPlans" : 1,
    "scanAndOrder" : false,
    "indexOnly" : true,
    "nYields" : 0,
    "nChunkSkips" : 0,
    "millis" : 0,
    "indexBounds" : {
        "gender" : [
            [
                "M",
                "M"
            ]
        ],
        "user_name" : [
            [
                {
                  "$minElement" : 1
                },
                {
                  "$maxElement" : 1
                }
            ]
        ]
    }
}
```

- [ ] indexOnly 的真值代表该查询使用了索引
- [ ] cursor 字段指定了游标所用的类型，BTreeCursor 类型代表了使用了索引并且提供了所用索引的名称，BasicCursor 表示进行了完整扫描，没有使用任何索引
- [ ] n 代表所返回的匹配文档的数量
- [ ] nscannedObjects 表示已扫描文档的总数
- [ ] nscanned 所扫描的文档或索引项的总数

**`$hint`** 操作符强制索引优化器使用指定的索引运行查询，适用于测试带有多个索引的查询性能，如下列查询指定了用于该查询的 gender 和 user_name 字段的索引：

```
db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})
```
使用 `$hint `来优化上述查询：
```
>db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1}).explain()
```



#### 原子操作

MongoDB 只提供了对单个文档的原子操作，并不支持多文档原子事务，维持原子性的建议方法是利用**内嵌文档**



#### 高级索引
假如一个 users 集合中具有下列文档：
```json
{
    "address": {
        "city": "Los Angeles",
        "state": "California",
        "pincode": "123"
    },
    "tags": [
        "music",
        "cricket",
        "blogs"
    ],
    "name": "Tom Benzamin"
}
```
上述文档包含一个地址子文档（address sub-document）与一个标签数组（tags array）

> 索引数组字段

要根据标签来搜索用户文档，首先在集合中创建一个标签数组的索引：
```
db.users.ensureIndex({"tags":1})
```
在标签数组上创建一个索引，也就为每一个字段创建了单独的索引项，因此在该例中，当创建了标签数组的索引时，也就为它的music（音乐）、cricket（板球）以及 blog（博客）值创建了独立的索引

验证索引的正确：
```
db.users.find({tags:"cricket"}).explain()
```
上述 explain 命令的执行结果是 "cursor" : "BtreeCursor tags_1"，表示使用了正确的索引



> 索引子文档字段

假设需要根据市（city）、州（state）、个人身份号码（pincode）字段来搜索文档，因为所有这些字段都属于地址子文档字段的一部分，所以需要在子文档的所有字段上创建索引：
```
db.users.ensureIndex({"address.city":1,"address.state":1,"address.pincode":1})
```
一旦创建了索引，就可以使用索引来搜索任何子文档字段：
```
db.users.find({"address.city":"Los Angeles"})
```
记住，查询表达式必须遵循指定索引的顺序，因此上面创建的索引将支持如下查询：
```
db.users.find({"address.city":"Los Angeles","address.state":"California"})
```
另外也支持如下这样的查询：
```
db.users.find({"address.city":"LosAngeles","address.state":"California","address.pincode":"123"})
```




#### 索引限制

每个索引都会占据一些空间，从而也会在每次插入、更新与删除操作时产生一定的开销，所以如果集合很少使用读取操作，就尽量不要使用索引

因为索引存储在内存中，所以应保证索引总体的大小不超过内存的容量，如果索引总体积超出了内存容量，就会删除部分索引，从而降低性能

**查询限制**，当查询使用以下元素时，不能使用索引：
- [ ] 正则表达式或否定运算符（ $nin 、 $not ，等等）
- [ ] 算术运算符（比如 $mod ）
- [ ] $where 子句

索引建限制：自 MongoDB 2.6 版本起，如果已有索引字段的值超出了索引键限制，则无法创建索引


插入文档超过索引建限制：如果文档的索引字段值超出了索引键的限制，MongoDB 不会将任何文档插入已索引集合

最大范围：
- [ ] 集合索引数不能超过 64 个
- [ ] 索引名称长度不能大于 125 个字符
- [ ] 复合索引最多能有 31 个被索引的字段




#### ObjectId

ObjectId 是一个 12 字节的 BSON 类型，其结构如下：
- [ ] 前 4 个字节代表 UNIX 的时间戳（以秒计）
- [ ] 接下来的 3 个字节代表机器标识符
- [ ] 接下来的 2 个字节代表进程 id
- [ ] 最后 3 个字节代表随机数

MongoDB 使用 ObjectId 作为每一文档的 _id 字段的默认值（在创建文档时产生）,ObjectId 的复杂组合保证了 _id 字段的唯一性

**创建新的 ObjectId**

```
newObjectId = ObjectId()
```
上述代码返回一个唯一的 id：`ObjectId("5349b4ddd2781d08c09890f3")`

如果不用 MongoDB 来生成，可以自己提供一个 12 字节的 id：
```
myObjectId = ObjectId("5349b4ddd2781d08c09890f4")
```
**创建文档时间戳**，因为 _id ObjectId 默认保存 4 字节的时间戳，所以在大多数情况下不需要保存文档的创建时间，使用 getTimestamp 方法可获取文档的创建时间：

```
ObjectId("5349b4ddd2781d08c09890f4").getTimestamp()
```
返回标准时期格式表示的文档创建时间：`ISODate("2014-04-12T21:49:17Z")`


**将 ObjectId 转换为字符串**：`newObjectId.str`








#### Map Reduce
在 MongoDB 中，映射归约 Map-Reduce 是一种将大量数据压缩成有用的聚合结果的数据处理范式，MongoDB 使用 mapReduce 命令来实现映射归约操作，通常用来处理大型数据

mapReduce 命令的基本格式为：
```
db.collection.mapReduce(
    function() {emit(key,value);}, //map function
    function(key,values) {return reduceFunction}, //reduce function
    {
        out: collection,
        query: document,
        sort: document,
        limit: number
    }
)
```

mapReduce 函数首先查询集合，然后将结果文档利用 emit 函数映射为键值对，再根据有多个值的键来简化，上述语法格式中：
- [ ] map 一个 JavaScript 函数，将一个值与键对应起来，并生成键值对
- [ ] reduce 一个 JavaScript 函数，用来减少或组合所有拥有同一键的文档
- [ ] out 指定映射归约查询结果的位置
- [ ] query 指定选择文档所用的选择标准（可选）
- [ ] sort 指定可选的排序标准
- [ ] limit 指定返回的文档的最大数量值（可选）


> 使用 mapReduce


以下面这个存储用户发帖的文档结构。该文档存储用户的用户名（user_name）和发帖状态（status）
```json
{
"post_text": "tutorialspoint is an awesome website for tutorials",
"user_name": "mark",
"status":"active"
}
```
在 posts 集合上使用 mapReduce 函数选择所有的活跃帖子，将它们基于用户名组合起来，然后计算每个用户的发帖量,代码如下：
```
db.posts.mapReduce(
    function() { emit(this.user_id,1); },
    function(key, values) {return Array.sum(values)},
    {
        query:{status:"active"},
        out:"post_total"
    }
)
```
查询输出结果如下：
```
{
    "result" : "post_total",
    "timeMillis" : 9,
    "counts" : {
        "input" : 4,
        "emit" : 4,
        "reduce" : 2,
        "output" : 2
    },
    "ok" : 1,
}
```
结果显示，只有 4 个文档符合查询条件（ status:"active" ），于是 map 函数就生成了 4 个带有键值对的文档，而最终 reduce 函数将具有相同键值的映射文档变为了 2 个
要想查看 mapReduce 查询的结果，使用 find 操作符:
```
db.posts.mapReduce(
    function() { emit(this.user_id,1); },
    function(key, values) {return Array.sum(values)},
    {
        query:{status:"active"},
        out:"post_total"
    }
).find()
```





#### 全文检索

MongoDB 从 2.4 版本起开始支持全文索引，以便搜索字符串内容，文本搜索使用字干搜索技术查找字符串字段中的指定词语，丢弃字干停止词（比如 a、an、the等）

启用文本搜索：`db.adminCommand({setParameter:true,textSearchEnabled:true})`，从 2.6 版本起文本搜索为默认功能


创建文本索引：
假设在 posts 集合中的下列文档中含有帖子文本及其标签：
```
{
    "post_text": "enjoy the mongodb articles on tutorialspoint",
    "tags": [
        "mongodb",
        "tutorialspoint"
    ]
}
```
我们将在 post_text 字段上创建文本索引，以便搜索帖子文本之内的内容:
```
db.posts.ensureIndex({post_text:"text"})
```
搜索包含 tutorialspoint 文本内容的帖子：
```
db.posts.find({$text:{$search:"tutorialspoint"}})

#旧版本查询方法
#db.posts.runCommand("text",{search:" tutorialspoint "})
```
查询文本索引：`db.posts.getIndexes()`
删除文本索引：`db.posts.dropIndex("name")`





#### 正则表达式

使用 `$regex ` 运算符来匹配字符串模式

假如 posts 集合有下面这个文档，它包含着帖子文本及其标签：
```
{
    "post_text": "enjoy the mongodb articles on tutorialspoint",
    "tags": [
        "mongodb",
        "tutorialspoint"
    ]
}
```
使用下列正则表达式来搜索包含 tutorialspoint 的所有帖子:
```
db.posts.find({post_text:{$regex:"tutorialspoint"}})
```
如果正则表达式为 ^tut，那么查询将搜索所有以 tut 开始的字符串


同样的查询也可以写作下列形式：
```
db.posts.find({post_text:/tutorialspoint/})
```

要想使搜索不区分大小写，使用 `$options` 参数和值 `$i` ，下列命令将搜索包含 “tutorialspoint” 的字符串，不区分大小写：
```
db.posts.find({post_text:{$regex:"tutorialspoint",$options:"$i"}})
```

在数组字段上使用正则表达式，假如想搜索标签以 “tutorial”开始（tutorial、tutorials、tutorialpoint 或 tutorialphp）的帖子，可以使用下列代码：
```
db.posts.find({tags:{$regex:"tutorial"}})
```





#### Rockmongo 管理工具



#### 可视化工具 Robo 3t

[下载地址](https://robomongo.org)







#### GridFS

GridFS 是 MongoDB 的一个用来存储/获取大型数据（图像、音频、视频等类型的文件）的规范，相当于一个存储文件的文件系统，但它的数据存储在 MongoDB 的集合中。GridFS 能存储超过文档尺寸限制（16 MB）的文件

GridFS 将文件分解成块，将每块数据保存在不同的文档中，每块大小最高为 255 KB

GridFS 默认使用 fs.files 和 fs.chunks 集合来存储文件的元数据和块。每个块都由唯一的 _id ObjectId 字段来标识，fs.files 用作父级文档，fs.chunks 文档中的 files_id 字段将块连接到父级文档上

fs.files 集合的一个范例文档如下所示，该文档指定了文件名、块大小、上传日期以及长度：
```
{
    "filename": "test.txt",
    "chunkSize": NumberInt(261120),
    "uploadDate": ISODate("2014-04-13T11:32:33.557Z"),
    "md5": "7b762939321e146569b07f72c62cca4f",
    "length": NumberInt(646)
}
```

下面是 fs.chunks 文档的一个范例文档：
```
{
"files_id": ObjectId("534a75d19f54bfec8a2fe44b"),
"n": NumberInt(0),
"data": "Mongo Binary Data"
}
```



使用 put 命令为 GridFS 添加文件，利用 GridFS 存储一个 mp3 文件

该例中，我们将使用 MongoDB 的 bin 文件夹下的 mongofiles.exe 工具，打开命令行提示符，浏览至 mongofiles.exe，输入下列代码：
```
mongofiles.exe -d gridfs put song.mp3
```
上述代码中，gridfs 是保存文件所在的数据库名称，如果数据库不存在，MongoDB 将自动创建一个新数据库，Song.mp3 是上传文件名，要想查看数据库中文件的文档，使用 find() 查询：
```
db.fs.files.find()
```

返回结果文档如下所示：
```
{
_id: ObjectId('534a811bf8b4aa4d33fdf94d'),
filename: "song.mp3",
chunkSize: 261120,
uploadDate: new Date(1397391643474), md5: "e4f53379c909f7bed2e9d631e15c1c41",
length: 10401959
}
```
我们还可以查看 fs.chunks 集合中所有与存储文件相关的块，代码如下，其中使用了上次查询所返回的文档 id：
```
db.fs.chunks.find({files_id:ObjectId('534a811bf8b4aa4d33fdf94d')})
```





#### 固定集合

固定集合 Capped Collection 是一种尺寸固定的“循环”集合，可提供高效的创建、读取、删除等操作，“循环”的意思是，当分配给集合的文件尺寸耗尽时，就会自动开始删除最初的文档，不需要提供任何显式的指令

如果文档更新后增加了文档的尺寸，那么固定集合会限制对文档的更新，因为固定集合按照磁盘存储的顺序来保存文档，所以能确保文档尺寸不会增加磁盘分配的尺寸

固定集合最适合保存日志信息，缓存数据以及任何其他大容量数据

需要使用 createCollection 命令，并将 capped 选项设为 true，同时还需要指定集合的最
大尺寸（以字节计），创建固定集合：
```
db.createCollection("cappedLogCollection",{capped:true,size:10000})
```

除了集合尺寸外，还可以使用 max 参数限制集合中的文档最大数量
```
db.createCollection("cappedLogCollection",{capped:true,size:10000,max:1000})
```

如果想要检查集合是否固定，使用 isCapped 命令即可
```
db.cappedLogCollection.isCapped()
```

如想将现有集合转化为固定集合，使用下列代码：
```
db.runCommand({"convertToCapped":"posts",size:10000})
```



默认情况下，利用 find 查询固定集合，结果会按照插入顺序进行显示，但如果想按相反顺序获取文档，可以使用sort 命令，如下所示：
```
db.cappedLogCollection.find().sort({$natural:-1})
```





关于固定集合，有以下几个非常值得注意的要点：

- [ ] 无法从固定集合中删除文档
- [ ] 固定集合没有默认索引，甚至在 _id 字段中也没有
- [ ] 在插入新的文档时，MongoDB 并不需要寻找磁盘空间来容纳新文档，只是盲目地将新文档插入到集合末尾，这使得固定集合中的插入操作非常快速
- [ ] 同样的，在读取文档时，MongoDB 会按照插入磁盘的顺序来读取文档，从而使读取操作也非常快



#### 自动增长

MongoDB 没有提供类似 SQL 数据库所具有的自动增长功能，默认情况下，MongoDB将 _id 字段（使用 12 字节的 ObjectId）来作为文档的唯一标识，但在有些情况下，我们希望 _id 字段值能够自动增长，而不是固守在 ObjectId 值上

由于这不是 MongoDB 的默认功能，所以使用 `counters` 集合来程序化地实现该功能


假设存在下列文档 products，我们希望 _id 字段值是一个能够自动增长的整数序列（1、2、3、4 …… n）

```
{
"_id":1,
"product_name": "Apple iPhone",
"category": "mobiles"
}
```

创建一个 counters 集合，其中包含了所有序列字段最后的序列值：
```
db.createCollection("counters")
```
现在，将下列文档（productid 是它的键）插入到 counters 集合中，sequence_value 字段保存了序列的最后值：
```
{
    "_id":"productid",
    "sequence_value": 0
}
```

使用下列命令将序列文档插入到 counters 集合中:
```
db.counters.insert({_id:"productid",sequence_value:0})
```

接着创建一个 getNextSequenceValue 函数，该函数以序列名为输入，按照 1 的幅度增加序列数，返回更新的序列数，在该例中，序列名称为 productid
```js
function getNextSequenceValue(sequenceName){
    var sequenceDocument = db.counters.findAndModify({
        query:{_id: sequenceName },
        update: {$inc:{sequence_value:1}},
        new:true
    });
    return sequenceDocument.sequence_value;
}
```

在创建新文档以及将返回的序列值赋予文档的 _id 字段过程中，使用 getNextSequenceValue ：
```
db.products.insert({
"_id":getNextSequenceValue("productid"),
"product_name":"Apple iPhone",
"category":"mobiles"})

db.products.insert({
"_id":getNextSequenceValue("productid"),
"product_name":"Samsung S3",
"category":"mobiles"})
```
使用 find 命令来获取文档，_id 字段为自增：`db.prodcuts.find()`