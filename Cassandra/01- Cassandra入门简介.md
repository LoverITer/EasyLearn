## Cassandra入门简介



### 一、Cassandra 简介

**Apache Cassandra**是一套**开源分布式Key-Value存储系统**。它最初由Facebook开发，用于储存特别大的数据。

主要特性：

- **分布式，无中心，无单点故障**
- **基于column的结构化**
- **高伸展性**

Cassandra的主要特点就是它不是一个数据库，而是由一堆数据库节点共同构成的一个分布式网络服务，对Cassandra 的一个写操作，会被复制到其他节点上去，对Cassandra的读操作，也会被路由到某个节点上面去读取。对于一个Cassandra群集来说，扩展性能 是比较简单的事情，只管在群集里面添加节点就可以了。

Cassandra是一个混合型的非关系的数据库，类似于Google的BigTable。其主要功能比 [Dynomite](http://www.oschina.net/p/dynomite)（分布式的Key-Value存 储系统）更丰富，但支持度却不如文档存储[MongoDB](http://www.oschina.net/p/mongodb)（介于关系数据库和非关系数据库之间的开源产品，是非关系数据库当中功能最丰富，最像关系数据库 的。支持的数据结构非常松散，是类似json的bjson格式，因此可以存储比较复杂的数据类型。）Cassandra最初由Facebook开发，后转变成了开源项目。它是一个网络社交云计算方面理想的数据库。以Amazon专有的完全分布式的Dynamo为基础，结合了Google BigTable基于列族（Column Family）的数据模型。P2P去中心化的存储。很多方面都可以称之为Dynamo 2.0。

和其他数据库比较，有几个突出特点：

* **模式灵活** ：使用Cassandra，像文档存储，你不必提前解决记录中的字段。你可以在系统运行时随意的添加或移除字段。这是一个惊人的效率提升，特别是在大型部 署上。
* **真正的可扩展性** ：Cassandra是纯粹意义上的水平扩展。为给集群添加更多容量，可以指向另一台电脑。你不必重启任何进程，改变应用查询，或手动迁移任何数据。
* **多数据中心识别** ：你可以调整你的节点布局来避免某一个数据中心起火，一个备用的数据中心将至少有每条记录的完全复制。

一些使Cassandra提高竞争力的其他功能：

* **范围查询** ：如果你不喜欢全部的键值查询，则可以设置键的范围来查询。
* **列表数据结构** ：在混合模式可以将超级列添加到5维。对于每个用户的索引，这是非常方便的。
* **分布式写操作** ：有可以在任何地方任何时间集中读或写任何数据。并且不会有任何单点失败。



### 二、Cassandra 基本原理和存储数据模型

#### 2.1 Cassandra 相关概念

（1） **keyspace**：Cassandra 的 keyspace 相当于关系型数据的 db，可以设置 replication factor

（2）**column family**：Cassandra 的 column family 相当于关系型数据的 table，但是数据存储和表现完全不同，是Nested stored map

（3） **column**：Cassandra 的 column 相当于关系型数据的 column，列可以无限多，是直接存储数据的地方

#### 2.2 CQL

Cassandra 提供一种叫做 CQL 的查询语言来作为客户端和Cassandra服务器之间的交互，CQL 全称 Cassandra Query Language，和SQL比较类似，主要的区别是Cassandra不支持join或子查询，除了支持通过Hive进行批处理分析。

Cassandra在CQL语言层面支持多种数据类型：

| QL类型    | 对应Java类型      | 描述                                                         |
| --------- | ----------------- | ------------------------------------------------------------ |
| ascii     | String            | ascii字符串                                                  |
| bigint    | long              | 64位整数                                                     |
| blob      | ByteBuffer/byte[] | 二进制数组                                                   |
| boolean   | boolean           | 布尔                                                         |
| counter   | long              | 计数器，支持原子性的增减，不支持直接赋值                     |
| decimal   | BigDecimal        | 高精度小数                                                   |
| double    | double            | 64位浮点数                                                   |
| float     | float             | 32位浮点数                                                   |
| inet      | InetAddress       | [ipv4](http://zh.wikipedia.org/wiki/Ipv4)或[ipv6](http://zh.wikipedia.org/wiki/Ipv6)协议的ip地址 |
| int       | int               | 32位整数                                                     |
| list      | List              | 有序的列表                                                   |
| map       | Map               | 键值对                                                       |
| set       | Set               | 集合                                                         |
| text      | String            | [utf-8](http://zh.wikipedia.org/w/index.php?title=Utf-8&action=edit&redlink=1)编码的字符串 |
| timestamp | Date              | 日期                                                         |
| uuid      | UUID              | [UUID](http://zh.wikipedia.org/wiki/UUID)类型                |
| timeuuid  | UUID              | 时间相关的UUID                                               |
| varchar   | string            | text的别名                                                   |
| varint    | BigInteger        | 高精度整型                                                   |

#### 2.3 数据模型

##### **集群（Cluster）**

Cassandra 数据库分布在几个一起操作的机器上。最外层容器被称为群集。对于故障处理，每个节点包含一个副本，如果发生故障，副本将复制。Cassandra 按照环形格式将节点排列在集群中，并为它们分配数据。

##### 键空间 **（Keyspace） **

键空间是 Cassandra 中数据的最外层容器。Cassandra 中的一个键空间的基本属性是：

- **复制因子** - 它是集群中将接收相同数据副本的计算机数。
- **副本放置策略** - 它只是把副本放在介质中的策略。我们有简单策略（机架感知策略），旧网络拓扑策略（机架感知策略）和网络拓扑策略（数据中心共享策略）等策略。
- **列族** - 键空间是一个或多个列族的列表的容器。列族又是一个行集合的容器。每行包含有序列。列族表示数据的结构。每个键空间至少有一个，通常是许多列族。

创建 keyspace 的语法如下：

```cql
CREATE KEYSPACE crius_dev WITH replication={'class':'SimpleStrategy','replication_factor':3}
```

下图是键空间的示意图。

![KEYSPACE](https://atts.w3cschool.cn/attachments/tuploads/cassandra/keyspace.jpg)

##### 列族（column family）

列族是有序收集行的容器。每一行又是一个有序的列集合。下表列出了区分列系列和关系数据库表的要点。

| 关系表                                                       | Cassandra 列族                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 关系模型中的模式是固定的。 一旦为表定义了某些列，在插入数据时，在每一行中，所有列必须至少填充一个空值。 | 在 Cassandra 中，虽然定义了列族，但列不是。 您可以随时向任何列族自由添加任何列。 |
| 关系表只定义列，用户用值填充表。                             | 在 Cassandra 中，表包含列，或者可以定义为超级列族。          |

Cassandra column family 具有以下属性 - 

- **keys_cached** ：它表示每个 SSTable 保持缓存的位置数。
- **rows_cached** ：它表示其整个内容将在内存中缓存的行数。
- **preload_row_cache** ：它指定是否要预先填充行缓存。

**注 -** 与不是固定列族的模式的关系表不同，Cassandra 不强制单个行拥有所有列。下图显示了 Cassandra column family 的示例。

![卡桑德拉柱族](https://atts.w3cschool.cn/attachments/tuploads/cassandra/cassandra_column_family.jpg)

##### 列 Column

列是 Cassandra 的基本数据结构，具有三个值，即键或列名称，值和时间戳。下面给出了列的结构。

![卡桑德拉结构柱](https://atts.w3cschool.cn/attachments/tuploads/cassandra/cassandra_structure_of_column.jpg)

##### 超级列

超级列是一个特殊列，因此，它也是一个键值对。但是超级列存储了子列的地图。

通常列族被存储在磁盘上的单个文件中。因此，为了优化性能，重要的是保持您可能在同一列族中一起查询的列，并且超级列在此可以有所帮助。下面是超级列的结构。

![卡桑德拉超级列](https://atts.w3cschool.cn/attachments/tuploads/cassandra/cassandra_super_column.jpg)

### 三、Cassandra 和 RDBMS 对比

的数据模型

下表列出了区分 Cassandra 的数据模型和 RDBMS 的数据模型的要点。

| RDBMS                                              | Cassandra                                                    |
| -------------------------------------------------- | ------------------------------------------------------------ |
| RDBMS 处理结构化数据。                             | Cassandra 处理非结构化数据。                                 |
| 它具有固定的模式。                                 | Cassandra 具有灵活的架构。                                   |
| 在 RDBMS 中，表是一个数组的数组。 （ROW x COLUMN） | 在 Cassandra 中，表是“嵌套的键值对”的列表。 （ROW x COLUMN 键 x COLUMN 值） |
| 数据库是包含与应用程序对应的数据的最外层容器。     | Keyspace 是包含与应用程序对应的数据的最外层容器。            |
| 表是数据库的实体。                                 | 表或列族是键空间的实体。                                     |
| Row 是 RDBMS 中的单个记录。                        | Row 是 Cassandra 中的一个复制单元。                          |
| 列表示关系的属性。                                 | Column 是 Cassandra 中的存储单元。                           |
| RDBMS 支持外键的概念，连接。                       | 关系是使用集合表示。                                         |