##  Cassandra CQL数据类型



CQL提供了一组丰富的内置数据类型，包括集合类型。除了这些数据类型，用户还可以创建自己的自定义数据类型。下表提供了CQL中可用的内置数据类型的列表。



## 一、基本数据类型

CQL支持的基本数据类型如下表格所示，可能会有变动，具体以[官方文档](https://cassandra.apache.org/doc/latest/cassandra/cql/types.html)为准。

| Type        | Constants supported | Description                                                  |
| :---------- | :------------------ | :----------------------------------------------------------- |
| `ascii`     | `string`            | ASCII 字符串                                                 |
| `bigint`    | `integer`           | 64 位有符号长整型                                            |
| `blob`      | `blob`              | 二进制数据                                                   |
| `boolean`   | `boolean`           | 布尔值，true 或 false                                        |
| `counter`   | `integer`           | 计数器列（64 位有符号值）。                                  |
| `date`      | `integer`, `string` | 日期（没有对应的时间值）                                     |
| `decimal`   | `integer`, `float`  | 可变精度十进制                                               |
| `double`    | `integer` `float`   | 64 位 IEEE-754 浮点数                                        |
| `duration`  | `duration`,         | 纳秒精度的时间                                               |
| `float`     | `integer`, `float`  | 32 位 IEEE-754 浮点数                                        |
| `inet`      | `string`            | IP 地址，IPv4（4 字节长）或 IPv6（16 字节长）。              |
| `int`       | `integer`           | 32位无符号整型                                               |
| `smallint`  | `integer`           | 16位无符号整型                                               |
| `text`      | `string`            | UTF8 编码字符串                                              |
| `time`      | `integer`, `string` | 具有纳秒精度的时间（没有相应的日期值）.                      |
| `timestamp` | `integer`, `string` | 具有毫秒精度的时间戳（日期和时间）                           |
| `timeuuid`  | `uuid`              | Version 1 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), |
| `tinyint`   | `integer`           | 8位无符号整型                                                |
| `uuid`      | `uuid`              | UUID                                                         |
| `varchar`   | `string`            | UTF8 编码字符串                                              |
| `varint`    | `integer`           | 任意精度整数                                                 |

**示例**

使用基本数据类型定义student，它具有id(s_id),家庭地址(s_addr)，分数(s_score)，年龄(s_age)，身高(s_height)，性别(s_gender)以及入学时间(s_create_time) 和 信息更新时间 (s_update_time)，我们可以使用如下命令实现：

```cql
CREATE TABLE student(
  s_id bigint PRIMARY KEY,
  s_score int,
  s_age int,
  s_gender varchar,
  s_create_time timestamp,
  s_update_time timestamp
)
```



## 二、集合类型

CQL 支持三种集合：`maps`, `sets`和 `lists`。 这些集合的类型可以由以下内容定义：

### 2.1 Maps

Maps是一组（排序的）键值对，其中键是唯一的，并且映射按其键排序。 您可以定义和插入map：

```cql
//使用map作为数据类型定义TABLE
CREATE TABLE user(
  id text PRIMARY KEY,
  name text,
  favs map<text,text>  // map的键和值的类型均为text
);

//向具有map类型的TABLE中插入数据
INSERT INTO(id,name,favs) 
       VALUES ('jsmith', 'John Smith', { 'fruit' : 'Apple', 'band' : 'Beatles' });

//修改map的值
UPDATE user set favs = { 'fruit' : 'Banana' } WHERE id = 'jsmith'
```



### 2.2 Sets

Sets是唯一值的（排序）集合。 您可以定义和插入Sets：

```cql
//使用set作为数据类型定义TABLE
CREATE TABLE images (
   name text PRIMARY KEY,
   owner text,
   tags set<text> // A set of text values
);

//向具有set类型的TABLE插入数据
INSERT INTO(name,owner,tags)
       VALUES ('cat.jpg', 'jsmith', { 'pet', 'cute' });
       
//修改set的值
UPDATE images SET tags = { 'kitten', 'cat', 'lol' } WHERE name = 'cat.jpg';
```



### 2.3 Lists

Lists是非唯一值的（排序）集合，其中元素按列表中的位置排序。 您可以定义和插入一个列表：

```cql
//使用list作为数据类型定义TABLE
CREATE TABLE plays (
    id text PRIMARY KEY,
    game text,
    players int,
    scores list<int> // A list of integers
)

//向具有list类型的TABLE插入数据
INSERT INTO plays (id, game, players, scores)
           VALUES ('123-afde', 'quake', 3, [17, 4, 2]);

//修改list的值
UPDATE plays SET scores = [ 3, 9, 4] WHERE id = '123-afde';
```



## 三、自定义类型（User-Defined Types ,UDTs）

cqlsh为用户提供了创建自己的数据类型的工具。下面给出了处理用户定义的数据类型时使用的命令。

- **CREATE TYPE** -创建用户定义的数据类型。
- **ALTER TYPE** -修改用户定义的数据类型。
- **DROP TYPE** -删除用户定义的数据类型。
- **DESCRIBE TYPE** -描述用户定义的数据类型。
- **DESCRIBE TYPES** -描述用户定义的数据类型。

这些语句的使用方式和表语句的操作方式一直，具体参考官方文档或者之前的文章。

```cql
CREATE TABLE user(
  name text PRIMARY KEY,
  addr map<text,frozen<address>>  //用户地址映射 address 类型这里没有需要自定义
);

//自定义address类型
CREATE TYPE address(
  street text,
  city text,
  zip text,
  phone map<text,phone>   //phone类型这里也没有，需要自定义
);

CREATE TYPE phone(
  country_no int,
  number text
);
```



