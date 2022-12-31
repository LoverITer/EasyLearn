## MongoDb从入门到精通—04 MongoDB数据库操作

**MongoDB 数据库（database）可以类比关系型数据库中的数据库（database）**,针对MongoDB数据库的操作主要有创建和删除两种，下面分别介绍具体的用法。



### 一、MongoDB 创建数据库

MongoDB 创建数据库的语法格式如下：

```shell
use DATABASE_NAME
```

如果数据库不存在，则创建数据库，否则切换到指定数据库。

```shell
use article
```

查看有权限查看的所有的数据库命令 

```shell
show dbs
或
show databases
```

查看当前正在使用的数据库

```shell
db
```

> 注意：MongoDB 中默认的数据库为 `test`，如果你没有选择数据库，集合将存放在 `test` 数据库中。



### 二、MongoDB 删除数据库

MongoDB 删除数据库的语法格式如下：

```shell
db.dropDatabase()
```

删除当前数据库，默认为 test，你可以使用 `db` 命令查看当前数据库名。





### 三、小试牛刀

以下实例我们创建了数据库 **easyblog_dev** :

```shell
> use easyblog_dev
switched to db easyblog_dev
> db
easyblog_dev
> 
```

使用 **show dbs** 命令查看全部数据库：

```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> 
```

可以看到，我们刚创建的数据库 easyblog_dev 并不在数据库的列表中， 要显示它，我们需要向 easyblog_dev 数据库插入一些数据。

```
> db.easyblog_dev.insert({"article_id":"100000","name":"MongoDB教程","author":"1001"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin         0.000GB
config        0.000GB
easyblog_dev  0.000GB
local         0.000GB
```

删除 easyblog_dev 数据库：

```shell
> db.dropDatabase()
> { "dropped" : "easyblog_dev", "ok" : 1 }

```

