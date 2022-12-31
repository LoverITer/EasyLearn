## Spring Cloud 中级 — Spring Cloud Bus分布式配中心



## 一、Spring Cloud Bus简介

上一篇文章我们聊了Spring Cloud Config配置中心，当我们在更新github上面的配置以后，如果想要获取到最新的配置，需要手动刷新或者利用webhook的机制每次提交代码发送请求来刷新客户端，客户端越来越多的时候，需要每个客户端都执行一遍，这种方案就不太适合了。使用Spring Cloud Bus（国人很形象的翻译为消息总线）可以完美解决这一问题。

### 1.1 Spring Cloud Bus 是什么？

Spring Cloud Bus（一下简称Bus）通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其他的消息指令。Bus 的一个核心思想是通过分布式的启动器对Spring Boot应用进行扩展，也可以用来建立一个多个应用之间的通信频道。目前唯一实现的方式是用AMQP消息代理作为通道，同样特性的设置（有些取决于通道的设置）在更多通道的文档中。

大家可以将它理解为管理和传播所有分布式项目中的消息既可，其实本质是利用了MQ的广播机制在分布式的系统中传播消息，目前常用的有Kafka和RabbitMQ。利用bus的机制可以做很多的事情，其中配置中心客户端刷新就是典型的应用场景之一，我们用一张图来描述bus在配置中心使用的机制。

![img](https://image.easyblog.top/configbus1.jpg)

根据此图我们可以看出利用Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Bus
3. Bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置





![](https://image.easyblog.top/image-20211001131208490.png)



### 1.2 为什么被称为总线？实现基本原理是什么？

#### 1、什么是总线

在微服务架构的系统中，通常会使用**轻量级的消息代理**来构建一个**共用的消息[主题](https://www.jb51.cc/tag/zhuti/)**，并让系统中所有微服务实例都连接上来。由于**该[主题](https://www.jb51.cc/tag/zhuti/)中产生的消息会被所有实例监听和消费，所以称之为消息总线**。总线上的各个实例，都可以方便地广播一些需要让其他连接在该[主题](https://www.jb51.cc/tag/zhuti/)上的实例都知道的消息。

#### 2、基本原理

ConfigClient 实例都监听MQ中同一个topic【默认是Spring Cloud Bus】。当一个服务刷新数据的时候，它会把这个消息放入Topic中，这样其他监听同一Topic的服务就能够得到[通知](https://www.jb51.cc/tag/tongzhi/)，然后去更新自身的配置。



## 二、Spring Cloud Bus 动态刷新全局广播

Bus支持两种消息代理：RabbitMQ和Kafka。本篇将基于RabbitMQ实现Bus的消息代理，因此在配置之前我们需要首先安装好RabbitMQ。可以参考[RabbitMQ入门安装]() 进行安装配置。

### 2.1 利用消息总线触发一个客户端 `/bus/refresh`,而刷新所有客户端的配置

<img src="https://image.easyblog.top/image-20211001211557031.png" style="width:70%;" />

### 2.2  利用消息总线触发一个服务端ConfigServer的`/bus/refresh`端点,而刷新所有客户端的配置（推荐)

<img src="https://image.easyblog.top/image-20211001211657403.png" style="width:70%;" />

相比之下，2.2 设计的架构显然更加合适，2.1 不适合的原因如下：

1. **打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新职责**
2. **破坏了微服务各节点的对等性**
3. **有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改**

下面步骤基于2.2架构的设计思想构建Config 配置全局动态刷新广播

#### 1、分别在在Config server 和clinet 项目的pom.xml中新增消息总线RabbitMQ依赖

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

#### 2、修改Config server项目的 application.yml 文件新增rabbitmq以及Bus相关配置

```yml
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
spring: 
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    
#rabbitmq相关配置 暴露bus刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

#### 3、在Config client项目的 bootstrap.yml 文件中新增rabbitmq 相关配置

```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

#### 4、改变配置信息的版本号，并向Config server发送一次POST请求

```shell
$ curl -X POST "http://localhost:3344/actuator/bus-refresh"
```

再次刷新本地3355端口的 config client，已经成功实现一次通过，处处更新。

### 原理回顾

所有 Config Client 实例都监听RabbitMQ中同一个topic（默认是`SpringCloudBus`）。当一个服务刷新数据的时候，它会把这个消息放入Topic中，这样其他监听同一Topic的服务就能够得到通过，然后去更新自身的配置。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211001214742.png" style="width:75%;" />



## 三、Spring Cloud Bus 动态刷新定点通知

我们已经通过Spring Cloud Bus + RabbitMQ实现了一处通过，全局广播，处处更新。那，如果我只想通过其中某个client更新呢？如何定点通过只指定某个实例生效呢？

比如我们可以通过通知的url指定实例的destination：`http://localhost:3344/actuator/bus-refresh/{destination}`，此时`bus/refresh` 通过会通过destination参数类指定需要更新配置的服务或实例。

比如，只通知3355端口Config client可以发送下面这个请求：

```shelll
$ curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"
```

执行完上面的命令后再次刷新在本地3344和3355启动的Config client服务，发现只有3355得到通知而配置跟随改变，3344由于没有收到通知而配置没有同步改变。



## 四、总结

![](https://image.easyblog.top/image-20211001215247485.png)