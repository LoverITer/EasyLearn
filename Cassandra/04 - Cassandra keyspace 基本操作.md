## Cassandra keyspace 基本操作







## 一、创建 keyspace

### 1、使用cqlsh创建一个**Keyspace**

Cassandra中的键空间是一个定义节点上数据复制的命名空间。集群每个节点包含一个键空间。如下是创建键空间的语法。

#### 1.1 语法

```cql
CREATE KEYSPACE <indentifier> WITH <properties>
```

**即**

```CQL
CREATE KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of   replicas’};

CREATE KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’}

AND durable_writes = ‘Boolean value’;
```

CREATE KEYSPACE语句有两个属性：`replication`和`durable_writes`。

#### 1.2 参数：replication

replication（复制）选项用于**指定副本位置策略和所需副本的数量**。下表列出了所有副本位置策略。

| 策略名称           | 描述                                             |
| ------------------ | ------------------------------------------------ |
| **简单的策略**     | 为集群指定简单的复制因子。                       |
| **网络拓扑策略**   | 使用此选项，可以单独为每个数据中心设置复制因子。 |
| **旧网络拓扑策略** | 使用此选项，可以单独为每个数据中心设置复制因子。 |

使用此选项，您可以指示Cassandra是否对当前KeySpace的更新使用commitlog。此选项不是强制性的，默认情况下，它设置为true。



**示例**

下面给出了创建KeySpace的示例。

- 这里我们创建一个名为**test_space** 的KeySpace。
- 我们使用第一个副本放置策略，即**简单策略**。
- 我们选择复制因子为**1个副本**。

```cql
CREATE KEYSPACE test_space 
WITH replication ={'calss':'SimpleStrategy','replication_factor' : 1};
```

**查看创建的keyspace**

在cqlsh命令行可以使用命令 `describe keyspaces;` 查看当前所有的keyspace

<img src="img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%881.57.33.png" style="zoom:50%;" />

在这里可以看到上面使用命令创建的test_sapce空间

#### 1.3 参数：durable_writes

默认情况下，表的durable_writes属性设置为**true**，但可以将其设置为false。

**示例**

下面给出了示例持久写入属性的使用示例。

```cql
CREATE KEYSPACE test_space_1 
WITH replication = {'class':'SimpleStrategy','replication_factor' : 3}
AND durable_writes =false
```

**验证**

可以通过查询系统键空间来验证test_space_1 KeySpace的durable_writes属性是否设置为false。此查询提供了所有KeySpaces及其属性。

```cql
SELECT * FROM system.schema_keyspaces;
```



### 2、使用keyspace

可以使用关键字USE使用创建的KeySpace。其语法如下：

```cql
USE <identifier>
```

#### 示例

在下面的示例中，我们使用KeySpace **test_space**。

```cql
cqlsh> use test_space;
cqlsh:test_space>   --进入到test_space中
```



### 3、使用Java API创建一个Keyspace



## 二、修改keyspace

ALTER KEYSPACE可用于更改属性，下面给出了此命令的语法。

**语法**

```cql
ALTER KEYSPACE <identifier> WITH <properties>
```

**即：**

```cql
ALTER KEYSPACE “KeySpace Name”
WITH replication = {'class': ‘Strategy name’, 'replication_factor' : ‘No.Of  replicas’};
```

ALTER KEYSPACE的属性与CREATE KEYSPACE相同。它有两个属性：replication和durable_writes。

## 三、删除keyspace

可以使用命令DROP KEYSPACE删除KeySpace。下面给出了删除KeySpace的语法。

```cql
DROP KEYSPACE <identifier>
```

即：

```cql
DROP KEYSPACE “KeySpace name”
```