###  Apache Kafka—基础架构





### 1. Kafak基础架构

在深入理解Kafka之前，先介绍一下Kafka中的术语。下图展示了Kafka的相关术语以及之间的关系：

<img src="https://image.easyblog.top/8066565-3665cf6f9e1fb3fc.png" alt="img" style="zoom:77%;" />

**1）Producer**：消息生产者，比如后端业务系统

**2）Consumer Group （CG）**：消费组是一个**逻辑上的概念**，它将旗下的消费者归为一类，每一个消费者只隶属于一个消费组。每一个消费组都会有一个固定的名称，消费者在进行消费前需要指定其所属消费组的名称，这个可以通过消费者客户端参数 group.id 来配置，默认值为空字符串。 

**3）Consumer**：消息消费者，消费者并非逻辑上的概念，它是实际的应用实例，它可以是一个线程，也可以是一个进程。同一个消费组内的消费者既可以部署在同一台机器上，也可以部署在不同的机器上。

**4）Broker**：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker可以容纳多个topic。 

**5）Topic**：可以理解为一个队列，**生产者和消费者面向的都是topic**

**6）Partition**：为了实现扩展性，一个非常大的 topic 可以分布到多个broker（即服务器）上，一个topic 可以分为多个partition，每个partition 是一个有序的队列； 

**7）Replication **：副本，为保证集群中的某个节点发生故障时，该节点上的partition 数据不丢失，且kafka 仍然能够继续工作，kafka 提供了**副本机制，一个topic 的每个分区都有若干个副本，一个leader 和若干个follower。** 

**8）leader**：每个分区多个副本的master，**生产者发送数据的对象，以及消费者消费数据的对象都是leader**。 
**9）follower**：每个分区多个副本中的salve，**实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 leader。**

