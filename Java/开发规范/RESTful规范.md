###  知海拾贝

###  一、RESTful 规范

​		RESTful API是目前比较成熟的一套互联网应用程序的API设计理论。REST（Representational State Transfer）表述性状态转换，REST指的是一组架构约束条件和原则。 如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。


#### 1.1 URI规范

URI 表示资源，资源一般对应服务器端领域模型中的实体类。URI规范规定路径中：

* 不用大写；
* 用中杠`-`不用下杠`_`；
* 参数列表要encode；
* URI中的名词表示资源集合，使用复数形式。

#### 1.2 Request请求方法

**GET：查询**

```text
GET /zoos
GET /zoos/1
GET /zoos/1/employees
```



**POST：创建单个资源。POST一般向“资源集合”型uri发起**

```text
POST /animals  //新增动物
POST /zoos/1/employees //为id为1的动物园雇佣员工
```


**PUT：更新单个资源（全量）**，客户端提供完整的更新后的资源。与之对应的是 PATCH，**PATCH 负责部分更新**，客户端提供要更新的那些字段。

```text
PUT /animals/1
PUT /zoos/1
```

**DELETE：删除**

```text
DELETE /zoos/1/employees/2
DELETE /zoos/1/employees/2;4;5
DELETE /zoos/1/animals  //删除id为1的动物园内的所有动物
```



#### 1.3  安全性和幂等性

1. 安全性：不会改变资源状态，可以理解为只读的；
2. 幂等性：执行1次和执行N次，对资源状态改变的效果是等价的。

| .      | 安全性 | 幂等性 |
| ------ | ------ | ------ |
| GET    | √      | √      |
| POST   | ×      | ×      |
| PUT    | ×      | √      |
| DELETE | ×      | √      |



#### 1.4 只用以下常见的3种body format：

1. **Content-Type: application/json**

```http
POST /v1/animal HTTP/1.1
Host: api.example.org
Accept: application/json
Content-Type: application/json
Content-Length: 24

{   
  "name": "Gir",
  "animalType": "12"
}
```

2. **Content-Type: application/x-www-form-urlencoded** (浏览器POST表单用的格式)

```http
POST /login HTTP/1.1
Host: example.com
Content-Length: 31
Accept: text/html
Content-Type: application/x-www-form-urlencoded

username=root&password=Zion0101
```

3. **Content-Type: multipart/form-data;** boundary=—-RANDOM_jDMUxq4Ot5 (表单有文件上传时的格式)



#### 1.5 Response

（1）各HTTP方法成功处理后的数据格式：

| 请求方法  | response 格式  |
| --------- | -------------- |
| GET       | 单个对象、集合 |
| POST      | 新增成功的对象 |
| PUT/PATCH | 更新成功的对象 |
| DELETE    | 空             |



（2） 分页response

```json
{
    "paging":{"limit":10,"offset":0,"total":729},
    "data":[{},{},{}...]
}
```





#### 1.6 错误处理

1. 不要发生了错误但给2xx响应，客户端可能会缓存成功的http请求；

2. 正确设置http状态码，不要自定义；

3. Response body 提供 1) 错误的代码（日志/问题追查）；2) 错误的描述文本（展示给用户）。

对第三点的实现稍微多说一点：

​		Java 服务器端一般用异常表示 RESTful API 的错误。API 可能抛出两类异常：**业务异常和非业务异常**。业务异常由自己的业务代码抛出，表示一个用例的前置条件不满足、业务规则冲突等，比如参数校验不通过、权限校验失败。非业务类异常表示不在预期内的问题，通常由类库、框架抛出，或由于自己的代码逻辑错误导致，比如数据库连接失败、空指针异常、除0错误等等。



**业务类异常必须提供2种信息**：

1. 如果抛出该类异常，HTTP 响应状态码应该设成什么；
2. 异常的文本描述；

**在Controller层使用统一的异常拦截器：**

1. 设置 HTTP 响应状态码：对业务类异常，用它指定的 HTTP CODE；对非业务类异常，统一500；
2. Response Body 的错误码：异常类名
3. Response Body 的错误描述：对业务类异常，用它指定的错误文本；对非业务类异常，线上可以统一文案如“服务器端错误，请稍后再试”，开发或测试环境中用异常的 stacktrace，服务器端提供该行为的开关。

**常用的http状态码及使用场景：**

<table><thead><tr><th align="left">状态码</th><th>使用场景</th></tr></thead><tbody><tr><td align="left">400 bad request</td><td>常用在参数校验</td></tr><tr><td align="left">401 unauthorized</td><td>未经验证的用户，常见于未登录。如果经过验证后依然没权限，应该 403（即 authentication 和 authorization 的区别）。</td></tr><tr><td align="left">403 forbidden</td><td>无权限</td></tr><tr><td align="left">404 not found</td><td>资源不存在</td></tr><tr><td align="left">500 internal server error</td><td>非业务类异常</td></tr><tr><td align="left">503 service unavaliable</td><td>由容器抛出，自己的代码不要抛这个异常</td></tr></tbody></table>



#### 1.7 异步任务

对耗时的异步任务，服务器端接受客户端传递的参数后，应返回创建成功的任务资源，其中包含了任务的执行状态。客户端可以轮训该任务获得最新的执行进度。

```json
提交任务：
POST /batch-publish-msg
[{"from":0,"to":1,"text":"abc"},{},{}...]

返回：
{"taskId":3,"createBy":"Anonymous","status":"running"}

GET /task/3
{"taskId":3,"createBy":"Anonymous","status":"success"}
```

如果任务的执行状态包括较多信息，可以把“执行状态”抽象成组合资源，客户端查询该状态资源了解任务的执行情况。

```json
提交任务：
POST /batch-publish-msg
[{"from":0,"to":1,"text":"abc"},{},{}...]

返回：
{"taskId":3,"createBy":"Anonymous"}

GET /task/3/status
{"progress":"50%","total":18,"success":8,"fail":1}
```



#### 1.8 API的演进

版本常见的三种方式：

1、在uri中放版本信息：GET /v1/users/1

2、Accept Header：Accept: application/json+v1

3、自定义 Header：X-Api-Version: 1

**建议使用第一种，虽然没有那么优雅，但最明显最方便。**

**URI失效**
随着系统发展，总有一些API失效或者迁移，对失效的API，返回404 not found 或 410 gone；对迁移的API，返回 301 重定向。





### 二、Java单元测试利器—Mockito

#### 1. Mock依赖

```xml
<!-- test -->
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4-legacy</artifactId>
    <version>${powermock.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>${powermock.version}</version>
    <scope>test</scope>
</dependency>
```



#### 2. Mock的概念

所谓的mock就是创建一个类的**虚假**的对象，在测试环境中，用来替换掉真实的对象，以达到两大目的：

1. **验证**：验证这个对象的某些方法的调用情况，调用了多少次，参数是什么等等
2. **确认**：指定这个对象的某些方法的行为，返回特定的值，或者是执行特定的动作



**几个常用的测试注解**：

* **@InjectMocks**：通过创建一个实例，它可以调用真实代码的方法，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中。
* **@Mock**：对函数的调用均执行mock（即虚假函数），不执行真正部分。
* **@Spy**：对函数的调用均执行真正部分。



#### 3. 常用的Mockito方法

Mockito的使用，一般有以下几种组合：参考链接

* do/when：包括doThrow(…).when(…)/doReturn(…).when(…)/doAnswer(…).when(…)
* given/will：包括given(…).willReturn(…)/given(…).willAnswer(…)
* when/then: 包括when(…).thenReturn(…)/when(…).thenAnswer(…)/when(…).thenThrow(…)

Mockito 有多种匹配函数，部分如下：

<table><thead><tr><th align="left">函数名</th><th align="left">匹配类型</th></tr></thead><tbody><tr><td align="left">any()</td><td align="left">所有对象类型</td></tr><tr><td align="left">anyInt()</td><td align="left">基本类型 int、非 null 的 Integer 类型</td></tr><tr><td align="left">anyChar()</td><td align="left">基本类型 char、非 null 的 Character 类型</td></tr><tr><td align="left">anyShort()</td><td align="left">基本类型 short、非 null 的 Short 类型</td></tr><tr><td align="left">anyBoolean()</td><td align="left">基本类型 boolean、非 null 的 Boolean 类型</td></tr><tr><td align="left">anyDouble()</td><td align="left">基本类型 double、非 null 的 Double 类型</td></tr><tr><td align="left">anyFloat()</td><td align="left">基本类型 float、非 null 的 Float 类型</td></tr><tr><td align="left">anyLong()</td><td align="left">基本类型 long、非 null 的 Long 类型</td></tr><tr><td align="left">anyByte()</td><td align="left">基本类型 byte、非 null 的 Byte 类型</td></tr><tr><td align="left">anyString()</td><td align="left">String 类型(不能是 null)</td></tr><tr><td align="left">anyList()</td><td align="left">List 类型(不能是 null)</td></tr><tr><td align="left">anyMap()</td><td align="left">Map&lt;K, V&gt;类型(不能是 null)</td></tr></tbody></table>





### Git在线学习网站

https://learngitbranching.js.org/?locale=zh_CN



### Java五子棋（人机版），昨天买的棋子今天就用不上了

https://blog.csdn.net/dkm123456/article/details/118904917?utm_medium=distribute.pc_feed_v2.none-task-blog-hot_rank_bottoming-13.pc_personrecdepth_1-utm_source=distribute.pc_feed_v2.none-task-blog-hot_rank_bottoming-13.pc_personrec

