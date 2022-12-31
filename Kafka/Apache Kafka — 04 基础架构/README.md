### Apache Kafka—架构及原理





### 一、 Kafak基础架构

在深入理解Kafka之前，先介绍一下Kafka中的术语。下图展示了Kafka的相关术语以及之间的关系：

<img src="https://image.easyblog.top/8066565-3665cf6f9e1fb3fc.png" alt="img" style="zoom:77%;" />

**1）producer**：生产者即数据的发布者，该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息**追加**到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition中，生产者也可以指定数据存储的partition。

**2）consumer**：消费者可以从broker中读取数据。消费者可以消费多个topic中的数据。

**3）consumer group （CG）**：消费者组，由多个 consumer 组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即**消费者组是逻辑上的一个订阅者**。 

**4）broker**：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker可以容纳多个topic。 

**5）topic**：每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）类似于数据库的表名

**6）partition**：topic中的数据分割为一个或多个partition。每个topic至少有一个partition。每个partition中的数据使用多个segment文件存储。partition中的数据是有序的，不同partition间的数据没有顺序。如果topic有多个partition，消费数据时就不能保证数据的顺序。在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1。

**7）leader**：每个分区（partition）多个副本的master，**leader是当前负责数据的读写的partitionr**。 
**8）follower**：每个分区（partition）多个副本中的salve，follower跟随leader，所有写请求都通过leader路由，数据变更会广播给所有follower，follower与leader保持数据同步。如果leader失效，则从follower中选举出一个新的leader。当follower与leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个follower。

**9）replication **：副本，为保证集群中的某个节点发生故障时，该节点上的partition 数据不丢失，且kafka 仍然能够继续工作，kafka 提供了**副本机制，一个topic 的每个分区都有若干个副本，一个leader 和若干个follower。** 

**10）zookeeper**：kafka集群依赖zookeeper来保存集群的的元信息，来保证系统的可用性。



### 二、Kafka 工作流程及文件存储机制

#### 1. 工作流程

Kafka集群将 Record 流存储在称为 Topic 的类别中，每个记录由一个键、一个值和一个时间戳组成。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210807172417.png)

我们定义Kafka 是一个分布式流平台，这到底是什么意思？

发布和订阅记录流，类似于消息队列或企业消息传递系统。以容错的持久方式存储记录流，处理记录流。

Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，面向的都是同一个 topic。

topic 是逻辑上的概念，而partition 是物理上的概念，每个 partition 对应于一个 log 文件，该 log 文件中存储的就是 producer 生产的数据。Producer 生产的数据会不断追加到该 log 文件末端，且每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个 Offset，以便出错恢复时，从上次的位置继续消费。



#### 2. 存储机制

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210807173855.png)

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了**==分片==**==和**索引**机制，将每个 partition 分为多个 segment。每个 segment 对应两个文件——“.index”文件和“.log”文件。这些文件位于同一文件下，该文件夹的命名规则为：topic 名-分区号。例如，first 这个 topic 有三分分区，则其对应的文件夹为 first-0，first-1，first-2。

​                   