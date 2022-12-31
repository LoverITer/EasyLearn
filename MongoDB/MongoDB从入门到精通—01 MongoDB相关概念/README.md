### MongoDB从入门到精通—01 MongoDB相关概念



### 一、业务应用场景

传统的关系型数据库（如：MySQL）,在数据操作的“三高”需求以及应用对web2.0的网站需求面前，显得力不从心。具体表现在以下三方面：

* High performance：对数据库高并发读写的需求
* Huge Storage：对海量数据的高效存储和访问的需求
* High Scalability & High Avaliability：对数据库的高可扩展性和高可用性的需求

**而MongoDB可以应对“三高”需求**，具体的应用场景如：

* 1）社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能。
* 2）游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、高效率存储和访问。
* 3）物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将 订单所有的变更读取出来。 
* 4）物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析。 
* 5）视频直播，使用 MongoDB 存储用户信息、点赞互动信息等。 

这些应用场景中，数据操作方面的共同特点是： 

* （1）**数据量大**
* （2）**写入操作频繁（读写都很频繁）** 
* （3）**价值较低的数据，对事务性要求不高** 

对于这样的数据，我们更适合使用MongoDB来实现数据的存储。

#### 什么时候选择MongoDB 

在架构选型上，除了上述的三个特点外，如果你还犹豫是否要选择它？可以考虑以下的一些问题： 

* 应用不需要事务及复杂 join 支持 
* 新应用，需求会变，数据模型无法确定，想快速迭代开发 
* 应用需要2000-3000以上的读写QPS（更高也可以） 
* 应用需要TB甚至 PB 级别数据存储 
* 应用发展迅速，需要能快速水平扩展 应用要求存储的数据不丢失 
* 应用需要99.999%高可用 
* 应用需要大量的地理位置查询、文本查询

如果上述有1个符合，可以考虑 MongoDB，2个及以上的符合，选择 MongoDB 绝不会后悔。

### 二、MongoDB 简介

#### 2.1 什么是MongoDB?

[MongoDB](https://www.mongodb.com/) 是一个**开源、高性能、无模式的文档型数据库**，当初的设计就是用于简化开发和方便扩展，是NoSQL数据库产品中的一种。是最像关系型数据库（MySQL）的非关系型数据库。 

它支持的数据结构非常松散，是一种类似于 JSON 的 格式叫 **BSON**，所以它**既可以存储比较复杂的数据类型，又相当的灵活**。 

MongoDB中的记录是一个文档，它是一个由字段和值对（`field:value`）组成的数据结构。MongoDB文档类似于JSON对象，即一个文档认 为就是一个对象。字段的数据类型是字符型，它的值除了使用基本的一些类型外，还可以包括其他文档、普通数组和文档数组。



#### 2.2 MongoDB 的 体系结构

MySQL和MongoDB对比

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220411220453.png" style="width:65%;" />



| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
|              | 嵌入文档         | MongoDB通过嵌入式文档来替代多表连接 |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |



#### 2.3 MongoDB 的数据模型

MongoDB 的最小存储单位就是文档(document)对象。文档(document)对象对应于关系型数据库的行。数据在MongoDB中以 BSON（Binary-JSON）文档的格式存储在磁盘上。

`BSON（Binary Serialized Document Format）` 是一种类json的一种二进制形式的存储格式，简称 **Binary  JSON**。BSON和JSON一样，支持 内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。

BSON采用了类似于 C 语言结构体的名称、对表示方法，支持内嵌的文档对象和数组对象，具有轻量性、可遍历性、高效性的三个特点，可 以有效描述非结构化数据和结构化数据。这种格式的优点是灵活性高，但它的缺点是空间利用率不是很理想。

Bson中，除了基本的JSON类型：string,integer,boolean,double,null,array和object，mongo还使用了特殊的数据类型。这些类型包括 date,object id,binary data,regular expression 和code。每一个驱动都以特定语言的方式实现了这些类型，查看你的驱动的文档来获取详 细信息。BSON数据类型参考列表：

| 类型                     | 整数 | 别名                | 备注                             | 说明                         |
| :----------------------- | :--- | :------------------ | :------------------------------- | :--------------------------- |
| Double                   | 1    | double              | shell中的数字类型                | 64位浮点数                   |
| String                   | 2    | string              | {"x" : "foobar"}                 | 字符串类型                   |
| Object                   | 3    | object              |                                  | 对象类型                     |
| Array                    | 4    | array               | {"x" ： ["a", "b", "c"]}         | 数组类型                     |
| Binary data              | 5    | binData             | shell中不可用                    | 二进制数据类型               |
| Undefined                | 6    | undefined           | 已过期                           | 未定义类型                   |
| ObjectId                 | 7    | objectId            | {"X" :ObjectId() }               | 对象id类型                   |
| Boolean                  | 8    | bool                | {"x":true}+                      | 布尔类型                     |
| Date                     | 9    | date                |                                  | 日期类型                     |
| Null                     | 10   | null                |                                  | 用于表示空值或者不存在的字段 |
| Regular Expression       | 11   | regex               |                                  | 正则表达式类型               |
| DBPointer                | 12   | dbPointer           | 已过期                           |                              |
| JavaScript               | 13   | javascript          | {"x" ： function() { /* …… */ }} | JavaScript代码               |
| Symbol                   | 14   | symbol              | shell中不可用，已过期            |                              |
| JavaScript（with scope） | 15   | javascriptWithScope |                                  | 带做用域的JavaScript代码     |
| 32-bit integer           | 16   | int                 | shell中不可用                    | 32位整数                     |
| Timestamp                | 17   | timestamp           |                                  | 时间戳类型                   |
| 64-bit integer           | 18   | long                | shell中不可用                    | 64位整数                     |
| Decimal128               | 19   | decimal             | 3.4版本新增                      |                              |
| Min key                  | -1   | minKey              | shell中无此类型                  | 最小键                       |
| Max key                  | 127  | maxKey              | shell中无此类型                  | 最大键                       |

更详细的BSON数据类型的信息请参考：[MongoDB BSON数据类型详解](./MongoDB从入门到精通—02 MongoDB BSON数据类型详解.md)



#### 2.4 MongoDB 的特点

MongoDB主要有如下特点： 

（1）**高性能**： MongoDB提供高性能的数据持久性。特别是对嵌入式数据模型的支持减少了数据库系统上的I/O活动。索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。（文本索引解决搜索的需求、TTL索引解决历史数据自动过期的需求、地理位置索引可用于构建各种 O2O 应用） mmapv1、wiredtiger、mongorocks（rocksdb）、in-memory 等多引擎支持满足各种场景需求。 Gridfs解决文件存储的需求。 

（2）**高可用性**： MongoDB的复制工具称为副本集（replica set），它可提供自动故障转移和数据冗余。 

（3）**高扩展性**： MongoDB提供了水平可扩展性作为其核心功能的一部分。 分片将数据分布在一组集群的机器上。（海量数据存储，服务能力水平扩展） 从3.4开始，MongoDB支持基于片键创建数据区域。在一个平衡的集群中，MongoDB将一个区域所覆盖的读写只定向到该区域内的那些片。 

（4）**丰富的查询支持**： MongoDB支持丰富的查询语言，支持读和写操作(CRUD)，比如数据聚合、文本搜索和地理空间查询等。 

（5）**其他特点**：如无模式（动态模式）、灵活的文档模型、

