

### 事务消息原理

#### 原理图解

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueR8KYRtfIkFkOyTWEas2kbZicvO9fXliaLJn70sn2bqFH9QElwEXzFaF0zJjp8kEmkfQMv876ZA8JQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



#### 详细过程

`Producer`发送半消息`Half Message`到`broker`，半消息是预发送消息`prepare message`

- `Half Message`发送成功后开始执行本地事务

- 如果本地事务执行成功的话则返回`commit`，如果执行失败则返回`rollback`，这个是在事务消息的回调方法里由开发者自己决定`commit or rollback`

  

`Producer`发送上一步的`commit`还是`rollback`到`broker`，有两种情况：

- [ ] 如果`broker`收到了`commit/rollback`消息 ：

- 如果收到`commit`，则`broker`认为整个事务是没问题的，那么会下发消息给`Consumer`端消费
- 如果收到`rollback`，则`broker`认为本地事务执行失败了，`broker`将会删除`Half Message`，不下发给`Consumer`端



- [ ] 如果`broker`未收到消息：

- `broker`会定时回查本地事务的执行结果：如果回查结果是本地事务已经执行则返回`commit`，若未执行，则返回`rollback`

- `Producer`端回查的结果发送给Broker，Broker接收到的如果是commit，则broker视为整个事务执行成功，如果是rollback，则broker视为本地事务执行失败，broker删除Half Message，不下发给consumer

  如果broker未接收到回查的结果或者查到的是`unkonw`，则broker会定时进行重复回查，以确保查到最终的事务结，重复回查的时间间隔和次数都可配







事务消息是个监听器，有回调函数，回调函数里我们进行业务逻辑的操作，比如给账户-100元，然后发消息到积分的mq里，这时候如果账户-100成功了，且发送到mq成功了，则设置消息状态为commit，这时候broker会将这个半消息发送到真正的topic中。一开始发送他是存到半消息队列里的，并没存在真实topic的队列里，只有确认commit后才会转移



如果事务因为中断，或是其他的网络原因，导致无法立即响应的，RocketMQ当做`UNKNOW`处理，RocketMQ事务消息还提供了一个补救方案：定时查询事务消息的事务状态，这也是一个回调函数，这里面可以做补偿，补偿逻辑开发者自己写，成功的话就返回`commit`





### 事务消息代码实例



```java
import org.apache.rocketmq.client.producer.LocalTransactionState;
import org.apache.rocketmq.client.producer.TransactionListener;
import org.apache.rocketmq.client.producer.TransactionMQProducer;
import org.apache.rocketmq.client.producer.TransactionSendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.Date;

public class ProducerTransaction2 {
    public static void main(String[] args) throws Exception {
        TransactionMQProducer producer = new TransactionMQProducer("my-transaction-producer");
        producer.setNamesrvAddr("124.57.180.156:9876");

        // 回调
        producer.setTransactionListener(new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message message, Object arg) {
                LocalTransactionState state = null;
                //msg-4返回COMMIT_MESSAGE
                if(message.getKeys().equals("msg-1")){
                    state = LocalTransactionState.COMMIT_MESSAGE;
                }
                //msg-5返回ROLLBACK_MESSAGE
                else if(message.getKeys().equals("msg-2")){
                    state = LocalTransactionState.ROLLBACK_MESSAGE;
                }else{
                    //这里返回unknown的目的是模拟执行本地事务突然宕机的情况（或者本地执行成功发送确认消息失败的场景）
                    state = LocalTransactionState.UNKNOW;
                }
                System.out.println(message.getKeys() + ",state:" + state);
                return state;
            }

            /**
             * 事务消息的回查方法
             */
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                if (null != messageExt.getKeys()) {
                    switch (messageExt.getKeys()) {
                        case "msg-3":
                            System.out.println("msg-3 unknow");
                            return LocalTransactionState.UNKNOW;
                        case "msg-4":
                            System.out.println("msg-4 COMMIT_MESSAGE");
                            return LocalTransactionState.COMMIT_MESSAGE;
                        case "msg-5":
                            //查询到本地事务执行失败，需要回滚消息。
                            System.out.println("msg-5 ROLLBACK_MESSAGE");
                            return LocalTransactionState.ROLLBACK_MESSAGE;
                    }
                }
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });

        producer.start();

        //模拟发送5条消息
        for (int i = 1; i < 6; i++) {
            try {
                Message msg = new Message("transactionTopic", null, "msg-" + i, ("测试，这是事务消息！ " + i).getBytes());
                producer.sendMessageInTransaction(msg, null);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



输出结果：

```
msg-1,state:COMMIT_MESSAGE
msg-2,state:ROLLBACK_MESSAGE
msg-3,state:UNKNOW
msg-4,state:UNKNOW
msg-5,state:UNKNOW

msg-3 unknow
msg-3 unknow
msg-5 ROLLBACK_MESSAGE
msg-4 COMMIT_MESSAGE

msg-3 unknow
msg-3 unknow
msg-3 unknow
msg-3 unknow
```



结果分析：

- 只有`msg-1`和`msg-4`发送成功了，`msg-4`在`msg-1`前面是因为`msg-1`先成功的，`msg-4`是回查才成功的

- 先来输出五个结果，对应五条消息

  ```
  msg-1,state:COMMIT_MESSAGE
  msg-2,state:ROLLBACK_MESSAGE
  msg-3,state:UNKNOW
  msg-4,state:UNKNOW
  msg-5,state:UNKNOW
  ```



- 然后进入了回查，`msg-3`还是`unknow`，`msg-5`回滚了，`msg-4`提交了事务
- 过了一段时间再次回查`msg-3`，发现还是`unknow`，所以一直回查，回查的时间间隔和次数都是可配的，默认是回查15次还失败的话就会把这个消息丢掉了



**Spring事务、常规的分布式事务不行吗？Rocketmq的事务是否多此一举了**

MQ用于解耦，之前是分布式事务直接操作了两个系统，但是他两就是强耦合的存在，如果中间插了个mq，账号系统操作完发消息到mq，这时候只要保证发送成功就提交，发送失败则回滚，就是靠事务了，用RocketMQ做分布式事务的也蛮多的





### 顺序消息



RocketMQ的消息是存储到Topic的`queue`里面的，`queue`本身是FIFO先进先出队列，所以单个queue是可以保证有序性的



但问题是1个topic有N个queue，支持集群和负载均衡的特性，将海量数据均匀分配到各个queue上，发了10条消息到同一个topic上，这10条消息会自动分散在topic下的所有queue中，所以消费的时候不一定是先消费哪个queue，后消费哪个queue，这就导致了无序消费

#### 图解



![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueR8KYRtfIkFkOyTWEas2kbcvThU2icZNfskYGshD6A2KzWRExVkjQfMiaw3BT2IEoY5icSgQmfN4I5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





一个Producer发送了`m1、m2、m3、m4`四条消息到topic上，topic有四个队列，由于自带的负载均衡策略，四个队列上分别存储了一条消息，queue1上存储的m1，queue2上存储的m2，queue3上存储的m3，queue4上存储的m4



Consumer消费的时候是多线程消费，所以无法保证先消费哪个队列或者哪个消息，比如发送的时候顺序是m1，m2，m3，m4，但是消费的时候由于Consumer内部是多线程消费的，所以可能先消费了queue4队列上的m4，然后才是m1，这就导致了无序





### 顺序消息解决方案



#### 方案一



使用`MessageQueueSelector` 类，将消息都发送到同一个队列queue



生产者：

```java
import java.util.List;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;

/**
 * 消息发送者
 */
public class Producer5 {
    public static void main(String[] args)throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("my-order-producer");
        producer.setNamesrvAddr("124.57.180.156:9876");     
        producer.start();
        for (int i = 0; i < 5; i++) {
            Message message = new Message("orderTopic", ("hello!" + i).getBytes());
            producer.send(
                    // 要发的那条消息
                    message,
                    // queue 选择器 ，向 topic中的哪个queue去写消息
                    new MessageQueueSelector() {
                        // 手动 选择一个queue
                        @Override
                        public MessageQueue select(
                                // 当前topic 里面包含的所有queue
                                List<MessageQueue> mqs, 
                                // 具体要发的那条消息
                                Message msg,
                                // 对应到 send（） 里的 args，也就是2000前面的那个0
                                // 实际业务中可以把0换成实际业务系统的主键，比如订单号啥的，然后这里做hash进行选择queue等。能做的事情很多，我这里做演示就用第一个queue，所以不用arg。
                                Object arg) {
                            // 向固定的一个queue里写消息，比如这里就是向第一个queue里写消息
                            MessageQueue queue = mqs.get(0);
                            // 选好的queue
                            return queue;
                        }
                    },
                    // 自定义参数：0
                    // 2000代表2000毫秒超时时间
                    0, 2000);
        }
    }
}
```



消费者：

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.*;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;


public class ConsumerOrder {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("my-consumer");
        consumer.setNamesrvAddr("124.57.180.156:9876");
        consumer.subscribe("orderTopic", "*");
        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                for (MessageExt msg : msgs) {
                    System.out.println(new String(msg.getBody()) + " Thread:" + Thread.currentThread().getName() + " queueid:" + msg.getQueueId());
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.println("Consumer start...");
    }
}
```



若有如下新需求：把未支付的订单都放到queue1里，已支付的订单都放到queue2里，支付异常的订单都放到queue3里，然后你消费的时候要保证每个queue是有序的，不能消费queue1一条直接跑到queue2去了，要逐个queue去消费



这时候思路是发消息的时候利用自定义参数arg，消息体里肯定包含支付状态，判断是未支付的则选择queue1，以此类推，这样就保证了每个queue里只包含同等状态的消息，消费者目前是多线程消费的，三个queue随机消费肯定乱序，解决方案直接将消费端的线程数改为1个，这样就逐个消费了：

```java
// 最大线程数1
consumer.setConsumeThreadMax(1);
// 最小线程数
consumer.setConsumeThreadMin(1);
```











