### MongoDB从入门到精通—06 MongoDB文档操作

**MongoDB 中的文档（document）可以类比关系型数据库中的数据记录行（row）**,文档的数据结构和 JSON 类似，所有存储在集合中的数据都是 BSON 格式。BSON 是一种类似 JSON 的二进制形式的存储格式，是 **Binary JSON** 的简称。针对MongoDB文档的操作有创建、删除、更新、查询，下面分别介绍具体的用法。



### 一、插入文档

MongoDB 创建文档的语法格式如下：

#### 1.1 单个文档插入

```shell
db.<collection_name>.insert(
   <document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

参数说明：

| Parameter    | Type              | Description                                                  |
| ------------ | ----------------- | ------------------------------------------------------------ |
| document     | document or array | 要插入到集合中的文档或文档数组。（(json格式）                |
| writeConcern | document          | 写入策略，默认为 1，即要求确认写操作，0 是不要求。           |
| ordered      | boolean           | 可选。如果为真，则按顺序插入数组中的文档，如果其中一个文档出现错误，MongoDB将返回而 不处理数组中的其余文档。如果为假，则执行无序插入，如果其中一个文档出现错误，则继续处理 数组中的主文档。在版本2.6+中默认为true |

**示例**

使用 **db.comment.insert()** 向 comment 集合(表)中插入一条测试数据：

```shell
> db.comment.insert({"article_id":"10000","content":"专家说空腹不能吃饭，对身体不好","user_id":"1001","nickname":"张三","create_time":new Date(),"like_num":NumberInt(100)})
WriteResult({ "nInserted" : 1 })
>
```

**提示：**

* 1）comment 集合如果不存在，则会隐式创建
* 2）mongo中的数字，默认情况下是double类型，如果要存整型，必须使用函数 NumberInt() 转型，否则取出来就有问题了。
* 3）插入当前日期使用 new Date() 
* 4）插入的数据没有指定 \_id ，会自动生成主键值 
* 5）如果某字段没值，可以赋值为null，或不写该字段。
* 6）插入之后 `WriteResult({ "nInserted" : 1 })` 表示插入成功，否则都是失败了



#### 1.2 多个文档插入

**3.2 版本之后新增了 db.collection.insertMany() 可以支持多个文档批量插入**

```shell
db.<collection_name>.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
```

参数说明：

| Parameter    | Type              | Description                                                  |
| ------------ | ----------------- | ------------------------------------------------------------ |
| document     | document or array | 要插入到集合中的文档或文档数组。（(json格式）                |
| writeConcern | document          | 写入策略，默认为 1，即要求确认写操作，0 是不要求。           |
| ordered      | boolean           | 可选。如果为真，则按顺序插入数组中的文档，如果其中一个文档出现错误，MongoDB将返回而 不处理数组中的其余文档。如果为假，则执行无序插入，如果其中一个文档出现错误，则继续处理 数组中的主文档。在版本2.6+中默认为true |

**示例**

使用 **db.comment.insertMany()** 向 comment 集合(表)中批量插入测试数据：

```shell
> db.comment.insertMany([
...    {"article_id":"10001","content":"我夏天空腹喝凉开水，冬天喝温开水","user_id":"1001","nickname":"张三","create_time":new Date(),"like_num":NumberInt(100)},
... {"article_id":"10002","content":"我感觉你像谣言","user_id":"1001","nickname":"张三","create_time":new Date(),"like_num":NumberInt(200)},
... {"article_id":"10003","content":"我一直喝凉开水，冬天夏天都喝。","user_id":"1002","nickname":"李四","create_time":new Date(),"like_num":NumberInt(300)},
... {"article_id":"10004","content":"专家说空腹不能吃饭，对身体不好","user_id":"1002","nickname":"李四","create_time":new Date(),"like_num":NumberInt(400)},
... {"article_id":"10005","content":"研究表明，刚烧开的水千万不能喝，因为烫嘴","user_id":"1003","nickname":"王五","create_time":new Date(),"like_num":NumberInt(600)}
... ])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("6256dfb4f6eff1fc7f9176a5"),
                ObjectId("6256dfb4f6eff1fc7f9176a6"),
                ObjectId("6256dfb4f6eff1fc7f9176a7"),
                ObjectId("6256dfb4f6eff1fc7f9176a8"),
                ObjectId("6256dfb4f6eff1fc7f9176a9")
        ]
}
>
```



### 二、查询文档

MongoDB 查询数据的语法格式如下：

```shell
db.<collection_name>.find(query, projection)
```

- **query** ：可选，使用查询操作符指定查询条件
- **projection** ：可选，使用投影操作符指定返回的键。查询时默认会返回文档中所有键值， 如果只需某些键，那就在 `projection` 置位1。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

```
db.<collection_name>.find().pretty()
```

除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

#### MongoDB  Where 条件语句

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

| 操作       | 格式                      | RDBMS中的类似语句     |
| :--------- | :------------------------ | :-------------------- |
| 等于       | {\<key>:\<value>}         | where by = '菜鸟教程' |
| 小于       | {\<key>:{\\$lt:\<value>}} | where likes < 50      |
| 小于或等于 | {\<key>:{\$lte:\<value>}} | where likes <= 50     |
| 大于       | {\<key>:{\$gt:\<value>}}  | where likes > 50      |
| 大于或等于 | {\<key>:{\$gte:\<value>}} | where likes >= 50     |
| 不等于     | {\<key>:{\$ne:\<value>}}  | where likes != 50     |

**示例**

```shell
#查询article_id=10002的评论
db.comment.find({"article_id":"10002"})
#查询点赞数小于100的评论
db.comment.find({"like_num":{$lt:NumberInt(100)}})
#查询点赞数小于等于100的评论
db.comment.find({"like_num":{$lte:NumberInt(100)}})
#查询点赞数大于100的评论
db.comment.find({"like_num":{$gt:NumberInt(100)}})
#查询点赞数大于等于100的评论
db.comment.find({"like_num":{$gte:NumberInt(100)}})
#查询点赞数不等于100的评论
db.comment.find({"like_num":{$ne:NumberInt(100)}})
```



#### MongoDB AND 条件

MongoDB 的 find() 方法可以传入多个键(key)，每个键(key)以逗号隔开，即常规 SQL 的 AND 条件。

语法格式如下：

```shell
db.<collection_name>.find({key1:value1, key2:value2})
```

**示例**

比如要查询 article_id = 10002 且点赞数不小于100的评论，可以输入如下语句：

```shell
db.comment.find({"article_id":"10002","like_num":{$gte:NumberInt(100)}})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414225000.png)



#### MongoDB OR 条件

MongoDB OR 条件语句使用了关键字 **\$or**,语法格式如下：

```shell
db.<collection_name>.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
)
```

**示例**

比如要查询user_id=1002或者1003的评论，可以输入如下语句：

```shell
db.comment.find({$or:[{"user_id":"1002"},{"user_id":"1003"}]})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414225655.png)



#### MongoDB IN条件

MongoDB IN 条件语句使用了关键字 **\$in**,语法格式如下：

```shell
db.<collection_name>.find(
   {
      key: {
         $in:[value1,value2......]
      }
   }
)
```

**示例**

查询user_id为1001,1002,1003的所有评论，可以输入下面的语句：

```shell
db.comment.find({"user_id":{$in:["1001","1002","1003"]}})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220415202144.png)





#### MongoDB 投影查询

有时候我们并不想获取到文档的全部内容，而只是获取其中一些信息，此时我们可以使用MongoDB的投影操作，比如比如要查询user_id=1002或者1003的评论，要求只返回user_id、article_id 以及 nickname，那么可以输入如下语句：

```shell
db.comment.find({$or:[{"user_id":"1002"},{"user_id":"1003"}]},{"user_id":1,"article_id":1,"nickname":1})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414230300.png)

观察结果发现，MongoDB默认为我们查询到“_id”键，如果不需要也可以通过投影操作将其设置为不查询：

```shell
db.comment.find({$or:[{"user_id":"1002"},{"user_id":"1003"}]},{"user_id":1,"article_id":1,"nickname":1,"_id":0})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414230535.png)





### 三、更新文档

MongoDB 使用 **update()** 和 **save()** 方法来更新集合中的文档。

update() 方法用于更新已存在的文档。语法格式如下：

```shell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint: <document|string> 
   }
)
```

**参数说明**（主要关注前四个参数）：

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- writeConcern :可选，抛出异常的级别。
- collation：可选。指定要用于操作的校对规则。校对规则允许用户为字符串比较指定特定于语言的规则，例如字母大小写和重音标记的规则。
- arrayFilters：可选。一个筛选文档数组，用于确定要为数组字段上的更新操作修改哪些数组元素。
- hint：可选。指定用于支持查询谓词的索引的文档或字符串。该选项可以采用索引规范文档或索引名称字符串。如果指定的索引不存在，则说明操作错误。例如，请参阅版本4中的“为更新操作指定提示。



##### 示例

**（1）覆盖式修改** 

如果我们想修改article_id为10001的记录，点赞量为101，输入以下语句：

```shell
db.comment.update({"article_id":"10001"},{"like_num":NumberInt(101)})
```

执行后，我们会发现，article_id=10001 条文档除了like_num字段其它字段都不见了

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414214907.png)

**（2）局部修改** 

为了修改部分字段而覆盖整个文档这个问题，我们需要使用修改器 `$set` 来实现，比如我们想修改article_id为10002的记录，浏览量为889，输入以下语句：

```shell
db.comment({"article_id":"10002"},{$set:{"like_num":NumberInt(899)}})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414215528.png)

**（3）批量修改** 

当需要批量修改符合条件的文档时，我们可以指定在更新参数中指定 `{multi:true}` ，比如更新user_id=1002对应的nickname=李四，我们可以输入如下语句：

```shell
db.comment.update({"user_id":"1002"},{$set:{"nickname":"李四"}},{multi:true})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220414220248.png)



### 四、删除文档

MongoDB 删除文档的语法格式如下：

```shell
db.<collection_name>.remove(
   [query],
   [justOne]
)
```

 MongoDB 2.6 版本以后，语法格式如下：

```shell
db.<collection_name>.remove(
    [query],
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。



##### 示例

删除nickname=张三的评论数据：

```shell
db.comment.remove({"nickname":"张三"})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220415194723.png)



删除全部数据：

```shell
db.comment.remove({})
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220415194856.png)

