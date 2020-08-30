`ShardingSphere`  是一个分布式数据库中间件，主要包含 `Sharding JDBC`  `Sharding Proxy`  和 `Sharding Sidecar` 

> 垂直拆分

垂直分表：把表中的一部分字段放一个表中，剩余字段放另一个表中
垂直分库：把单一的数据库按照业务进行划分，如商品单独一个数据库，订单一个数据库

> 水平拆分

水平分表：在同一个数据库中创建多张结构一样的表
水平分库：创建多个相同结构的数据库

#### Sharding JDBC

```yml
```
