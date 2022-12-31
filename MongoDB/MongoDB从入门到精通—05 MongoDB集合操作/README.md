## MongoDB从入门到精通—05 MongoDB集合操作

**MongoDB 中的集合（collection）可以类比关系型数据库中的表（table）**,针对MongoDB集合的操作主要有创建和删除两种，下面分别介绍具体的用法。



### 一、MongoDB 创建集合

MongoDB 创建集合的语法格式如下：

```shell
db.createCollection(name[, options])
```

参数说明：

- **name**: 要创建的集合名称
- **options**: 可选参数, 指定有关内存大小及索引的选项

options 可以是如下参数：

| 字段        | 类型 | 描述                                                         |
| :---------- | :--- | :----------------------------------------------------------- |
| capped      | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数。** |
| autoIndexId | 布尔 | 3.2 之后不再支持该参数。（可选）如为 true，自动在 _id 字段创建索引。默认为 false。 |
| size        | 数值 | （可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

特别的，在 MongoDB 中，你不需要显示的创建集合。当你插入一些文档时，MongoDB 会自动创建集合，这就是MongoDB隐式地为我们创建了集合。





### 二、MongoDB 删除集合

MongoDB 删除集合的语法格式如下：

```shell
db.<collection_name>.drop()
```

`collection_name` 是你要删除的集合的名称，如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。





### 三、小试牛刀

使用 **easyblog_dev** 数据库：

```shell
> use easyblog_dev
switched to db easyblog_dev
> db
easyblog_dev
> 
```

显示创建 **article** 集合：

```shell
> db.createCollection("article")
{ "ok" : 1 }
>
```

隐式创建 **article** 集合

```shell
> db.article.insert({"article_id":"100000","name":"MongoDB教程","author":"1001"})
WriteResult({ "nInserted" : 1 })
>
```

如果要查看已经创建集合，可以使用 **show collections** 或 **show tables** 命令：

```shell
> show collections
article
easyblog_dev
```

删除 article 集合：

```shell
> db.article.drop()
true
>
```

