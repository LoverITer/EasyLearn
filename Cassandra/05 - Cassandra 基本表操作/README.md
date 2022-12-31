## Cassandra 基本表操作



## 一、创建表

可以使用命令**CREATE TABLE**创建表。下面给出了创建表的语法。

### 语句

```cql
CREATE (TABLE | COLUMNFAMILY) <tablename>
('<column-definition>' , '<column-definition>')
(WITH <option> AND <option>)
```

### 定义列

您可以定义一个列，如下所示。

```cql
<column name1> <data type>,
<column name2> <data type>,

example:

age int,
name text
```

### 主键

**主键是用于唯一标识行的列。因此，在创建表时，必须定义主键**。**主键由表的一个或多个列组成**。您可以定义表的主键，如下所示。

```cql
CREATE TABLE tablename(
   column1 name datatype PRIMARYKEY,
   column2 name data type,
   column3 name data type.
);
```

**或者**

```cql
CREATE TABLE tablename(
   column1 name datatype PRIMARYKEY,
   column2 name data type,
   column3 name data type,
   PRIMARY KEY (column1)
);
```

**示例**

下面给出一个使用cqlsh在Cassandra中创建表的示例。我们使用 test_space keyspace ，创建名为emp的表，它将有详细信息，如员工姓名，id，城市，工资和电话号码。Employee id是主键。

```cql
CREATE TABLE emp( 
  emp_id int PRIMARY KEY, 
  emp_name text, 
  emp_city text, 
  emp_sal varint, 
  emp_phone varint
);
```

**验证**

select语句将为您提供模式。使用select语句验证表，如下所示。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.32.59.png)

在这里，您可以观察使用给定列创建的表。



## 二、修改表

可以使用命令ALTER TABLE更改表。下面给出了修改表的语法。

```CQL
ALTER (TABLE | COLUMNFAMILY) <tablename> <instruction>
```

使用ALTER命令，可以执行以下操作：

- 添加列
- 删除列

### 添加列

使用ALTER命令，可以向表中添加列。在添加列时，必须注意，列名称不要与现有列名称冲突，并且表未使用紧凑存储选项定义。下面给出了向表中添加列的语法。

```cql
ALTER TABLE name
ADD  new_column datatype;
```

**示例
**

下面给出了向现有表中添加列的示例。这里我们在名为**emp**的表中添加一个名为**emp_email**的文本数据类型的列。

```cql
ALTER TABLE emp ADD emp_email text;
```

**验证**

使用SELECT语句验证列是否已添加。在这里可以观察新添加的列emp_email。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.39.07.png)

### 删除列

使用ALTER命令，可以从表中删除列。在从表中删除列之前，请检查该表是否未使用紧凑存储选项进行定义。下面给出了使用ALTER命令从表中删除列的语法。

```cql
ALTER TABLE name
DROP column_name;
```

**示例**

下面给出了从表中删除列的示例。这里我们删除名为**emp_email**的列。

```cql
ALTER TABLE emp DROP emp_email;
```

**验证**

使用**select**语句验证列是否已删除，如下所示。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.43.04.png)





## 三、删除表

可以使用命令Drop Table删除表。其语法如下：

```cql
DROP TABLE <tablename>
```

 **示例**

以下代码从KeySpace删除现有表。

```cql
cqlsh:test_space> DROP TABLE emp;
```

**验证**

用 `describe columnfamilies` 命令验证表是否已删除。由于emp表已删除，您不会在列族列表中找到它。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.47.27.png)



## 四、截断表

可以使用TRUNCATE命令截断表。截断表时，表的所有行都将永久删除。下面给出了此命令的语法。

```cql
TRUNCATE <tablename>
```

**示例**

假设有一个名为student的表有以下数据。

![截屏2022-01-17 下午2.57.08](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.57.08.png)

现在使用TRUNCATE命令截断表，那么表中的数据将会清空。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%882.58.48.png)





## 五、索引操作

### 5.1 创建索引

可以使用命令**CREATE INDEX**在Cassandra中创建索引。其语法如下：

```cql
CREATE INDEX <identifier> ON <tablename>
```

下面给出一个创建列的索引的例子。这里，我们在名为emp的表中为列“emp_name”创建索引。

```cql
cqlsh:test_space> CREATE INDEX name ON emp (emp_name);
```



### 5.2 删除索引

可以使用命令DROP INDEX删除索引。其语法如下：

```cql
DROP INDEX <identifier>
```

下面给出了删除表中列的索引的示例。这里我们删除表emp中的列名的索引。

```cql
cqlsh:test_space> drop index name;
```



## 六、批处理

使用**BATCH**，可以同时执行多个修改语句（插入，更新，删除）。其语法如下：

```cql
BEGIN BATCH
<insert-stmt>/ <update-stmt>/ <delete-stmt>
APPLY BATCH
```

 **示例**

假设Cassandra中有一个名为emp的表，具有以下数据：

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.08.02.png)

在这个例子中，我们将执行以下操作：

- 插入包含以下详细信息的新行（4，rajeev，pune，9848022331，30000）。
- 将emp_id=3的员工的工资更新为50000。
- 删除行ID为2的员工的城市。

要一次性执行上述操作，可以使用BATCH命令：

```cql
BEGIN BATCH 
INSERT INTO emp(emp_id,emp_name,emp_city,emp_phone,emp_sal) VALUES(4,'rajeev','pune',9848022331,30000); 
UPDATE emp SET emp_sal = 50000 WHERE emp_id = 3; 
DELETE emp_city FROM emp WHERE emp_id = 2; 
APPLY BATCH;
```

**验证**

更改后，使用SELECT语句验证表。它应该产生以下输出：

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.20.00.png)

