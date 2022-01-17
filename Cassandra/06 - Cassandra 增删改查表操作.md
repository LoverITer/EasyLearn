## Cassandra 增删改查表操作



### 一、新增数据

可以使用命令**INSERT**将数据插入到表中行的列中。下面给出了在表中创建数据的语法。

```cql
INSERT INTO <tablename>
(<column1 name>, <column2 name>....)
VALUES (<value1>, <value2>....)
USING <option>
```

**示例**

假设有一个名为**emp**的表（emp_id，emp_name，emp_city，emp_phone，emp_sal），并且必须将以下数据插入**emp**表。

| emp_id | emp_name | emp_city  | emp_phone  | emp_sal |
| ------ | -------- | --------- | ---------- | ------- |
| 1      | ram      | Hyderabad | 9848022338 | 50000   |
| 2      | robin    | Hyderabad | 9848022339 | 40000   |
| 3      | rahman   | Chennai   | 9848022330 | 45000   |

使用 INSERT命令将指定数据插入emp表中：

```cql
INSERT INTO emp(emp_id,emp_name,emp_city,emp_phone,emp_sal) VALUES (1,'ram','Hyderabad',9848022338,50000);
INSERT INTO emp(emp_id,emp_name,emp_city,emp_phone,emp_sal) VALUES (2,'robin','Hyderabad',9848022330,40000);
INSERT INTO emp(emp_id,emp_name,emp_city,emp_phone,emp_sal) VALUES (3,'rahman','Chennai',9848022330,45000);
```

**验证**

插入数据后，使用SELECT语句验证数据是否已插入。如果使用SELECT语句验证emp表，它将给您以下输出。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.35.57.png)



### 二、读取数据

SELECT子句用于从Cassandra中的表读取数据。使用此子句，您可以读取整个表，单个列或特定单元格。下面给出了SELECT子句的语法。

```cql
SELECT FROM <tablename>
```

**示例**

假设在名为emp的键空间中有一个具有以下详细信息的表：

| emp_id | emp_name | emp_city  | emp_phone  | emp_sal |
| ------ | -------- | --------- | ---------- | ------- |
| 1      | ram      | Hyderabad | 9848022338 | 50000   |
| 2      | robin    | Hyderabad | 9848022339 | 40000   |
| 3      | rahman   | Chennai   | 9848022330 | 45000   |

以下示例显示如何使用SELECT子句读取整个表。这里我们读一个表**emp**。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.35.57.png)



#### 读取必需的列

以下示例显示如何读取表中的特定列。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.45.27.png)



#### Where子句

使用WHERE子句，可以对必需的列设置约束。其语法如下：

```cql
SELECT FROM <table name> WHERE <condition>;
```

**注意：**WHERE子句只能用于作为主键的一部分或在其上具有辅助索引的列。

**示例**

在以下示例中，我们会读取薪水为50000的员工的详细信息。首先，将辅助索引设置为列emp_sal。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%883.49.01.png)



### 三、更新数据

UPDATE是用于更新表中的数据的命令。在更新表中的数据时使用以下关键字：

- **Where** - 此子句用于选择要更新的行。
- **Set** - 使用此关键字设置值。
- **Must** - 包括组成主键的所有列。

在更新行时，如果给定行不可用，则UPDATE创建一个新行。下面给出了UPDATE命令的语法：

```cql
UPDATE <tablename>
SET <column name> = <new value>
<column name> = <value>....
WHERE <condition>
```

**示例**

假设有一个名为emp的表。此表存储某公司员工的详细信息，其具有以下详细信息：

| **emp_id** | **emp_name** | **emp_city** | **emp_phone** | **emp_sal** |
| ---------- | ------------ | ------------ | ------------- | ----------- |
| 1          | ram          | Hyderabad    | 9848022338    | 50000       |
| 2          | robin        | Hyderabad    | 9848022339    | 40000       |
| 3          | rahman       | Chennai      | 9848022330    | 45000       |

现在让我们将robin的emp_city更新到Delhi，将他的工资更新为50000.以下是执行所需更新的查询。

```cql
UPDATE emp SET emp_city = 'Delhi' , emp_sal = 50000 WHERE emp_id= 2 ;
```



### 四、删除数据

使用命令DELETE从表中删除数据。其语法如下：

```CQL
DELETE <identifier> FROM <table_name> WHERE <condition>;
```

**示例**

让我们假设Cassandra中有一个名为emp的具有以下数据的表：

| emp_id | emp_name | emp_city  | emp_phone  | emp_sal |
| ------ | -------- | --------- | ---------- | ------- |
| 1      | ram      | Hyderabad | 9848022338 | 50000   |
| 2      | robin    | Hyderabad | 9848022339 | 50000   |
| 3      | rahman   | Chennai   | 9848022330 | 45000   |

现在我们可以使用如下语句删除表格最后一行emp_sal列：

```cql
DELETE emp_sal FROM emp WHERE  emp_id=3;
```

如下图所示，由于我们删除了rahman的薪资，你将看到一个空值代替薪资。

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%884.39.18.png)

#### 删除整行

以下命令从表中删除整个行。

```cql
DELETE FROM emp WHERE emp_id=3;
```

如下图所示，由于我们删除了最后一行，因此表中只剩下两行

![](img/%E6%88%AA%E5%B1%8F2022-01-17%20%E4%B8%8B%E5%8D%884.42.02.png)