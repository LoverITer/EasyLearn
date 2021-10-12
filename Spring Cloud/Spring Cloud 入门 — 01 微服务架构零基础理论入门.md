

互联网的快速发展，越来越多的公司开始由单体架构转向微服务架构。因此，微服务的学习需要被我们这些奋斗者们所掌握，在学习微服务之前，我们有必要盘点下所谓的微服务**是什么**，**包含什么**，**解决了什么样的业务场景**。



## 一、背景

### 1.1 传统单体应用

在IT互联网早期，大多软件架构采用的是`单体架构`，所谓单体架构就是将Application中的所有业务模块全部打包在一个文件中进行部署，这种架构使得不同系统间的通信变得异常困难。

<img src="https://upload-images.jianshu.io/upload_images/16751762-76c34a415f39bb33.png?imageMogr2/auto-orient/strip|imageView2/2/w/876" style="zoom:67%;" />

随着业务越来越复杂，将所有的业务模块耦合在一起的单体架构已经非常臃肿，使得代码的`开发`、`维护`、`拓展性`急剧下降。总的来说，单体架构面临的问题如下：

* （1）**代码冗余**
* （2）**可靠性差**：单体架构中的某个模块的bug都有可能导致整个应用的崩溃
* （3）**开发效率低下**
  * 多人维护一个单体应用，频繁的进行代码合并，频繁的解决代码冲突，解决冲突的时间和成本较高
  * 每次上线都要跟最新代码进行合并，重新进行全量功能的回归测试，耗费时间较多
  * 互相协调困难，而且可能会出现别人多次先上线，多次重复的合并代码，解决冲突，全量回归测试，做很多次重复的事情
* （4）**技术架构升级困难**：不能随意升级技术架构，因为升级后的技术可能会对别的团队开发的代码有影响
* （5）**应用臃肿**：所有的业务模块的耦合性太高，耦合性过高并且体量很大的项目势必会给各个技术环节带来挑战。项目越进行到后期，这种难度越大，只要有改动，整个应用都需要重新测试，部署，不仅极大的限制了开发的灵活性，而且也带有巨大的隐患，极易导致项目崩溃。

### 1.2 微服务架构

为了解决单体架构带来的问题，微服务架构应用而生。微服务的概念是大神 ***[Martin Fowler](https://martinfowler.com/)*** 提出的（学习微服务不知道他和学习JUC不知道 `Doug Lea` 是一个性质——白学！）。简单来说，**微服务架构是一种将单体应用拆分一组小型的应用服务的方法，每个服务运行在自己的进程中，服务间通信采用轻量级通信机制。这些服务围绕业务能力构建并且可通过全自动部署机制独立部署。这些服务共用一个最小型的集中式的管理，不同服务可用不同语言开发，使用不同数据存储技术**。

这样的好处是对于每一个简单的问题，`开发`、`维护`、`部署`的难度就降低了很多，可以实现自治，可自主选择最合适的技术框架，提高了项目开发的灵活性。

与此同时，微服务架构不仅是单纯的拆分，拆分之后的各个微服务之间还要进行`通信`，否则就无法协同完成需求。不同的微服务之间可以通过某种协议进行通信，相互调用、协同完成功能，并且各服务之间只需要制定统一的协议即可，至于每个微服务是用什么技术框架来实现的，统统不需要关心。

这种松耦合的方式使得开发、部署都变得更加`灵活`，同时系统更容易扩展，降低了开发、运维的难度，单体应用拆分之后的架构如下图所示。

<img src="https://upload-images.jianshu.io/upload_images/16751762-2df2cf7cd36f9531.png?imageMogr2/auto-orient/strip|imageView2/2/w/855" style="zoom:67%;" />



* **微服务优点：**
  - 单个服务更易于开发、维护
  - 单个服务启动比较快
  - 局部修改容易部署
  - 技术栈不受限
  - 按需伸缩
* **微服务缺点：**
  - 运维要求高
  - 分布式固有的复杂性
  - 重复劳动
* **微服务适用场景：**
  - 大型、复杂项目
  - 有快速迭代的需求
  - 访问压力大



## 二、微服务管理解决方案 Spring Cloud

面对拆分之后的众多微服务，首先需要完成的工作就是实现`服务治理`，包括`服务注册`和`服务发现`。除此之外，还需要考虑微服务间的`负载均衡`、`服务容错`、`分布式配置`、`网关`、`服务监控`等，针对上面一系列问题，Spring Cloud 给出了一套解决方案 。

`Spring Boot` 用于快速搭建基础系统，而`Spring Cloud` 在此基础上实现分布式系统中的公共组件，如`服务注册`、`服务发现`、`配置管理`、`熔断器`，服务调用方式是基于 `REST API`。

下面是官网给出的的SpringCloud架构图

![](https://spring.io/images/cloud-diagram-1a4cad7294b4452864b5ff57175dd983.svg)

提供的能力主要包括：

* 1、服务治理

* 2、服务负载与调用
* 3、服务熔断与降级
* 4、服务网关
* 5、服务分布式配置
* 6、分布式链路追踪

下面，我将在第三节~第八节将详细介绍Spring Cloud中各个组件的功能。

## 三、服务发现 Service discovery

常用的服务注册发现组件有`Eureka`、`Zookeeper`、`Consul`。下面我们一一介绍

### 3.1 Eureka

Eureka是SpringCloud中一个负责服务注册与发现的组件，它分为`eureka server`和`eureka client`。其中eureka server是作为服务的注册与发现中心。eureka client既可以作为服务的生产者，又可以作为服务的消费者。具体结构如下图：

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2F201901.oss-cn-hangzhou.aliyuncs.com%2Faliyun%2F3250440-8058a119cf2a414279c40ac460fe1080.jpg&refer=http%3A%2F%2F201901.oss-cn-hangzhou.aliyuncs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1636629761&t=b15d8333b7032af53250b05a3ff0c4bf)

**（1）Eureka Server**

`Eureka Server`提供服务注册的功能，当各个微服务节点通过配置启动后，会在Eureka Server中进行注册。因此，Eureka Server中的服务注册表中将会存储所有可用服务节点的信息。

**（2）Eureka Client**

`Eureka Client`用于与Eureka Server的交互，它具备一个内置的，使用轮询的负载均衡算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中移除该服务节点。

### 3.2 Zookeeper

`zookeeper`作为一个分布式协调服务，它的应用场景非常多，如`分布式锁`、`配置中心`、`服务注册与发现`等。

使用Zookeeper实现服务注册与发现，主要应用的是Zookeeper的`Znode`数据模型和`Watch`机制。

Zookeeper的数据模型是一个”树形结构“，树由节点组成，树中的每个节点为一个Znode。

而Watch机制可以理解为一个和Znode所绑定的监听器，当这个Znode发生变化，这个监听器监听到这些写操作之后会异步向请求Watch的客户端发送通知。

总的来说，Zookeeper服务注册与发现流程如下图所示：

<img src="https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEscvzFALsFKqtwTRfZwjkg9F04bADcg8SFBib7BDpwGgibDlr9niamyjqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" style="zoom:80%;" />

**（1）服务发现**

服务消费者启动时，会根据本身依赖的服务信息，向Zookeeper服务端获取注册的服务信息并设置Watch，获取到注册的服务信息之后将服务提供者信息缓存在本地，调用服务时直接根据从Zookeeper注册中心获取到的服务注册信息调用服务。

**（2）服务通知**

当服务生产者因为某种原因宕机或不提供服务之后，Zookeeper服务注册中心的对应服务节点会被删除，因为服务消费者在获取服务信息的时候在对应节点上设置了Watch，因此节点删除之后会触发对应的Watcher，Zookeeper注册中心会异步向服务所关联的所有服务消费者发出节点删除的通知，服务消费者根据收到的通知更新缓存的服务列表。



### 3.3 Consul



### 3.4 Nacos



## 四、服务负载与调用

服务负载与调用分为负载均衡与服务调用，服务调用想必大家都已经很清楚了，就是消费者在注册中心获取到提供者的地址后，进行的远程调用。

负载均衡是微服务架构中必须使用到的技术，简单来说，负载均衡就是将用户的请求按照某种策略分配到不同的服务器上，从而实现系统的高可用，常见的负载均衡工具有nginx、lvs等。

![](https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEa2DnG8E4wEoepWcYzYwhBldFBfqIdOKNu2UBTJ2jdn5tjjhBCtby9Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用户的请求先到达`负载均衡器`，负载均衡器会根据负载均衡算法转发到微服务上。微服务的负载均衡也是类似，Spring Cloud 提供了 `Ribbon` 和 `Feign/OpenFeign`

`Ribbon`的全称是`SpringCloud Ribbon`，它的主要功能是提供客户端软件的负载均衡算法，它提供了一系列的完善配置帮助你调用远程的服务。

而`Feign`是SpringCloud组件中一个轻量级Restful的声明式Http客户端（和RestTemplate作用类似），但是现在已经停更维护了，因此现在一般就是用`OpenFeign`。特别需要注意的是，OpenFeign内置了Ribbon，因此，它也可以做客户端的负载均衡。

<img src="https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEiaGgFwbFfaUnABHf2rE8os9XWuAQ9tZvtibwq23I6v1DicRyFIThXJuRA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" style="zoom:80%;" />



## 五、API 网管 API gateway

在微服务架构中，不同的微服务可以有不同的`网络地址`，各个微服务之间通过`互相调用`完成用户请求，客户端可能通过调用N个微服务的接口完成一个用户请求。

> 比如：用户购买一个产品，它可能包含商品基本信息、价格信息、评论信息、折扣信息、库存信息等等，而这些信息获取则来源于不同的微服务，诸如用户微服务、订单微服务、库存微服务、商品微服务存等等。

![](https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEP5Lva2kEGIOyFmIpHcFyDlibNSkgiaEecTUmr6MAughSdqfqj58oB8Eg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

用户要完成一个请求，竟然要调用这么多微服务，这无疑增加了客户端的复杂性。为了解决这个问题，可以在客户端与微服务中间加个中间人，即`API网关`。

<img src="https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEMj1RKicKH3icmic6Ld7l3tjdVsvSbEMtUdHoOBlCibibI30xFs7KxOlAIDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1"  style="zoom:80%;" />

API网关是一个服务器，是系统对外的`唯一入口`。API网关封装了系统内部架构，为每个客户端提供一个定制的API。API网关方式的核心要点是：所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。

事实上，网关的功能非常强大，它通常被用于以下的场景：

> - 反向代理
> - 鉴权
> - 熔断
> - 日志监控
> - ......

常用的网关技术组件有`Zuul`、`Gateway`等。zuul官方现在也停止更新和维护了，现在使用大家一般都是用的GeteWay，下面大概介绍一下GateWay：

### 5.1 GateWay



## 六、服务配置 Cloud configuration

在单体应用中，多多少少都存在配置文件，如Spring应用中的`application.xml`、`log4j.propertie`s等文件，可以方便地进行配置。但是，在微服务架构体系中，由于微服务众多，服务之间又有互相调用关系，这个微服务怎么知道被调用微服务的地址呢，万一被调用微服务地址变了咋办？我们如何方便地进行修改，且实时自动刷新，而不至于需要重启应用。也就是说，微服务的配置管理需要解决以下几个问题：

* （1）配置集中管理：统一对应用中各个微服务进行管理
* （2）动态配置：统一对应用中各个微服务进行管理
* （3）配置修改自动刷新：当修改配置后，能够支持自动刷新

因此，对于微服务架构而言，一个通用的`分布式配置管理`是必不可少的。

常用的配置组件很多，如`SpringCloud Config`、`Apollo`和`Nacos`，本文以SpringCloud Config简要介绍下。

SpringCloud Config为微服务架构中的微服务提供了集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个`中心化`的外部配置。

SpringCloud Config分为`服务端(Config Server)`和`客户端(Config Client)`两个部分。

服务端是分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供配置信息。

客户端则是通过指定的配置中心来管理应用资源，以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息，配置信息默认采用git服务器存储，这样就有助于对环境进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

<img src="https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbE7PKKoRKELNBzjYLyS6PjoQzqXPI7Lg6L4JQn7bOnGrCgUbR6IuxhoQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" style="zoom:80%;" />



## 七、服务熔断与降级 Circuit breakers

所有的系统，特别是分布式系统，都会遇到故障。因此，如何构建应用程序来应对这种故障，是每个软件开发人员必须要掌握的。

采用的策略大多是在远程服务发生错误或表现不佳时避免上一级调用崩溃，使得快速失败，而不是将这种性能不佳影响扩散至整个系统。

为了解决这些故障，常用的技术手段有客户端`负载均衡`、`服务熔断`、`服务降级`和`服务限流`。

<img src="https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbEllrLQzOjoiabreh7cKV2hpGppHiccbicibJOJ2Gb6Vt6NBFkftQvRiaKrKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" style="zoom:80%;" />

（1）服务熔断

服务熔断模仿的是电路中的`断路器`模式，如果调用时间过长，断路器将会介入并中断调用。如果对某一个远程资源的调用失败次数足够多，那么断路器就会采取`跳闸`，阻止继续调用失败的服务。但是，断路器在规定的时间内，会让少量的请求调用该服务，如果这些调用连续多次成功，断路器就会`自动复位`。

（2）服务降级

服务器繁忙，请稍后重试，不让客户端等待并立即返回一个友好的提示。出现服务降级情况如下：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

（3）服务限流

服务限流通常出现在秒杀或者高并发的场景下，对进来的请求进行排队，一个个进行。

使用这几种策略常用的技术组件有：`Hystrix` 和 `Sentinel`。

## 八、服务链路追踪 Tracing

微服务架构是将单体软件分解为更小、更易于管理的部分，这些部分可以独立构建和部署。

![](https://mmbiz.qpic.cn/mmbiz_png/2vOI7k3wdyAr4oWCgw3sPJn9JHniaTnbETttpxfjN33l9ItPx1n0acyg3zQ5sXjmZ01FmljsJnf9lB0dfYibU4CA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

然而，这种灵活性是要付出代价的，微服务的本质是分布式的，它意味着必须在多个服务、物理机器和不同的数据存储之间跟踪一个或多个事务，当我们的程序出现问题的时候，要定位问题出现的位置会让人抓狂。

分布式调用链其实就是将一次分布式请求还原成调用链路。显式的在后端查看一次分布式请求的调用情况，比如各个节点上的`耗时`、`请求`具体打到了哪台机器上、每个服务节点的`请求状态`等等。

`Spring Cloud Sleuth`提供了一套完整的服务跟踪的解决方案，在分布式系统中提供追踪解决方案并且兼容支持了`zipkin`，仅仅需要下载zipkin的jar包即可。

## 参考文献

* 【1】[Java3y.微服务是什么？](https://mp.weixin.qq.com/s/vWhxHGf0AO9froY-urIi_g)
* 【2】[初心myp.单体应用与微服务的比较](https://www.jianshu.com/p/8d840ace19b6)

