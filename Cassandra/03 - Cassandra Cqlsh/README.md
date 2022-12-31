## Cassandra cqlsh

本文介绍Cassandra查询语言shell，并解释如何使用其命令。

默认情况下，Cassandra提供了一个Cassandra查询语言shell（cqlsh），允许用户与它通信。使用此shell，您可以执行Cassandra查询语言（CQL）。

使用cqlsh，你可以

- 定义模式， 
- 插入数据，
- 执行查询。

## 一、启动cqlsh

在命令行使用命令`cqlsh`启动cqlsh，如下所示。它提供Cassandra cqlsh提示作为输出。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%881.34.56.png)

Cqlsh - 如上所述，此命令用于启动cqlsh提示符。此外，它还支持更多的选项。下表说明了cqlsh的所有选项及其用法。

| 选项                          | 用法                                                        |
| ----------------------------- | ----------------------------------------------------------- |
| cqlsh --help                  | 显示有关cqlsh命令的选项的帮助主题。                         |
| cqlsh --version               | 提供您正在使用的cqlsh的版本。                               |
| cqlsh --color                 | 指示shell使用彩色输出。                                     |
| cqlsh --debug                 | 显示更多的调试信息。                                        |
| cqlsh --execute cql_statement | 指示shell接受并执行CQL命令。                                |
| cqlsh --file= **“file name”** | 如果使用此选项，Cassandra将在给定文件中执行命令并退出。     |
| cqlsh --no-color              | 指示Cassandra不使用彩色输出。                               |
| cqlsh -u **“user name”**      | 使用此选项，您可以验证用户。默认用户名为：cassandra。       |
| **cqlsh-p “pass word”**       | 使用此选项，您可以使用密码验证用户。默认密码为：cassandra。 |

## 二、cqlsh命令

Cqlsh有几个命令，允许用户与它进行交互。命令如下所示。

### 2.1 记录的Shell命令

下面给出了Cqlsh记录的shell命令。这些是用于执行任务的命令，如显示帮助主题，退出cqlsh，描述等。

- **HELP** -显示所有cqlsh命令的帮助主题。
- **CAPTURE** -捕获命令的输出并将其添加到文件。
- **CONSISTENCY** -显示当前一致性级别，或设置新的一致性级别。
- **COPY** -将数据复制到Cassandra并从Cassandra复制数据。
- **DESCRIBE** -描述Cassandra及其对象的当前集群。
- **EXPAND** -纵向扩展查询的输出。 
- **EXIT** -使用此命令，可以终止cqlsh。
- **PAGING** -启用或禁用查询分页。
- **SHOW** -显示当前cqlsh会话的详细信息，如Cassandra版本，主机或数据类型假设。
- **SOURCE** -执行包含CQL语句的文件。
- **TRACING** -启用或禁用请求跟踪。

### 2.2 CQL数据定义命令

- **CREATE KEYSPACE** -在Cassandra中创建KeySpace。 
- **USE** -连接到已创建的KeySpace。
- **ALTER KEYSPACE** -更改KeySpace的属性。
- **DROP KEYSPACE** -删除KeySpace。
- **CREATE TABLE** -在KeySpace中创建表。
- **ALTER TABLE** -修改表的列属性。
- **DROP TABLE** -删除表。 
- **TRUNCATE** -从表中删除所有数据。
- **CREATE INDEX** -在表的单个列上定义新索引。
- **DROP INDEX** -删除命名索引。

### 2.3 CQL数据操作指令

- **INSERT** -在表中添加行的列。
- **UPDATE** -更新行的列。
- **DELETE** -从表中删除数据。 
- **BATCH** -一次执行多个DML语句。

### 2.4 CQL字句

- **SELECT** -此子句可以从表中读取数据
- **WHERE** -where子句与select一起使用以读取特定数据。
- **ORDERBY** -orderby子句与select一起使用，以特定顺序读取特定数据。