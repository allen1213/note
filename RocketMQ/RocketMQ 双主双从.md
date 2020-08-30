



分别在两台服务器上安装RocketMQ，角色如下：

- [ ] 172.17.160.28 ： namesrv、broker-a-master、broker-b-slave 
- [ ] 172.17.160.29 ： namesrv、broker-b-master、broker-a-slave 



分别启动namesrv，172.17.160.28和172.17.160.29交叉作为彼此的slave，参数介绍：

- [ ] `brokerClusterName`：集群名称，所有broker配置的必须一致
- [ ] `brokerName`：broker名称，M-S之间必须一致，比如broker-a/broker-a（M/S），broker-b/broker-b
- [ ] `brokerId`：0 表示 Master，>0 表示 Slave，比如1M2S，那么M是0，Slave1是1，Slave2配置为2
- [ ] `namesrvAddr：namesrv`地址
- [ ] `store`：持久化数据存储目录
- [ ] `brokerRole：Broker`的角色
- [ ] `flushDiskType`：刷盘方式，2m2s-sync中，master默认为SYNC_FLUSH，slave默认为ASYNC_FLUSH



### Master1

```bash
cd /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync

vi broker-a.properties
```

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，需要注意的是slave节点需要和master节点的brokerName一致，区分m还是s交由下面的brokerId来配置。
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=172.17.160.28:9876;172.17.160.29:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#存储路径
storePathRootDir=/home/chentongwei/data/rocketmq/broker-a/store
#commitLog 存储路径
storePathCommitLog=/home/chentongwei/data/rocketmq/broker-a/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/home/chentongwei/data/rocketmq/broker-a/store/consumequeue
#消息索引存储路径
storePathIndex=/home/chentongwei/data/rocketmq/broker-a/store/index
#checkpoint 文件存储路径
storeCheckpoint=/home/chentongwei/data/rocketmq/broker-a/store/checkpoint
#abort 文件存储路径
abortFile=/home/chentongwei/data/rocketmq/broker-a/store/abort
#限制的消息大小
maxMessageSize=65536
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
```



### Slave1

```bash
vi broker-a-s.properties
```

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，需要注意的是slave节点需要和master节点的brokerName一致，区分m还是s交由下面的brokerId来配置。
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
# 注意此处是1了，不在是0了，因为0代表Master
brokerId=1
#nameServer地址，分号分割
namesrvAddr=172.17.160.28:9876;172.17.160.29:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#存储路径
storePathRootDir=/home/chentongwei/data/rocketmq/broker-a/store
#commitLog 存储路径
storePathCommitLog=/home/chentongwei/data/rocketmq/broker-a/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/home/chentongwei/data/rocketmq/broker-a/store/consumequeue
#消息索引存储路径
storePathIndex=/home/chentongwei/data/rocketmq/broker-a/store/index
#checkpoint 文件存储路径
storeCheckpoint=/home/chentongwei/data/rocketmq/broker-a/store/checkpoint
#abort 文件存储路径
abortFile=/home/chentongwei/data/rocketmq/broker-a/store/abort
#限制的消息大小
maxMessageSize=65536
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
```





### Master2

```bash
vi broker-b.properties
```

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，需要注意的是slave节点需要和master节点的brokerName一致，区分m还是s交由下面的brokerId来配置。
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=172.17.160.28:9876;172.17.160.29:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#存储路径
storePathRootDir=/home/chentongwei/data/rocketmq/broker-b/store
#commitLog 存储路径
storePathCommitLog=/home/chentongwei/data/rocketmq/broker-b/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/home/chentongwei/data/rocketmq/broker-b/store/consumequeue
#消息索引存储路径
storePathIndex=/home/chentongwei/data/rocketmq/broker-b/store/index
#checkpoint 文件存储路径
storeCheckpoint=/home/chentongwei/data/rocketmq/broker-b/store/checkpoint
#abort 文件存储路径
abortFile=/home/chentongwei/data/rocketmq/broker-b/store/abort
#限制的消息大小
maxMessageSize=65536
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
```





### Slave2

```bash
vi broker-b-s.properties
```

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，需要注意的是slave节点需要和master节点的brokerName一致，区分m还是s交由下面的brokerId来配置。
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
# 注意此处是1了，不在是0了，因为0代表Master
brokerId=1
#nameServer地址，分号分割
namesrvAddr=172.17.160.28:9876;172.17.160.29:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#存储路径
storePathRootDir=/home/chentongwei/data/rocketmq/broker-b/store
#commitLog 存储路径
storePathCommitLog=/home/chentongwei/data/rocketmq/broker-b/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/home/chentongwei/data/rocketmq/broker-b/store/consumequeue
#消息索引存储路径
storePathIndex=/home/chentongwei/data/rocketmq/broker-b/store/index
#checkpoint 文件存储路径
storeCheckpoint=/home/chentongwei/data/rocketmq/broker-b/store/checkpoint
#abort 文件存储路径
abortFile=/home/chentongwei/data/rocketmq/broker-b/store/abort
#限制的消息大小
maxMessageSize=65536
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
```





### 运行

启动两台Namesrv：

```bash
cd /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/bin
nohup sh mqnamesrv &
```



启动broker：

```bash
cd /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/bin
# 启动Master1
nohup sh mqbroker -c /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a.properties &

# 启动Slave2
nohup sh mqbroker -c /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b-s.properties &
```

```bash
cd /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/bin
# 启动Master2
nohup sh mqbroker -c /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-b.properties &

# 启动Slave1
nohup sh mqbroker -c /home/chentongwei/rocketmq/rocketmq-all-4.7.0-bin-release/conf/2m-2s-sync/broker-a-s.properties &
```



打开管控台查看Cluster集群信息





### Dledger

Rocket MQ在4.5.0之前版本支持的主从不支持自动切换，1个M挂了后，其他Slave节点不会自动升级为Master，4.5.0之后使用Dledger让Rocket MQ做到高可用，Master挂了后Slave自动升级为Master



Dledger 和 Redis 的哨兵一样基于 raft，Dledger要求机器数必须是3台或3台以上，这也是raft的基础要求，因此需要多开一台虚拟机，采取Dledger搭建一组1M2S的集群：[官方搭建手册](https://github.com/apache/rocketmq/blob/master/docs/cn/dledger/deploy_guide.md)

- [ ] 172.17.160.28 ： namesrv、broker-a-master 
- [ ] 172.17.160.29 ： namesrv、broker-a-slave1 
- [ ] 172.17.160.30 ： namesrv、broker-a-slave2 



在上面2M2S配置不变的基础上，在每个properties里新增如下配置：

```properties
#是否启动 DLedger
enableDLegerCommitLog=true
#DLedger Raft Group的名字，建议和 brokerName 保持一致
dLegerGroup=broker-a
#DLedger Group 内各节点的端口信息，同一个 Group 内的各个节点配置必须要保证一致
dLegerPeers=n0-172.17.160.28:40911;n1-172.17.160.29:40911;n2-172.17.160.30:40911
#节点 id, 必须属于 dLegerPeers 中的一个；同 Group 内各个节点要唯一
dLegerSelfId=n0
#发送消息的线程数，建议与CPU核数设置成一样的
sendMessageThreadPoolNums=4
```

```properties
enableDLegerCommitLog=true
dLegerGroup=broker-a
dLegerPeers=n0-172.17.160.28:40911;n1-172.17.160.29:40911;n2-172.17.160.30:40911
#节点 id, 必须属于 dLegerPeers 中的一个；同 Group 内各个节点要唯一
dLegerSelfId=n1
sendMessageThreadPoolNums=4
```

```properties
enableDLegerCommitLog=true
dLegerGroup=broker-a
dLegerPeers=n0-172.17.160.28:40911;n1-172.17.160.29:40911;n2-172.17.160.30:40911
#节点 id, 必须属于 dLegerPeers 中的一个；同 Group 内各个节点要唯一
dLegerSelfId=n2
sendMessageThreadPoolNums=4
```



管控台去看的话应该是1个master，两个slave，都叫broker-a，这时候停掉master，会发现其中一个slave自动升级为Master



### 常见问题

- [ ] Lock failed,MQ already started

  ```bash
   sh ../../bin/mqbroker -c broker-b-s.properties
   
  java.lang.RuntimeException: Lock failed,MQ already started
          at org.apache.rocketmq.store.DefaultMessageStore.start(DefaultMessageStore.java:223)
          at org.apache.rocketmq.broker.BrokerController.start(BrokerController.java:853)
          at org.apache.rocketmq.broker.BrokerStartup.start(BrokerStartup.java:64)
          at org.apache.rocketmq.broker.BrokerStartup.main(BrokerStartup.java:58)
  ```

  解决方法：

  这个是由于没配置`store*`导致的，一台机器上部署两个broker，一个broker-a-master，一个broker-b-slave，都采取默认的`store*`配置，冲突了，所以报错，手动配上不同目录即可

  

- [ ] AllocateMappedFileService started:false lastThread:null

  这个也是由于store*配置的都一样导致的



- [ ] Address already in use

  在单台机器上配置了broker-a-master的端口为默认端口10911，broker-b-slave的端口为10912，但是Rocket默认启动了一个进程，端口号就是配置的端口+1，所以启动一个broker后他其实是给我们启动了两个端口，一个原端口，一个原端口+1，10912这个端口就会冲突

  源码如下：

  ```java
  // {@link org.apache.rocketmq.broker.BrokerStartup#createBrokerController(String[])}
  messageStoreConfig.setHaListenPort(nettyServerConfig.getListenPort() + 1);
  ```

  

- [ ] 管控台配置namesrv

  如果namesrv和broker都启动正常后，管控台上还是没有集群的话，可以检查下管控台是否配置了namesrv，配置完之后需要回车+点击update才能生效

