## MongoDB从入门到精通—08 Java整合MongoDB

本篇文章主要是介绍Java整合MongoDB，文章主要介绍使用原生MongoDB Java Driver 以及 spring-data-mongodb 来操作MongoDB的方式。



### 一、MongoDB Java Driver

MongoDB的java驱动程序主要时在java中实现MongoDB，相当于JDBC驱动，我们如果想在java应用程序中访问和使用MongoDB，就需要使用该驱动程序，mongodb-driver 驱动程序程序是一个库，提供了java应用程序中访问MongoDB服务器所需的对象和功能。详细的大家可以参考[MongoDB 驱动程序的官方文档](https://www.mongodb.com/docs/drivers/)，点击链接进去选择Java驱动。

#### 1、添加MongoDB驱动包依赖

创建Maven项目等步骤这里直接跳过，本文默认屏幕前的各位观众老爷已经创建好了一个基础的Maven项目，所以接下来就是导入驱动包依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver</artifactId>
        <version>3.8.2</version>
    </dependency>
</dependencies>
<dependencies>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongo-java-driver</artifactId>
        <version>3.6.3</version>
    </dependency>
</dependencies>
```

> MongoDB驱动程序mongodb-driver是同步更新的Java驱动程序，它包含传统API以及符合新的跨驱动程序(cross-driver)CRUD规范的新的通用MongoCollection接口。
>
> 注：mongodb-driver不是OSGi包：mongodb-driver和mongodb-driver-core（mongodb-driver的依赖）都是com.mongodb包中的类。 对于基于OSGi的应用程序，请改用mongo-java-driver。



#### 2、连接数据库

MongoDB 驱动包下的  `com.mongodb.MongoClient` 类提供了连接到MongoDB服务器和访问数据库的功能。要在java应用程序中实现MongoDB，首先需要创建一个 `MongoClient` 对象实例，然后就可以使用它来访问数据库、设置写入关注以及执行其他操作。

【**示例1**】

```java
import com.mongodb.MongoClient;
import com.mongodb.client.MongoDatabase;

public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";

    public static void main(String[] args) {
       try{
           // 获取到mongodb连接
           MongoClient client = new MongoClient(HOST, PORT);
           // 连接到数据库
           MongoDatabase database = client.getDatabase(DATABASE);

           System.out.printf("Get mongodb connection successfully => %s\n",database.toString());
       }catch (Exception e){
           System.err.println( e.getClass().getName() + ": " + e.getMessage() );
       }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416212104.png)



【**示例2**】

示例1 中 Mongo 数据库无需用户名密码验证。如果你的 Mongo 需要验证用户名及密码，可以参考使用以下代码：

```java
import com.mongodb.MongoClient;
import com.mongodb.MongoClientURI;
import com.mongodb.MongoCredential;
import com.mongodb.ServerAddress;
import com.mongodb.client.MongoDatabase;

import java.util.ArrayList;
import java.util.List;


public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";

    private final static String USER_NAME="root";
    private final static String PASSWORD="admin123";

    public static void main(String[] args) {
       try{
           //连接到MongoDB服务 如果是远程连接可以替换“localhost”为服务器所在IP地址
           //ServerAddress()两个参数分别为 服务器地址 和 端口
           ServerAddress serverAddress= new ServerAddress(HOST,PORT);
           List<ServerAddress> addrs = new ArrayList<>();
           addrs.add(serverAddress);

           MongoCredential credential = MongoCredential
               .createCredential(USER_NAME,  DATABASE, PASSWORD.toCharArray());
           List<MongoCredential> credentials = new ArrayList<>();
           credentials.add(credential);

           // 获取到mongodb连接
           MongoClient client = new MongoClient(addrs,credentials);
           // 连接到数据库
           MongoDatabase database = client.getDatabase(DATABASE);

           System.out.printf("Get mongodb connection successfully => %s\n", database);
       }catch (Exception e){
           System.err.println( e.getClass().getName() + ": " + e.getMessage() );
       }
    }
}
```

> **提示**：除了代码中演示的，MongoDBClient 还有很多种构造方法 以及API方法，篇幅原因这大家可以参考MongoClient的[官方文档](http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/MongoClient.html)



#### 3、查询集合

我们可以使用 `com.mongodb.client.MongoDatabase` 类的 `getCollection()` 方法来获取一个集合

【**示例**】

```java
import com.mongodb.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE="comment";

    public static void main(String[] args) {
       try{
           // 获取到mongodb连接
           MongoClient client = new MongoClient(HOST, PORT);
           // 连接到数据库
           MongoDatabase database = client.getDatabase(DATABASE);

           System.out.printf("Get mongodb connection successfully => %s\n", database);

           // 获取集合（表）
           MongoCollection<Document> collection = database.getCollection(TABLE);
           System.out.printf("Get collection successfully => %s\n",collection);
       }catch (Exception e){
           System.err.println( e.getClass().getName() + ": " + e.getMessage() );
       }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416214528.png)



#### 4、查询文档

使用 `com.mongodb.client.MongoCollection` 的 `find()` 以及其重载方法可以查询 MongodB 文档数据

【**示例1**】

查询所有文档数据：find 方法在不传如任何参数时会查询出集合中所有数据并返回 ` FindIterable<Document>` 类型对象，遍历这个对象就可以获取到具体 Document 数据，具体 用法参数如下示例：

```java
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;


public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE="comment";

    public static void main(String[] args) {
       try{
           // 获取到mongodb连接
           MongoClient client = new MongoClient(HOST, PORT);
           // 连接到数据库
           MongoDatabase database = client.getDatabase(DATABASE);

           System.out.printf("Get mongodb connection successfully => %s\n", database);

           // 获取集合（表）
           MongoCollection<Document> collection = database.getCollection(TABLE);
           System.out.printf("Get collection successfully => %s\n",collection);

           FindIterable<Document> documents = collection.find();
           for (Document document : documents) {
               System.out.println(document);
           }
       }catch (Exception e){
           System.err.println( e.getClass().getName() + ": " + e.getMessage() );
       }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416215442.png)



【**示例2**】

根据添加查询文档数据：find 方法可以传入一个Bson对象指定需要查询的条件，具体用法参考如下示例：

```java
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.BsonDocument;
import org.bson.Document;

public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE="comment";

    public static void main(String[] args) {
       try{
           // 获取到mongodb连接
           MongoClient client = new MongoClient(HOST, PORT);
           // 连接到数据库
           MongoDatabase database = client.getDatabase(DATABASE);

           System.out.printf("Get mongodb connection successfully => %s\n", database);

           // 获取集合（表）
           MongoCollection<Document> collection = database.getCollection(TABLE);
           System.out.printf("Get collection successfully => %s\n",collection);

           // 查询 article_id 等于 10001 的数据
           FindIterable<Document> documents = collection
                                               .find(BsonDocument.parse("{'article_id':'10001'}"));
           for (Document document : documents) {
               System.out.println(document);
           }
       }catch (Exception e){
           System.err.println( e.getClass().getName() + ": " + e.getMessage() );
       }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416220653.png)





#### 5、插入文档

使用 `com.mongodb.client.MongoCollection` 类的 `insertOne() / insertMany()` 方法来插入一条数据或多条数据

【**示例1**】

使用 insertOne() 方法插入一条文档数据，当然 insertMany() 方法也可以插入一条数据，一条只是多条的一个特例。

```java
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.BsonDocument;
import org.bson.Document;

import java.time.LocalDateTime;

public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE = "comment";

    public static void main(String[] args) {
        try {
            // 获取到mongodb连接
            MongoClient client = new MongoClient(HOST, PORT);
            // 连接到数据库
            MongoDatabase database = client.getDatabase(DATABASE);

            System.out.printf("Get mongodb connection successfully => %s\n", database);

            // 获取集合（表）
            MongoCollection<Document> collection = database.getCollection(TABLE);
            System.out.printf("Get collection successfully => %s\n", collection);

            Document doc = new Document("article_id", "10006")
                    .append("content", "研究表明，刚烧开的开水不能立即就喝，因为烫嘴！")
                    .append("user_id", "1006")
                    .append("nickname", "frank.huang")
                    .append("create_time", LocalDateTime.now())
                    .append("like_num", 10000000);
            // 插入一条文档数据到MongoDB
            collection.insertOne(doc);

            // 查询 article_id 等于 10006 的数据
            FindIterable<Document> documents = collection.find(BsonDocument.parse("{'article_id':'10006'}"));
            for (Document document : documents) {
                System.out.println(document);
            }
        } catch (Exception e) {
            System.err.println(e.getClass().getName() + ": " + e.getMessage());
        }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416222128.png)



【**示例2**】

insertOne() 方法一次只能插入一条数据，如果一次待插入的数据在2条及以上，我们完全可以使用 insertMany() 方法：

```java
import com.google.common.collect.Lists;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.BsonArray;
import org.bson.BsonDocument;
import org.bson.Document;

import java.time.LocalDateTime;


public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE = "comment";

    public static void main(String[] args) {
        try {
            // 获取到mongodb连接
            MongoClient client = new MongoClient(HOST, PORT);
            // 连接到数据库
            MongoDatabase database = client.getDatabase(DATABASE);

            System.out.printf("Get mongodb connection successfully => %s\n", database);

            // 获取集合（表）
            MongoCollection<Document> collection = database.getCollection(TABLE);
            System.out.printf("Get collection successfully => %s\n", collection);

            Document doc1 = new Document("article_id", "10007")
                    .append("content", "切记不要空腹吃饭。！")
                    .append("user_id", "1006")
                    .append("nickname", "frank.huang")
                    .append("create_time", LocalDateTime.now())
                    .append("like_num", 10000000);
            Document doc2 = new Document("article_id", "10008")
                    .append("content", "世界最神奇的东西就是！！专家")
                    .append("user_id", "1008")
                    .append("nickname", "fei.zhang")
                    .append("create_time", LocalDateTime.now())
                    .append("like_num", 10000000);
            Document doc3 = new Document("article_id", "10009")
                    .append("content", "某知名养生专家，享年59岁")
                    .append("user_id", "1009")
                    .append("nickname", "frank.huang")
                    .append("create_time", LocalDateTime.now())
                    .append("like_num", 10000000);
            collection.insertMany(Lists.newArrayList(doc1,doc2,doc3));

            FindIterable<Document> documents = collection.find();
            for (Document document : documents) {
                System.out.println(document);
            }
        } catch (Exception e) {
            System.err.println(e.getClass().getName() + ": " + e.getMessage());
        }
    }
}
```



#### 6、更新文档

使用 `com.mongodb.client.MongoCollection` 类中的 `updateOne()/updateMany()` 方法来更新集合中的文档

【**示例**】

updateOne() 方法只会更新匹配的第一条记录，如果需要更新条件所匹配的所有文档则需要必须使用 updateMany()，因此一般情况下我们直接使用updateMany() 方法即可：

```java
import com.google.common.collect.Lists;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Filters;
import org.bson.BsonArray;
import org.bson.BsonDocument;
import org.bson.Document;

import java.time.LocalDateTime;

public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE = "comment";

    public static void main(String[] args) {
        try {
            // 获取到mongodb连接
            MongoClient client = new MongoClient(HOST, PORT);
            // 连接到数据库
            MongoDatabase database = client.getDatabase(DATABASE);

            System.out.printf("Get mongodb connection successfully => %s\n", database);

            // 获取集合（表）
            MongoCollection<Document> collection = database.getCollection(TABLE);
            System.out.printf("Get collection successfully => %s\n", collection);

            collection.updateMany(Filters.eq("user_id",1008),
                                  new Document("$set",new Document("nickname","王富贵")));

            FindIterable<Document> documents = collection.find(Filters.eq("user_id",1008));
            for (Document document : documents) {
                System.out.println(document);
            }
        } catch (Exception e) {
            System.err.println(e.getClass().getName() + ": " + e.getMessage());
        }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416231306.png)



#### 7、删除文档

使用 `com.mongodb.client.MongoCollection` 类中的 `deleteOne()/deleteMany()` 方法来删除集合中的文档

【**示例**】

```java
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Filters;
import org.bson.Document;


public class MongoJDBC {

    private final static String HOST = "47.99.161.205";
    private final static int PORT = 27017;
    private final static String DATABASE = "easyblog_dev";
    private final static String TABLE = "comment";

    public static void main(String[] args) {
        try {
            // 获取到mongodb连接
            MongoClient client = new MongoClient(HOST, PORT);
            // 连接到数据库
            MongoDatabase database = client.getDatabase(DATABASE);

            System.out.printf("Get mongodb connection successfully => %s\n", database);

            // 获取集合（表）
            MongoCollection<Document> collection = database.getCollection(TABLE);
            System.out.printf("Get collection successfully => %s\n", collection);

            // 删除所有nickname=王富贵的数据
            collection.deleteMany(Filters.eq("nickname","王富贵"));

            //再次查询所有nickname=王富贵的数据
            FindIterable<Document> documents = collection.find(Filters.eq("nickname","王富贵"));
            for (Document document : documents) {
                System.out.println(document);
            }
        } catch (Exception e) {
            System.err.println(e.getClass().getName() + ": " + e.getMessage());
        }
    }
}
```

运行结果：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220416231716.png)



### 二、Spring Data MongoDB

上面大体介绍了基于MongoDB提供的整合java的原生驱动MongoDB Java Driver如何操作MongoDB，整体使用起来还是比较费劲的，不过好在 Spring 封装的`spring-data-mongodb`提供了操作的MongoDB的另一种方法。

Spring Data其实是一个高级别的Spring Source项目，而Spring Data MongoDB仅仅是其中的一个子项目。Spring Data旨在为关系型数据库、非关系型数据、Map-Reduce框架、云数据服务等等提供统一的数据访问API。
无论是哪种持久化存储， 数据访问对象（或称作为DAO，即Data Access Objects）通常都会提供对单一域对象的CRUD （创建、读取、更新、删除）操作、查询方法、排序和分页方法等。Spring Data则提供了基于这些层面的统一接口（CrudRepository，PagingAndSortingRepository）以及对持久化存储的实现。

#### 1、添加 spring-data-mongodb 依赖

直接使用 spring-data-mongodb 还是比较费劲，因为还需要写一大堆的配置，Spring 已经提供了MongoDB 场景启动器，直接添加MongoDB场景启动器，然后通过yml文件配置MongoDB连接信息就可以了。

```xml
<dependency>                                               
  <groupId>org.springframework.boot</groupId>              
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
  <version>2.1.8.RELEASE</version> <!--版本随意，这里选择2.1.8.RELEASE-->                        
</dependency> 
```



#### 2、创建application.yml并提添加MongoDB配置 

```yml
spring:
  data:
    mongodb:
      host: 47.99.161.205   #mongodb地址
      database: comment     #链接的数据库
      port: 27017           #端口
```



#### 3、创建启动类

```java
@SpringBootApplication
public class MongodbDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MongodbDemoApplication.class, args);
    }

}
```



#### 4、创建MongoDB数据持久化对象

和使用MyBatis操作数据库一样，使用 spring-data-mongodb 操作MongoDB也需要提供一下数据持久化对象：

```java
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

import java.time.LocalDateTime;
import java.util.Date;


@Data
@Document(collection = "comment")
public class Comment {
    //主键标识，该属性的值会自动对应mongodb的主键字段"_id"，如果该属性名就叫“id”,则该注解可以省略，否则必须写
    @Id
    private String id;//主键
    private String content;//评论内容
    //该属性对应mongodb的字段的名字，如果一致，则无需该注解
    @Field("publish_time")
    private Date publishTime;//发布日期
    //添加了一个单字段的索引  不建议这么干
    //@Indexed
    @Field("user_id")
    private String userId;//发布人ID
    private String nickname;//昵称
    @Field("create_time")
    private LocalDateTime createTime;//评论的日期时间
    @Field("like_num")
    private Integer likeNum;//点赞数
    @Field("reply_num")
    private Integer replyNum;//回复数
    private String state;//状态
    @Field("parent_id")
    private String parentId;//上级ID
    @Field("article_id")
    private String articleId;
}
```

需要注意的点：

* 1、MongoDB的持久化对象需要使用 Spring Data MongoDB 包下的 `Document` 注解标记其为一个MongoDB文档，可以通过collection参数指定这个类对应的集合（表）。
* 2、可以使用Spring Data MongoDB 包下的 `@Id` 注解指定字段为主键，如果该字段的名称就是 id 则可以省略此注解。
* 3、持计划对象中的字段名称和MongoDB中的字段名称不一致时，应该使用 Spring Data MongoDB 包下的`Field` 注解指明其在MongoDB中的名称。
* 4、可以使用Spring Data MongoDB 包下的 `Indexed` 注解或 `CompoundIndex` 指定单字段索引或者复合索引，但是一般不建议这么干。



#### 5、使用 MongoRepository 操作MongoDB

##### 5.1 继承 MongoRepository 实现 CommentRepository

一些简单的查询可以通过继承 `org.springframework.data.mongodb.repository.MongoRepository` 接口来实现，有点像MyBatis-Plus，接口内部已经给生成了CRUD的默认方法，我们只需再写一个关联到具体集合的接口就可以直接使用了。如下实现Comment表的操作接口 `CommentRepository`

```java
import com.example.mongodb.dao.model.Comment;
import org.springframework.data.mongodb.repository.MongoRepository;


public interface CommentRepository extends MongoRepository<Comment,String> {
}
```



##### 5.2 实现CRUDU业务类

有了 CommentRepository 之后就可以在service类中直接使用了，如下实现 comment 集合的简单CRUD:

```java
import com.example.mongodb.dao.mapper.CommentRepository;
import com.example.mongodb.dao.model.Comment;
import com.example.mongodb.request.QueryCommentRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;


@Service
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    /**
     * 保存评论
     *
     * @param comment
     */

    public Comment saveComment(Comment comment) {
        return commentRepository.save(comment);
    }

    /**
     * 更新评论
     *
     * @param comment
     */
    public Comment updateComment(Comment comment) {
        return commentRepository.save(comment);
    }

    /**
     * 通过id查询
     *
     * @param id
     * @return
     */
    public Comment queryById(String id) {
        return commentRepository.findById(id).orElse(null);
    }


    /**
     * 查询所有评论
     *
     * @return
     */
    public List<Comment> queryCommentList() {
        return commentRepository.findAll();
    }


    /**
     * 根据id删除
     *
     * @param id
     */
    public void deleteById(String id) {
        commentRepository.deleteById(id);
    }

    /**
     * @param request
     * @return
     */
    public Comment queryCommentByRequest(QueryCommentRequest request) {
        return commentRepository.findById(request.getId()).orElse(null);
    }
}
```



##### 5.3 控制器层提供CRUD接口

```java
import com.example.mongodb.dao.model.Comment;
import com.example.mongodb.service.CommentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/comment")
public class CommentController {

    @Autowired
    private CommentService commentService;

    @GetMapping("/{id}")
    public Comment queryById(@PathVariable("id") String id) {
        return commentService.queryById(id);
    }

    @GetMapping("/list")
    public List<Comment> queryList() {
        return commentService.queryCommentList();
    }

    @PostMapping
    public Comment save(@RequestBody Comment comment) {
        return commentService.saveComment(comment);
    }

    @PutMapping
    public Comment update(@RequestBody Comment comment) {
        return commentService.updateComment(comment);
    }

    @DeleteMapping
    public void deleteById(@RequestParam("id") String id) {
        commentService.deleteById(id);
    }

}
```





#### 6、使用 MongoTemplate 操作 MongoDB

一些复杂的查询或者更新操作可以使用 MongoTemplate  来操作，MongoTemplate  中的 `org.springframework.data.mongodb.core.query.Criteria` 可以拼接复杂的查询条件

【**示例1**】

多条件复杂查询：

```java
import com.example.mongodb.dao.mapper.CommentRepository;
import com.example.mongodb.dao.model.Comment;
import com.example.mongodb.request.QueryCommentRequest;
import com.google.common.collect.Iterables;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Example;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.List;
import java.util.Objects;


@Service
public class CommentService {

    @Autowired
    private MongoTemplate mongoTemplate;

    /**
     * 根据请求条件查询
     *
     * @param request
     * @return
     */
    public Comment queryByRequest(QueryCommentRequest request) {
        Criteria criteria = new Criteria();
        if (Boolean.FALSE.equals(StringUtils.isEmpty(request.getId()))) {
            criteria.and("_id").is(request.getId());
        }
        if (Boolean.FALSE.equals(StringUtils.isEmpty(request.getArticleId()))) {
            criteria.and("article_id").is(request.getArticleId());
        }
        if (Boolean.FALSE.equals(StringUtils.isEmpty(request.getUserId()))) {
            criteria.and("user_id").is(request.getUserId());
        }
        Query query = Query.query(criteria);
        return Iterables.getFirst(mongoTemplate.find(query, Comment.class), null);
    }

}
```



【**示例2**】

多条件复杂查询：

```java
package com.example.mongodb.service;

import com.example.mongodb.dao.mapper.CommentRepository;
import com.example.mongodb.dao.model.Comment;
import com.example.mongodb.request.QueryCommentRequest;
import com.example.mongodb.request.UpdateCommentRequest;
import com.google.common.collect.Iterables;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Example;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.List;
import java.util.Objects;


@Service
public class CommentService {

    @Autowired
    private MongoTemplate mongoTemplate;

    /**
    *  更新文档数据
    */
    public void updateByRequest(UpdateCommentRequest request) {
        Query query = Query.query(Criteria.where("_id").is(request.getId()));
        Update update = new Update();
        if(Boolean.FALSE.equals(StringUtils.isEmpty(request.getArticleId()))){
            update.set("artcile_id",request.getArticleId());
        }
        if(Boolean.FALSE.equals(StringUtils.isEmpty(request.getUserId()))){
            update.set("user_id",request.getUserId());
        }
        if(Boolean.FALSE.equals(StringUtils.isEmpty(request.getContent()))){
            update.set("content",request.getContent());
        }
        if(Boolean.FALSE.equals(StringUtils.isEmpty(request.getNickname()))){
            update.set("nickname",request.getNickname());
        }
        if(Boolean.FALSE.equals(StringUtils.isEmpty(request.getLikeNum()))){
            update.set("like_num",request.getLikeNum());
        }
        //....

        // 三个参数：
        // 参数1: 更新匹配条件
        // 参数2: 更新对象
        // 参数3：集合的名字或实体类的类型
        mongoTemplate.updateMulti(query, update, Comment.class);
    }

}
```

