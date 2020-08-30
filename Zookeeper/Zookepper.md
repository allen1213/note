



Zookeeper=文件系统+通知机制

|                                              |                                             |
| -------------------------------------------- | ------------------------------------------- |
| PERSISTENT-持久化目录节点                    | 客户端与zookeeper断开连接后，该节点依旧存在 |
| PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点 |                                             |
| EPHEMERAL-临时目录节点                       |                                             |
| EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点    |                                             |



### 安装

```bash
# 下载
wget http://mirror.bit.edu.cn/apache/zookeeper/stable/zookeeper-3.4.12.tar.gz

# 解压
tar -zxvf zookeeper-3.4.12.tar.gz

cd zookeeper-3.4.12

# 复制配置文件
cp conf/zoo_sample.cfg conf/zoo.cfg
```



启动zookeeper：

```bash
bin/zkServer.sh start
```



使用客户端连接，检查是否成功启动：

```bash
bin/zkCli.sh
```





解读zoo.cfg 文件中参数含义

1）tickTime：通信心跳数，Zookeeper服务器心跳时间，单位毫秒

Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。

它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)

2）initLimit：LF初始通信时限

集群中的follower跟随者服务器(F)与leader领导者服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。

投票选举新leader的初始化时间

Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。

Leader允许F在initLimit时间内完成这个工作。

3）syncLimit：LF同步通信时限

集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，

Leader认为Follwer死掉，从服务器列表中删除Follwer。

在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。

如果L发出心跳包在syncLimit之后，还没有从F那收到响应，那么就认为这个F已经不在线了。

4）dataDir：数据文件目录+数据持久化路径保存内存数据库快照信息的位置，如果没有其他说明，更新的事务日志也保存到数据库。

5）clientPort：客户端连接端口

监听客户端连接的端口