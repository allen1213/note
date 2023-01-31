

MySQL `binlog` 是一个二进制文件，保存在磁盘中，用来记录数据库表结构变更及表数据修改的二进制日志

`binlog` 除了数据复制外，还可以实现数据恢复，增量备份等功能



查看MySQL是否已经开启binlog：

```sql
show variables like 'log_bin';
```



若值为OFF，表示没有启动，需要修改配置：

```shell
# 在配置文件中加入了log_bin配置项后，表示启用了binlog
log_bin=mysql-bin

# 日志格式，支持三种类型，分别是STATEMENT、ROW、MIXED，在这里使用ROW模式
binlog-format=ROW

# server-id用于标识一个sql语句是从哪一个server写入的，这里一定要进行设置，否则后面的代码中会无法正常监听到事件
server-id=1
```



更改完配置文件后，重启`mysql`服务，再次查看是否启用`binlog`，返回为`ON`，表示已经开启成功





### MySQL 主从同步原理

MySQL的主从复制中主要有3个线程：

1. master节点：binlog dump thread
2. slave节点：IO thread，SQL thread

binlog 是主从复制的基础，master节点log dump 线程读取binlog 以及偏移量，从偏移量位置开始并发送给slave节点，slave节点开启IO 线程接收数据并写入到 relay log中，slave节点开启SQL线程读取relay log更新数据保持主从数据一致



MySQL默认的复制方式是异步的，master将binlog发送给slave后，并不关心slave是否已经处理成功，若此时master宕机slave处理失败，slave成为新的master后，原master的日志就会丢失，因此产生了全同步复制与半同步复制

#### 全同步复制

所有slave节点都执行完成后，返回ACK确认给master，才算写操作完成

#### 半同步复制

slave节点处理完成后返回ACK确认给master，master收到其中一个确认就认为写操作完成





















