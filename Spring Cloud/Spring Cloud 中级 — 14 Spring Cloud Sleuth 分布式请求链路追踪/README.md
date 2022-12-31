## Spring Cloud 中级 — Spring Cloud Sleuth 分布式请求链路追踪



## 一、Spring Cloud Sleuth 简介

随着业务的发展，我们的系统规模也会变得越来越大，各微服务间的调用关系也变得越来越错综复杂。这时候对于每个请求全链路调用的跟踪就变得越来越重要，通过实现对请求调用的跟踪可以帮助我们快速的发现错误根源以及监控分析每条请求链路上的性能瓶颈等好处。
针对上面所述的分布式服务跟踪问题，Spring Cloud Sleuth提供了一套完整的解决方案。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002112538.png)

### 1.1 Spring Cloud Sleuth 是什么？

> Spring Cloud Sleuth provides Spring Boot auto-configuration for distributed tracing.
>
> Sleuth configures everything you need to get started. This includes where trace data (spans) are reported to, how many traces to keep (sampling), if remote fields (baggage) are sent, and which libraries are traced.

这是 [GitHub](https://github.com/spring-cloud/spring-cloud-sleuth) 上Spring Cloud Sleuth（以下简称Sleuth）官方对Sleuth的介绍，一句话概括就是：Sleuth提供了**一套完整的服务跟踪的解决方案**。同时，在分布式系统中不仅提供追踪解决方案而且还兼容支持了zipkin。下图所示是基于 Sleuth 和 zipkin 搭建的可视化分布式系统链路追踪解决方案。

<img src="https://image.easyblog.top/image-20211002111102141.png" style="zoom:50%;" />

### 1.2 Sleuth 跟踪原理

如下所示表示一条完整的调用链路，一条链路通过Trace Id 唯一标识，Span表示发起的请求信息，各个Span通过parent Id关联起来。

<img src="https://image.easyblog.top/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3NwcmluZy1jbG91ZC9zcHJpbmctY2xvdWQtc2xldXRoL21hc3Rlci9kb2NzL3NyYy9tYWluL2FzY2lpZG9jL2ltYWdlcy90cmFjZS1pZC5wbmc" style="width:80%;" />

**跟踪原理**
分布式系统中的服务跟踪在理论上主要包括下面两个关键点：

1、为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪框架为该请求创建一个唯一的跟踪标识，同时在分布式系统内部流转的时候，框架始终保持传递该唯一标识，直到返回给请求方为止，这个唯一标识就是前文中提到的Trace ID。通过Trace ID的记录，我们就能将所有请求过程日志关联起来。
2、为了统计各处理单元的时间延迟，当请求达到各个服务组件时，或是处理逻辑到达某个状态时，也通过一个唯一标识来标记它的开始、具体过程以及结束，该标识就是我们前文中提到的Span ID，对于每个Span来说，它必须有开始和结束两个节点，通过记录开始Span和结束Span的时间戳，就能统计出该Span的时间延迟，除了时间戳记录之外，它还可以包含一些其他元数据，比如：事件名称、请求信息等。

下图显示了跨度的父子关系：

![](https://image.easyblog.top/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3NwcmluZy1jbG91ZC9zcHJpbmctY2xvdWQtc2xldXRoL21hc3Rlci9kb2NzL3NyYy9tYWluL2FzY2lpZG9jL2ltYWdlcy9wYXJlbnRzLnBuZw)





Spring Cloud Sleuth借用了Google Dapper的术语。

* `Span`：工作的基本单位。例如，发送RPC是一个新的跨度，就像发送响应到RPC一样。Span是由一个唯一的64位ID来标识的，而另一个64位ID用于跟踪。span还具有其他数据，如描述、时间戳事件、键值标注(标记)、导致它们的span的ID和进程ID(通常是IP地址)。

可以启动和停止跨度，并跟踪其时间信息。 创建跨度后，必须在将来的某个时刻停止它。

启动跟踪的初始范围称为根跨度。 该范围的ID值等于跟踪ID。

* `Trace`：一组span形成树状结构。 例如，如果运行分布式大数据存储，则可能由PUT请求形成跟踪。



## 二、搭建基于Sleuth+zipkin的可视化链路监控

### 2.1 安装并启动zipkin

#### 1、下载zipkin并启动zipkin

下载地址：https://repo1.maven.org/maven2/io/zipkin/zipkin-server/

**（1）选择版本**

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002114828.png" style="width:80%;" />

**（2）选择exec版本的jar包**

在win平台上运行的选择exec后缀的jar包，linux平台选择不含有任何后缀名的jar包

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002115153.png" style="width:80%;" />

#### 2、启动zipkin

**（1）在jar包的安装目录执行java -jar命令启动zipkin**

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002115559.png" style="width:80%;" />

执行命令之后看到如上截图所示zipkin在9411端口监听那就表示启动成功，此时我们在浏览器访问 http://127.0.0.1:9411/zipkin 不出意外就可以看到如下监控界面：

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002115903.png" style="width:90%;" />



### 2.2 改造服务Provider和 Consumer

在已有的服务提供者 [cloud-payment-service]() 和 服务消费者 [cloud-payment-consumer]() 项目中作如下修改。

#### 1、分别在在Provider 和 Consumer 项目的pom.xml文件中新增zipkin 和 sleuth 的依赖

```xml
<!--包含了sleuth+zipkin-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```



#### 2、分别在在Provider 和 Consumer 项目的application.yml文件中新增配置

```yml
spring:
  zipkin:
    base-url: http://localhost:9411  # zipkin的监听端口
  sleuth:
    sampler:
      probability: 1  #采样率 ，通常介于 0~1
```



### 2.3 启动测试

在本地8001端口和80端口分别启动服务提供者 [cloud-payment-service]() 和 服务消费者 [cloud-payment-consumer]() ，并在消费者放进行访问操作：http://127.0.0.1/payment?id=3，浏览器显示正常

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002124655.png)

此时我们去到 **服务消费者 [cloud-payment-consumer]()** 的控制台看下信息：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002131800.png)

再去到 **服务提供者 [cloud-payment-service]()** 的控制台查看信息：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002131810.png)

从上面的控制台输出内容中，我们看到形如：`INFO [cloud-payment-service,2945747a39bf3e0a,297d2b308e113e09,true]` 的日志

这些元素正式实现分布式服务跟踪的重要组成部分，每个值的含义如下描述：

* 第一个值：它记录应用的名称，也就是 application.yml 配置的服务名。

* 第二个值：2945747a39bf3e0a，是 Sleuth 生成的一个 ID，称为 `Trace ID`，它用来标识一条请求链路。一条请求链路包含一个 Trace ID 和多个 Span ID。

* 第三个值：297d2b308e113e09，是 Sleuth 生成的另一个 ID，称为 Span ID，它标识一个基本的工作单元，比如发送一个 HTTP 请求。

* 第四个值：`false/true`，表示是否要将该信息输出到 Zipkin 等服务中来收集和展示。

在一次请求调用过程中，会保持并传递同一个 Trace ID，从而将整个分布于不同微服务进程中的请求跟踪信息串联起来。

Zipkin 是 Twitter 的一个开源项目，它可以用于收集各个服务上请求链路的跟踪数据，并通过它提供的 REST API 接口来辅助查询跟踪数据以实现对分布式系统的监控程序，从而及时发现系统中出现的延迟升高问题，并找出系统性能瓶颈的根源。下图所示是zipkin的监控Dashboard：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002132529.png)

查看依赖关系：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211002134434.png)