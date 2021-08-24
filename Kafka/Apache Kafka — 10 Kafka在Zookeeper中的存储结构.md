## Apache Kafka — Kafka在Zookeeper中的存储结构



Kafka集群依赖于Zookeeper，了解Kafka在Zookeeper中的一些存储结构，便于更好的理解Kafka的工作原理，Zookeeper在Kafka中扮演了举足轻重的作用（0.9版本后offest不放在zk上，由Kafka内部topic自己保存），Kafka的 broker、消费者等相关的信息都存在zk的节点上，zk还提供了对Kafka的动态负载均衡等机制。本篇就让我门来详细学习一下Kafka是如何利用Zookeeper进行数据存储的。



### 一、Kafka在ZooKeeper中保存了哪些信息？

![](https://image.easyblog.top/20140923175837147)

#### 1.1 Consumer注册信息

* **注册新的消费者分组**

  当新的消费者组注册到 ZooKeeper 中时，ZooKeeper 会创建专用的节点来保存相关信息，其节点路径为 `/consumers/{group_id}`，其节点下有三个子节点，分别为 `[ids, owners, offsets]`。

  * **ids 节点**：记录该消费组中当前正在消费的消费者；
  * **owners 节点**：记录该消费组消费的 topic 信息；
  * **offsets 节点**：记录每个 topic 的每个分区的 offset。

* **注册新的消费者**

同时，当新的消费者注册到 Kafka 中时，会在 `/consumers/{group_id}/ids ` 节点下创建临时子节点，并记录相关信息。

* **监听消费者分组中消费者的变化**

每个消费者都要关注其所属消费者组中消费者数目的变化，即监听 `/consumers/{group_id}/ids ` 下子节点的变化。一单发现消费者新增或减少，就会触发消费者的负载均衡。


#### 1.2 Broker注册信息



<img src="https://image.easyblog.top/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMzc4MDM0,size_16,color_FFFFFF,t_70.png" style="zoom:80%;" />



**注意⚠️  上图中圆角的矩形是临时节点，直角矩形是持久化节点。**

- 为了记录 broker 的注册信息，在 ZooKeeper 上，专门创建了属于 Kafka 的一个节点，其路径为 `/brokers`
- Kafka 的每个 broker 启动时，都会到 ZooKeeper 中进行注册，告诉 ZooKeeper 其 `broker.id`，在整个集群中，`broker.id` 应该全局唯一，并在 ZooKeeper 上创建其属于自己的节点，其节点路径为 `/brokers/ids/{broker.id}`
- 创建完节点后，Kafka 会将该 broker 的 `broker.name` 及端口号记录到该节点
- 另外**，该 broker 节点属性为临时节点，当 broker 会话失效时，ZooKeeper 会删除该节点**，这样，我们就可以很方便的监控到broker 节点的变化，及时调整负载均衡等。
  

#### 1.3 Topic注册信息

在 Kafka 中，所有 topic 与 broker 的对应关系都由 ZooKeeper 进行维护，在 ZooKeeper 中会建立专门的节点来记录这些信息，其节点路径为 `/brokers/topics/{topic_name}`。

 前面说过，为了保障数据的可靠性，每个 Topic 的 Partitions 实际上是存在备份的，并且备份的数量由 Kafka 机制中的 replicas 来控制。那么问题来了：如下图所示，假设某个 TopicA 被分为 2 个 Partitions，并且存在两个备份，由于这 2 个 Partitions（1-2）被分布在不同的 broker 上，同一个 partiton 与其备份不能（也不应该）存储于同一个 broker 上。以 Partition1 为例，假设它被存储于 broker2，其对应的备份分别存储于 broker1 和 broker4，有了备份，可靠性得到保障，但数据一致性却是个问题。

<img src="https://image.easyblog.top/7161bf30-cb00-11e8-bcac-99cd81fed45b" alt="enter image description here" style="zoom:80%;" />

为了保障数据的一致性，ZooKeeper 得以引入。基于 ZooKeeper 的选举机制，Kafka 为每一个 partition 找一个节点作为 leader，其余备份作为 follower。

如下图所示，就 ` TopicA  partition1 ` 而言，如果位于 broker2（Kafka 节点）上的 partition1 为 leader，那么位于 broker1 和 broker4 上面的 partition1 就充当 follower。

<img src="https://image.easyblog.top/780e1ef0-cb00-11e8-9b13-63a667cc1a24" alt="enter image description here" style="zoom:80%;" />

基于上图的架构，当 producer push 的消息写入 partition 时，作为 leader 的 broker（Kafka 节点） 会将消息写入自己的分区，同时还会将此消息复制到各个 follower 实现同步。如果某个follower 挂掉，leader 会再找一个替代并同步消息；但是，如果 leader 挂了，follower 们会选举出一个新的 leader 替代，继续业务，这些都是由 ZooKeeper 完成的。


#### 1.4 Controller注册信息





#### 1.5 Controller Epoch

我考虑这个应该是需要改的，我这边已经提前改好测好了
