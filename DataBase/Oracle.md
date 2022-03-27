https://mrbird.cc/Oracle-SQL%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0.html





### Oracle 递归查询

在 Oracle 中是通过 `start with connect by prior` 语法来实现递归查询的，

```sql
# 查询 id 为 1001 下的所有部门，包括1001，pid为父id
select * from dept start with id = '1001' connet by prior id = pid;

# 查询 id 为 1001 下的所有部门，不包括1001，pid为父id
select * from dept start with pid = '1001' connect by prior id = pid;


# 将 prior 后的字段反过来，则表示向上查询父节点，查询结果包括自己及其所有父节点
select * from dept start with id = '1001' connect by prior pid = id;

# 对当前节点的第一代子节点及其父节点递归查询，查询结果包括自己的第一代子节点以及所有父节点
select * from dept start with pid='1001' connect by prior pid=id;

```

