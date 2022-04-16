---

---

## MongoDB从入门到精通—07 MongoDB索引

索引支持在MongoDB中高效地执行查询。如果没有索引，MongoDB将有可能采用全集合扫描，即扫描集合中的每个文档（类似MySQL 中的全部扫描），以选择与查询语句 匹配的文档。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

 如果查询存在适当的索引，MongoDB可以使用该索引限制必须检查的文档数。 

**索引是特殊的数据结构（MongoDB索引使用的是 B-Tree，MySQL是B+Tree）**，它以易于遍历的形式存储集合数据集的一小部分。索引存储特定字段或一组字段的值，按字段值排序。索引项的排 序支持有效的相等匹配和基于范围的查询操作。此外，MongoDB还可以使用索引中的排序返回排序结果。



### 一、MongoDB 索引的类型

#### 1.1 单字段索引

MongoDB支持在文档的单个字段上创建用户定义的升序/降序索引，称为单字段索引（Single Field Index）。 

对于单个字段索引和排序操作，索引键的排序顺序（即升序或降序）并不重要，因为MongoDB可以在任何方向上遍历索引。

![](http://image.easyblog.top/index-ascending.bakedsvg.svg)

#### 1.2 复合索引

MongoDB还支持多个字段的用户定义索引，即复合索引（Compound Index）。 

复合索引中列出的字段顺序具有重要意义。例如，如果复合索引由 { userid: 1, score: -1 } 组成，则索引首先按userid正序排序，然后 在每个userid的值内，再在按score倒序排序。

![](http://image.easyblog.top/index-compound-key.bakedsvg.svg)



#### 1.3 多key索引 （Multikey Index）

当索引的字段为数组时，创建出的索引称为多key索引，多key索引会为数组的每个元素建立一条索引。

![](http://image.easyblog.top/index-multikey.bakedsvg.svg)

#### 1.4 其他索引

地理空间索引（Geospatial Index）、文本索引（Text Indexes）、哈希索引（Hashed Indexes）。

* **地理空间索引（Geospatial Index）** 

  为了支持对地理空间坐标数据的有效查询，MongoDB提供了两种特殊的索引：返回结果时使用平面几何的二维索引和返回结果时使用球面 几何的二维球面索引。地理空间索引能很好的解决O2O的应用场景，比如『查找附近的美食』、『查找某个区域内的车站』等。

* **文本索引（Text Indexes）**

  MongoDB提供了一种文本索引类型，支持在集合中搜索字符串内容。这些文本索引不存储特定于语言的停止词（例如“the”、“a”、“or”）， 而将集合中的词作为词干，只存储根词。文本索引能解决快速文本查找的需求，比如有一个博客文章集合，需要根据博客的内容来快速查找，则可以针对博客内容建立文本索引。

* **哈希索引（Hashed Indexes）**

  为了支持基于散列的分片，MongoDB提供了散列索引类型，它对字段值的散列进行索引。这些索引在其范围内的值分布更加随机，但只支 持相等匹配，不支持基于范围的查询。





###  二、MongoDB 索引的管理操作

#### 2.1 查看索引

MongoDB 查看索引的语法格式如下：

```shell
db.<collection_name>.getIndexes()
```

> 提示：该语法命令运行要求是MongoDB 3.0+，执行命令后MongoDB会返回一个集合中的所有索引的数组。

【**示例**】

查看comment集合中所有的索引情况

```shell
> db.comment.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
>

```

结果中显示的是**默认\_id 索引**。 MongoDB在创建集合的过程中，在 \_id 字段上创建一个唯一的索引，默认名字为 \_id_ ，该索引可防止客户端插入两个具有相同值的文档，不能在_id字段上删除此索引。 

> 提示：该索引是唯一索引，因此值不能重复，即 _id 值不能重复的。在分片集群中，通常使用 _id 作为片键



#### 2.2 创建索引

MongoDB 创建索引的语法格式如下：

```shell
db.<collection_name>.createIndex(keys,options)
```

**参数说明**：

| Parameter |   Type   | Description                                                  |
| :-------: | :------: | ------------------------------------------------------------ |
|   keys    | document | 包含字段和值对的文档，其中字段是索引键，值描述该字段的索引类型。对于字段上的升序索引，请 指定值1；对于降序索引，请指定值-1。比如： {字段:1或-1} ，其中1 为指定按升序创建索引，如果你 想按降序来创建索引指定为 -1 即可。另外，MongoDB支持几种不同的索引类型，包括文本、地理空 间和哈希索引。 |
|  options  | document | 可选。包含一组控制索引创建的选项的文档。详细选项信息，请参见选项详情列表。 |



**MongoDB 选项（options ）信息**：

|     Parameter      |     Type      | Description                                                  |
| :----------------: | :-----------: | ------------------------------------------------------------ |
|     background     |    Boolean    | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为false。 |
|       unique       |    Boolean    | 建立的索引是否唯一。指定为true创建唯一索引。默认值为false.   |
|        name        |    string     | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名 称。 |
|      dropDups      |    Boolean    | **3.0+版本已废弃**。在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 false. |
|       sparse       |    Boolean    | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索 引字段中不会查询出不包含对应字段的文档.。默认值为 false. |
| expireAfterSeconds |    integer    | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
|         v          | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
|      weights       |   document    | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重 |
|  default_language  |    string     | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  |    string     | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language |

> 提示： 在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex() ，之后的版本使用了 db.collection.createIndex() 方法， ensureIndex() 还能用，但只是 createIndex() 的别名。



【**示例**】

（1）单字段索引：对comment集合的 user_id 字段建立升序索引

```shell
> db.comment.createIndex({"user_id":1})   #添加索引,1 表示升序排列索引
{
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "createdCollectionAutomatically" : false,
        "ok" : 1
}
> db.comment.getIndexes()   #再次查看结果
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_"
        },
        {
                "v" : 2,
                "key" : {
                        "user_id" : 1
                },
                "name" : "user_id_1"   #默认的生成的新索引名称 key_1/-1
        }
]
>
```

（2）复合索引：对 user_id 和 nickname 同时建立复合（Compound）索引

```shell
> db.comment.createIndex({"user_id":1,"nickname":-1})  #添加索引,1 表示升序排列索引 -1 表示降序排列索引
{
        "numIndexesBefore" : 2,
        "numIndexesAfter" : 3,
        "createdCollectionAutomatically" : false,
        "ok" : 1
}
> db.comment.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_"
        },
        {
                "v" : 2,
                "key" : {
                        "user_id" : 1
                },
                "name" : "user_id_1"
        },
        {
                "v" : 2,
                "key" : {                      #复合索引
                        "user_id" : 1,
                        "nickname" : -1
                },
                "name" : "user_id_1_nickname_-1"
        }
]
>
```



#### 2.3 删除索引

MongoDB 删除索引的语法格式如下：

```suosshell
db.<collection_name>.dropIndex(index)
```

**参数说明**：

| Parameter |        Type        | Description                                                  |
| :-------: | :----------------: | ------------------------------------------------------------ |
|   index   | string or document | 指定要删除的索引。可以通过索引名称或索引规范文档指定索引。若要删除文本索引，则只能指定索引名称。 |

【**示例**】

删除 comment 集合的 user_id 单字段索引 

```shell
db.comment.dropIndex({"user_id":1})   
#或者
db.comment.dropIndex("user_id_1")
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220415222737.png)





### 三、MongoDB 索引执行计划

#### 3.1 explain 语法

如果你熟悉关系型数据库的话（比如 MySQL），那么一定不会对执行计划这个词感到陌生的，在MySQL中我们可以使用 `explain` 工具来分析SQL查询语句性能，同样，在 MongoDB 中也存在同样的工具： **explain()**。

explain 语法如下：

```shell
db.<collection>.find(query,options).explain(options)
```

* options 参数：

  默认 explain 会携带 “queryPlanner” 参数，





#### 3.2 explain 输出结果分析

比如现在有如下查询语句：

```shell
db.comment.find({"artcile_id":{$in:["10001","10002"]}})
```

可以使用explain 对其进行分析：

```shell
db.comment.find({"article_id":{$in:["10001","10002"]}}).explain()
```

以上explain执行分析结果如下：

```json
{
        "explainVersion" : "1",
        "queryPlanner" : {
                "namespace" : "easyblog_dev.comment",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "article_id" : {
                                "$in" : [
                                        "10001",
                                        "10002"
                                ]
                        }
                },
                "queryHash" : "498CBB18",
                "planCacheKey" : "564AC023",
                "maxIndexedOrSolutionsReached" : false,
                "maxIndexedAndSolutionsReached" : false,
                "maxScansToExplodeReached" : false,
                "winningPlan" : {                 /*重点关注这里*/
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "article_id" : {
                                        "$in" : [
                                                "10001",
                                                "10002"
                                        ]
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "command" : {
                "find" : "comment",
                "filter" : {
                        "article_id" : {
                                "$in" : [
                                        "10001",
                                        "10002"
                                ]
                        }
                },
                "$db" : "easyblog_dev"
        },
        "serverInfo" : {
                "host" : "iZbp195ve61zmmqydk0z96Z",
                "port" : 27017,
                "version" : "5.0.7",
                "gitVersion" : "b977129dc70eed766cbee7e412d901ee213acbda"
        },
        "serverParameters" : {
                "internalQueryFacetBufferSizeBytes" : 104857600,
                "internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
                "internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
                "internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
                "internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
                "internalQueryProhibitBlockingMergeOnMongoS" : 0,
                "internalQueryMaxAddToSetBytes" : 104857600,
                "internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
        },
        "ok" : 1
}
```



##### 3.2.1 queryPlanner

查询计划的选择器，首先进行查询分析，最终选择一个winningPlan，是explain返回的默认层面。对于非分片集合，explain 返回以下 queryPlanner 信息：

```shell
"queryPlanner" : {
   "plannerVersion" : <int>,
   "namespace" : <string>, 
   "indexFilterSet" : <boolean>, 
   "parsedQuery" : {
      ...
   },
   "queryHash" : <hexadecimal string>, 
   "planCacheKey" : <hexadecimal string>,
   "optimizedPipeline" : <boolean>, // Starting in MongoDB 4.2, only appears if true
   "winningPlan" : {
      "stage" : <STAGE1>,
      ...
      "inputStage" : {
         "stage" : <STAGE2>,
         ...
         "inputStage" : {
            ...
         }
      }
   },
   "rejectedPlans" : [
      <candidate plan 1>,
      ...
   ]
}
```

各个字段具体含义如下：

| 字段             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| namespace        | 该值返回的是该query所查询的表                                |
| indexFilterSet   | 针对该query是否有indexfilter                                 |
| winningPlan      | 查询优化器针对该query所返回的最优执行计划的详细内容。        |
| stage            | 最优执行计划的stage，这里返回是FETCH，可以理解为通过返回的index位置去检索具体的文档（stage有数个模式，将在后文中进行详解） |
| inputStage       | 用来描述子stage，并且为其父stage提供文档和索引关键字。       |
| inputStage.stage |                                                              |
| keyPattern       | 所扫描的index内容，此处是did:1,status:1,modify_time: -1与scid : 1 |
| indexName        | winning plan所选用的index                                    |
| isMultiKey       | 是否是Multikey，如果索引建立在array上，此处将是true。        |
| direction        | query的查询顺序                                              |
| indexBounds      | winningplan所扫描的索引范围,如果没有制定范围就是[MaxKey, MinKey]，这主要是直接定位到mongodb的chunck中去查找数据，加快数据读取。 |
| rejectedPlans    | 其他执行计划（非最优而被查询优化器reject的）的详细返回，其中具体信息与winningPlan的返回中意义相同 |



##### 3.2.2 executionStats

executionStats 放回MongoDB 执行查询语句的详情。 此选项默认不返回，如果需要结果中包含 executionStats，需要在 explain 中指定 `"executionStats"` 参数。对于非分片集合，explain 返回以下 executionStats 信息：

```js
"executionStats" : {
   "executionSuccess" : <boolean>,
   "nReturned" : <int>,
   "executionTimeMillis" : <int>,
   "totalKeysExamined" : <int>,
   "totalDocsExamined" : <int>,
   "executionStages" : {
      "stage" : <STAGE1>
      "nReturned" : <int>,
      "executionTimeMillisEstimate" : <int>,
      "works" : <int>,
      "advanced" : <int>,
      "needTime" : <int>,
      "needYield" : <int>,
      "saveState" : <int>,
      "restoreState" : <int>,
      "isEOF" : <boolean>,
      ...
      "inputStage" : {
         "stage" : <STAGE2>,
         "nReturned" : <int>,
         "executionTimeMillisEstimate" : <int>,
         ...
         "inputStage" : {
            ...
         }
      }
   }
}
```

各个字段具体含义如下：

| 字段                | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| executionSuccess    | 是否执行成功                                                 |
| nReturned           | 满足查询条件的文档个数，即查询的返回条数                     |
| executionTimeMillis | 整体执行时间                                                 |
| totalKeysExamined   | 索引整体扫描的文档个数，和早起版本的nscanned 是一样的        |
| totalDocsExamined   | document扫描个数， 和早期版本中的nscannedObjects 是一样的    |
| executionStages     | 整个winningPlan执行树的详细信息，一个executionStages包含一个或者多个inputStages |



##### 3.2.3  allPlansExecution

为返回所有执行计划的统计，包括rejectedPlan。一般我们在查询优化的时候，只需要关注queryPlanner， executionStats即可，因为queryPlanner为我们选择出了winningPlan， 而executionStats为我们统计了winningPlan的所有关键数据。

```shell
"allPlansExecution" : [
      {
         "nReturned" : <int>,
         "executionTimeMillisEstimate" : <int>,
         "totalKeysExamined" : <int>,
         "totalDocsExamined" :<int>,
         "executionStages" : {
            "stage" : <STAGEA>,
            "nReturned" : <int>,
            "executionTimeMillisEstimate" : <int>,
            ...
            "inputStage" : {
               "stage" : <STAGEB>,
               ...
               "inputStage" : {
                 ...
               }
            }
         }
      },
      ...
]
```



##### 3.2.4  explain 执行结果 stage的类型和意义

MongoDB 的文档中列出了前4种类型，还有一些没有列出来，但是会比较常见，这里一并解释一下。

| stage           | 含义                                                        |
| --------------- | ----------------------------------------------------------- |
| COLLSCAN        | 全表扫描                                                    |
| IXSCAN          | 索引扫描                                                    |
| FETCH           | 根据索引去检索指定document                                  |
| SHARD_MERGE     | 各个分片返回数据进行合并                                    |
| SORT            | 表明在内存中进行了排序（与前期版本的scanAndOrder:true一致） |
| SORT_MERGE      | 表明在内存中进行了排序后再合并                              |
| LIMIT           | 使用limit限制返回数                                         |
| SKIP            | 使用skip进行跳过                                            |
| IDHACK          | 针对\_id进行查询                                            |
| SHARDING_FILTER | 通过mongos对分片数据进行查询                                |
| COUNT           | 利用db.\<collection>.count()之类进行count运算               |
| COUNTSCAN       | count不使用用Index进行count时的stage返回                    |
| COUNT_SCAN      | count使用了Index进行count时的stage返回                      |
| SUBPLA          | 未使用到索引的$or查询的stage返回                            |
| TEXT            | 使用全文索引进行查询时候的stage返回                         |

