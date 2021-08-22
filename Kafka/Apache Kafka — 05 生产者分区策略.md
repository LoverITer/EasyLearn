## Apache Kafka—生产者分区（Partition）策略





## 一、分区的原因

Kafka中Topic的概念，它是承载真实数据的逻辑容器，而在Topic之下还分为若干个分区，也就是说Kafka的消息组织方式实际上是三级结构：**Topic-Partition-Message**。主题下的每条消息只会保存在某一个分区中，而不会在多个分区中被保存多份。官网上的这张图非常清晰地展示了：

![](http://www.louisvv.com/wp-content/uploads/2019/12/20191228112029_11745.png)

**可是，为什么Kafka要做这样的设计？为什么使用分区而不是直接使用多个Topic呢？**

#### 分区的作用

其实，分区的作用就是提供负载均衡的能力，或者说对数据进行分区的主要原因，就是为了实现系统的高伸缩性（Scalability）。

不同的分区能够被放置到不同节点的机器上，而数据的读写操作也都是针对分区这个粒度而进行的，这样每个节点的机器都能独立地执行各自分区的读写请求处理，并且，我们还可以通过添加新的节点机器来增加整体系统的吞吐量

实际上分区的概念以及分区数据库早在1980年就已经有大牛们在做了，比如那时候有个叫Teradata的数据库就引入了分区的概念

在不同的分布式系统对分区的叫法也不尽相同：比如在Kafka中叫分区，在MongoDB和Elasticsearch中就叫分片Shard，而在HBase中则叫Region，在Cassandra中又被称作vnode

从表面看起来，它们实现原理可能不尽相同，但对底层分区（Partitioning）的整体思想却从未改变。

除了提供负载均衡这种最核心的功能之外，利用分区也可以实现其他一些业务级别的需求，比如实现业务级别的消息顺序的问题

**综上所述，分区的主要原因有以下三点**：

* **方便在集群中扩展**，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic又可以有多个 Partition 组成，因此整个集群就可以适应合适的数据了；

* **可以提高并发**，因为能够以 Partition 为单位读写了。
* **方便控制业务级别的消息顺序问题**。



## 二、分区的原则

Kafka中的分区策略，就是**决定生产者将消息发送到哪个分区的算法**。Kafka提供了默认的分区策略，同时，也支持自定义分区策略

- **默认分区策略**
  - **轮询策略（Round-robin）**
  - **随机策略（Randomness）**
  - **hash策略（Key-ordering）**

- 自定义分区策略

### 默认分区策略

#### 1. 轮询策略（默认分区策略）

Kafka的默认分区策略，是按顺序进行分配的。

比如一个主题下有3个分区，那么第一条消息被发送到分区0，第二条被发送到分区1，第三条被发送到分区2，以此类推。当生产第4条消息时又会重新开始，即将其分配到分区0，如下图所示：

<img src="https://upload-images.jianshu.io/upload_images/5850202-3ae926f10b7bc1da.png?imageMogr2/auto-orient/strip|imageView2/2/w/943" style="zoom:67%;" />

✅ 优点：可以提供**非常优秀的负载均衡能力**，可以保证消息被平均分配到所有分区上。

❎ 缺点：**无法保证消息的有序性。**



#### 2.  随机策略

顾名思义，这种策略会随意地将消息放置到任意一个分区上，如下图所示：

<img src="https://upload-images.jianshu.io/upload_images/5850202-4c3c5bc4408dc312.png?imageMogr2/auto-orient/strip|imageView2/2/w/943" style="zoom:67%;" />

✅ 优点：消息的分区选择逻辑简单。

❎ 缺点：负载均衡能力一般，也无法保证消息的有序性



#### 3. hash策略（如果指定了key，默认分区策略为hash策略）

Kafka允许为每条消息定义消息key，这个Key它可以是一个有着明确业务含义的字符串，比如客户代码、部门编号或是业务ID等；也可以用来表征消息元数据，一旦消息被定义了Key，那么**Kafka就可以保证同一个Key的所有消息都进入到相同的分区里面**，由于每个分区下的消息处理都是有顺序的，故这个策略被称为按消息键策略，如下图所示可以根据key来为消息选择分区。

<img src="https:////upload-images.jianshu.io/upload_images/5850202-3040d950bf22049d.png?imageMogr2/auto-orient/strip|imageView2/2/w/943" style="zoom:67%;" />

✅ 优点：可以保证相同key的消息被发送到相同的分区，因此可以**保证相同key的所有消息之间的顺序性。**

❎ 缺点：可能会产生数据倾斜 —— 取决于数据中key的分布，以及使用的hash算法。

**注意⚠️：Kafka在默认分区策略的选择：如果指定了Key，那么默认实现按消息键策略；如果没有指定Key，则使用轮询策略**



### 自定义分区策略

Kafka中如果要自定义分区策略，需要**显式地配置**生产者端的参数**partitioner.class**。具体方法是，在编写生产者程序时，可以编写一个具体的类实现`org.apache.kafka.clients.producer.Partitioner`接口

这个接口也很简单，只定义了两个方法：partition()和close()，通常只需要实现最重要的partition()方法，代码如下所示

![](img/%E6%88%AA%E5%B1%8F2021-08-09%20%E4%B8%8B%E5%8D%885.05.20.png)

这里的topic、key、keyBytes、value和valueBytes都属于消息数据，cluster则是集群信息（比如当前Kafka集群共有多少topic、多少broker等）

Kafka给这么多信息，就是希望让开发者能够充分地利用这些信息对消息进行分区，计算出它要被发送到哪个分区中

只要你自己的实现类定义好了partition方法，同时设置partitioner.class参数为你自己实现类的Full Qualified Name，那么生产者程序就会按照你的代码逻辑对消息进行分区



下面是一个例子：

```java
package top.easyblog.kafka;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;

import java.util.List;
import java.util.Map;
import java.util.Random;

/**
 * 自定义分区策略: key含有order的一部分消息发送到最后一个分区上，其他消息在其他分区随机分配
 *
 * @author frank.huang
 * @date 2021/08/09 17:14
 */
public class PartitionerImp implements Partitioner {

    private Random random;

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value,
                         byte[] valueBytes, Cluster cluster) {
        String keyObj = (String) key;
        List<PartitionInfo> partitionInfoList = cluster.availablePartitionsForTopic(topic);
        int partitionCount = partitionInfoList.size();
        int auditPartition = partitionCount - 1;
        return keyObj == null || "".equals(keyObj) || !keyObj.contains("order") ?
                random.nextInt(partitionCount - 1) : auditPartition;
    }

    @Override
    public void close() {
       //清理工作
    }

    @Override
    public void configure(Map<String, ?> map) {
        //做必要的初始化工作
        random = new Random();
    }
}
```

接下来设置启动类参数，将我们实现的分区策略告诉Kafka

```java
//实现类告诉Kafka
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"top.easyblog.kafka.PartitionerImpl");

String topic = "order-test";
Producer<String,String> producer = new KafkaProducer<String, String>(properties);
ProducerRecord nonKeyRecord = new ProducerRecord(topic,"non-key record");
//这类消息需要放在最后一个分区
ProducerRecord orderRecord = new ProducerRecord(topic,"order","order record");
ProducerRecord nonOrderRecord = new ProducerRecord(topic,"other","other record");

try {
    producer.send(nonOrderRecord).get();
    producer.send(nonOrderRecord).get();
    producer.send(orderRecord).get();
    producer.send(nonOrderRecord).get();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```



## 参考资料

* [louisvv.com 生产者消息分区机制原理](http://www.louisvv.com/archives/2408.html)
* [cnblogs.com Kafka 生产者 自定义分区策略](https://www.cnblogs.com/fubinhnust/p/11967881.html)

