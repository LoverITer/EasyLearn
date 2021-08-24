## Apache Kafka — 消费者分区分配策略

在前面的教程文章里，我们逐一分析了Kafka的**工作流程、存储机制、分区策略、数据可靠性以及一致性保证**。从而弄清楚了Kafka的整体架构以及生产者生产的数据是怎么存储，保证可靠性以及Kafka是如何保证数据一致性的。

接下来这篇教程里，我们将分析Kafka架构里的消费者是如何工作的，本文将重点分析Kafka消费者**三种分区分配策略（RangeAssignor策略、RoundRobinAssignor策略、StickyAssignor策略）** 。



一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及到 partition 的分配问题，即确定那个 partition 由哪个 consumer 来消费。

Kafka 提供了三种种分配策略：

* **RangeAssignor策略**
* **RoundRobinAssignor策略**
* **StickyAssignor策略**

Kafka提供了消费者客户端参数 `partition.assignment.strategy` 用来设置消费者与订阅主题之间的分区分配策略。默认情况下，此参数的值为：`org.apache.kafka.clients.consumer.RangeAssignor`，即采用 RangeAssignor 分配策略。除此之外，Kafka中还提供了另外两种分配策略： RoundRobinAssignor和StickyAssignor。消费者客户端参数partition.asssignment.strategy 可以配置多个分配策略，彼此之间以逗号分隔。

### 一、RangeAssignor 分配策略（默认分配策略）

![](https://image.easyblog.top/1402378-20181210165325890-1633597010.png)

RangeAssignor是**对每个Topic而言**的（即一个Topic一个Topic的分），首先对同一个Topic里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。然后用Partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区。

假设：`n=分区数/消费者数量`，`m=分区数%消费者数量`，

那么：**前m个消费者每个分配n+1个分区**，**后面的`消费者数量-m`个消费者每个分配n个分区**。

假如有10个分区，3个消费者线程，把分区按照序号排列0，1，2，3，4，5，6，7，8，9；消费者线程为C1，C2，C3，那么用partition数除以消费者线程的总数来决定每个消费者线程消费几个partition，如果除不尽，前面几个消费者将会多消费一个分区。在我们的例子里面，我们有10个分区，3个消费者线程，10/3 = 3，而且除除不尽，那么消费者线程C1将会多消费一个分区，所以最后分区分配的结果看起来是这样的：

```tex
C1：0，1，2，3
C2：4，5，6
C3：7，8，9
```

如果有分区数增加到11将会是：

```te
C1：0,1,2,3
c2：4,5,6,7
c3：8,9,10
```

假如我们有两个主题T1,T2，分别有10个分区，最后的分配结果将会是这样：

```te
C1：T1（0，1，2，3） T2（0，1，2，3）
C2：T1（4，5，6） T2（4，5，6）
C3：T1（7，8，9） T2（7，8，9）
```

可以看出， C1消费者线程比其他消费者线程多消费了2个分区

如上，只是针对 1 个 topic 而言，C1消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1都将多消费 1 个分区，topic越多，C1消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了。



### 二、RoundRobinAssignor 分配策略

![](https://image.easyblog.top/1402378-20181210164657270-1615029320.png)

RoundRobinAssignor分配策略的原理是**将消费组内所有消费者以及消费者所订阅的所有topic的partition按照字典序排序，然后通过轮询方式逐个将分区以此分配给每个消费者**。round-robin分配策略对应的`partition.assignment.strategy`参数值为：`org.apache.kafka.clients.consumer.RoundRobinAssignor`。

同时，使用RoundRobinAssignor策略有两个前提条件必须满足：

1. **同一个消费者组里面的所有消费者的num.streams（消费者消费线程数）必须相等**；
2. **每个消费者订阅的主题必须相同**。



### 三、StickyAssignor 策略

我们再来看一下StickyAssignor策略，“sticky”这个单词可以翻译为“粘性的”，Kafka从 0.11.x 版本开始引入这种分配策略，它主要有两个目的：

* **分区的分配要尽可能的均匀，分配给消费者者的主题分区数最多相差一个**；

* **分区的分配尽可能的与上次分配的保持相同**。

当两者发生冲突时，第一个目标优先于第二个目标。鉴于这两个目标，StickyAssignor策略的具体实现要比RangeAssignor和RoundRobinAssignor这两种分配策略要复杂很多。我们举例来看一下StickyAssignor策略的实际效果。

假设消费组内有3个消费者：C0、C1和C2，它们都订阅了4个主题：t0、t1、t2、t3，并且每个主题有2个分区，也就是说整个消费组订阅了t0p0、t0p1、t1p0、t1p1、t2p0、t2p1、t3p0、t3p1这8个分区。最终的分配结果如下：

```none
消费者C0：t0p0、t1p1、t3p0
消费者C1：t0p1、t2p0、t3p1
消费者C2：t1p0、t2p1
```

这样初看上去似乎与采用RoundRobinAssignor策略所分配的结果相同，但事实是否真的如此呢？

此时假设消费者C1脱离了消费组，那么消费组就会执行再平衡操作，进而消费分区会重新分配。如果采用RoundRobinAssignor策略，那么此时的分配结果如下：

```none
消费者C0：t0p0、t1p0、t2p0、t3p0
消费者C2：t0p1、t1p1、t2p1、t3p1
```

如分配结果所示，RoundRobinAssignor策略会按照消费者C0和C2进行重新轮询分配。而如果此时使用的是StickyAssignor策略，那么分配结果为：

```none
消费者C0：t0p0、t1p1、t3p0、t2p0
消费者C2：t1p0、t2p1、t0p1、t3p1
```

可以看到分配结果中**保留了上一次分配中对于消费者C0和C2的所有分配结果**，并将原来消费者C1的“负担”分配给了剩余的两个消费者C0和C2，最终C0和C2的分配还保持了均衡。

如果发生分区重分配，那么对于同一个分区而言有可能之前的消费者和新指派的消费者不是同一个，对于之前消费者进行到一半的处理还要在新指派的消费者中再次复现一遍，这显然很浪费系统资源。StickyAssignor策略如同其名称中的“sticky”一样，让分配策略具备一定的“粘性”，尽可能地让前后两次分配相同，进而减少系统资源的损耗以及其它异常情况的发生。

到目前为止所分析的都是消费者的订阅信息都是相同的情况，我们来看一下**订阅信息不同**的情况下的处理。

举例，同样消费组内有3个消费者：C0、C1和C2，集群中有3个主题：t0、t1和t2，这3个主题分别有1、2、3个分区，也就是说集群中有t0p0、t1p0、t1p1、t2p0、t2p1、t2p2这6个分区。消费者C0订阅了主题t0，消费者C1订阅了主题t0和t1，消费者C2订阅了主题t0、t1和t2。

如果此时采用RoundRobinAssignor策略，那么最终的分配结果如下所示（和讲述RoundRobinAssignor策略时的一样，这样不妨赘述一下）：

```none
消费者C0：t0p0
消费者C1：t1p0
消费者C2：t1p1、t2p0、t2p1、t2p2
```

如果此时采用的是StickyAssignor策略，那么最终的分配结果为：

```none
消费者C0：t0p0
消费者C1：t1p0、t1p1
消费者C2：t2p0、t2p1、t2p2
```

可以看到这是一个最优解（消费者C0没有订阅主题t1和t2，所以不能分配主题t1和t2中的任何分区给它，对于消费者C1也可同理推断）。

假如此时消费者C0脱离了消费组，那么RoundRobinAssignor策略的分配结果为：

```none
消费者C1：t0p0、t1p1
消费者C2：t1p0、t2p0、t2p1、t2p2
```

可以看到RoundRobinAssignor策略保留了消费者C1和C2中原有的3个分区的分配：t2p0、t2p1和t2p2（针对结果集1）。而如果采用的是StickyAssignor策略，那么分配结果为：

```none
消费者C1：t1p0、t1p1、t0p0
消费者C2：t2p0、t2p1、t2p2
```

可以看到StickyAssignor策略保留了消费者C1和C2中原有的5个分区的分配：t1p0、t1p1、t2p0、t2p1、t2p2。

从结果上看StickyAssignor策略比另外两者分配策略而言显得更加的优异，这个策略的代码实现也是异常复杂。