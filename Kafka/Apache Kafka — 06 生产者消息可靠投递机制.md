## Apache Kafka—生产者消息可靠投递机制



生产者唯一的职责就是发送消息，但是既然有发送，可能就会出现消息丢失，那么**Kafka是如何保证消息的不丢失呢**？
Kafka为了保证数据的可靠性，在生产者层面做了两个实现:

* 一是在存储层面，即为每个partition设置了副本，分leader和follower，发送的时候只发送到leader上，然后leader将消息数据同步到follower；
* 二是在发送层面，Kafka要求只有当leader回复了ack确认收到之后才会完成该条消息的投递，否则会重试。
  



## 一、Producer的ISR机制

在前面介绍了每个topic的分区可以设置若干个副本（Leader、Follower），其中Follower实时同步Leader的数据，为了保证生产者发送的数据，能可靠的发送到指定的 topic，topic 的每个分区收到生产者发送的数据后，都需要向生产者发送应答 ack，如果生产者收到 ack，就会进行下一轮的发送，否则重新发送数据。

在Kafka中要求所有Follower都完成同步时才会发送ack，这样的好处是当重新选举Leader时，只需要有n+1个副本即可容忍n台节点故障，但缺点也很明显就是**延迟很高**，因为必须等待所有follower都完成同步才行。

但这样又会带来新的问题，如果其中有一个follower由于网络延迟或者某种原因，迟迟不能完成数据同步，Leader就会一直阻塞等待直到该follower完成同步，这是非常影响性能的，于是Kafka引入了一个新的机制：**ISR（In-Sync Replica**）。

**ISR（In-Sync Replica）：**Leader 维护了一个动态的 **in-sync replica set** (ISR)，表示和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，就会给 leader 发送 ack。如果 follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由`replica.lag.time.max.ms`参数设定。同时， Leader 发生故障之后，就会从 ISR 中选举新的 Leader。

> **replica.lag.time.max.ms**
>
> **DESCRIPTION**: If a follower hasn't sent any fetch requests or hasn't consumed up to the leaders log end offset for at least this time, the leader will remove the follower from isr
>
> **TYPE**: long
>
> **DEFAULT**: 10000
>
> [Source](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fkafka.apache.org%2F0110%2Fdocumentation%2F%23brokerconfigs)



## 二、Producer的ACK机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接收成功。

所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。

**acks参数配置**

* **acks = 0** ：生产者只负责发消息，不管Leader 和Follower 是否完成落盘就会发送ack 。这样能够最大降低延迟，但当**Leader还未落盘时发生故障就会造成数据丢失**；
* **acks = 1** ：Leader将数据落盘后，不管Follower 是否落盘就会发送ack 。这样可以保证Leader节点内有一份数据，但当**Follower还未同步时Leader发生故障就会造成数据丢失**；
* **acks = -1(all)** ：生产者等待Leader 和ISR 集合内的所有Follower 都完成同步才会发送ack 。但当Follower 同步完之后，broker发送ack之前，Leader发生故障时，此时会重新从ISR内选举一个新的Leader，此时由于生产者没收到ack，于是生产者会重新发消息给新的Leader，此时就**会造成数据重复**；

>
>
>**acks**
>
>**DESCRIPTION**:
>
>The number of acknowledgments the producer requires the leader to have received before considering a request complete. This controls the durability of records that are sent. The following settings are allowed:
>
>- `acks=0` If set to zero then the producer will not wait for any acknowledgment from the server at all. The record will be immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the retries configuration will not take effect (as the client won't generally know of any failures). The offset given back for each record will always be set to -1.
>- `acks=1` This will mean the leader will write the record to its local log but will respond without awaiting full acknowledgement from all followers. In this case should the leader fail immediately after acknowledging the record but before the followers have replicated it then the record will be lost.
>- `acks=all` This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee. This is equivalent to the acks=-1 setting.
>
>**TYPE**:string
>
>**DEFAULT**:1
>
>**VALID VALUES**:[all, -1, 0, 1]
>
>[Source](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fkafka.apache.org%2F0110%2Fdocumentation%2F%23producerconfigs)





## 三、Producer的数据一致性

Kafka里数据有副本，那么必然会出现leader和follower不一致的情况，那么**Kafka又是如何保证数据一致性的呢**？

如下图所示，这里先介绍两个概念：

- **LEO**：Log end offset，即每个副本中的最后一个offset。
- **HW**：High Watermark，所有副本中最小的LEO。

<img src="https://img-blog.csdnimg.cn/20210206195237305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhcnNvbl9DaHU=,size_16,color_FFFFFF,t_70" style="zoom:60%;" />

* **follower故障**：follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后， follower 会读取本地磁盘记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了。

* **leader故障**：leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性， 其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader同步数据。

**注意⚠️：LEO和HW是保证数据一致性的机制，并不能保证数据不丢失或者数据不重复。**



## 四、Producer的Exactly Once

分布式消息传递一致性语义有下面三种：

> - **At least once**—Messages are **never lost** but may be redelivered.
> - **At most once**—Messages **may be lost** but are never redelivered.
> - **Exactly once**—this is what people actually want, each message is delivered once and only once.
>
> [Source](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fkafka.apache.org%2F0110%2Fdocumentation%2F%23semantics)

实际上，在交易业务里，对于精准一次性要求是比较高的，我们绝不能丢失交易数据，也不应该发起重复交易，这就是ExactlyOnce语义。

然而，在Kafka 0.11版本之前是无法在Kafka内实现exactly once 精确一次性保证的语义的，在0.11之后的版本，我们可以结合**幂等性以及acks=-1** 来实现kafka生产者的exactly once。

- **acks=-1**：在上面ack机制里已经介绍过，他能实现`at least once`语义，**保证数据不会丢，但可能会有重复数据**；
- **幂等性**：原本的含义是指用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。0.11版本之后新增的特性，针对生产者，指生产者无论向broker 发送多少次重复数据，broker 只会持久化一条；

要启用幂等性，只需要将 Producer 的参数中 `enable.idempotence` 设置为 true 即可。

> **enable.idempotence**
>
> DESCRIPTION:When set to 'true', the producer will ensure that exactly one copy of each message is written in the stream. If 'false', producer retries due to broker failures, etc., may write duplicates of the retried message in the stream. This is set to 'false' by default. Note that enabling idempotence requires `max.in.flight.requests.per.connection` to be set to 1 and `retries` cannot be zero. Additionally acks must be set to 'all'. If these values are left at their defaults, we will override the default to be suitable. If the values are set to something incompatible with the idempotent producer, a ConfigException will be thrown.
>
> TYPE:boolean
>
> DEFAULT:false
>
> [Source](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fkafka.apache.org%2F0110%2Fdocumentation%2F%23producerconfigs)

 Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的 Producer 在初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而Broker 端会对`<PID, Partition, SeqNumber>`做缓存，当具有相同主键的消息提交时， Broker 只会持久化一条。

但是 PID 在Kafka节点重启后就会变化。同时，不同的 Partition 也具有不同key，所以**幂等性+acks=-1无法保证跨分区跨会话的 Exactly Once**。



## 参考资料

* [csdn.net 【Kafka技术内幕】（三）：Kafka生产者：高可用Producer和ISR机制](https://blog.csdn.net/Carson_Chu/article/details/113728504)
* [oschina.net Kafka学习笔记](https://my.oschina.net/jallenkwong/blog/4449224)

