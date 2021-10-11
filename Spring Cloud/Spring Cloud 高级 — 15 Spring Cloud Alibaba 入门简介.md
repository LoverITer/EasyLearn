## Spring Cloud 高级 — Spring Cloud Alibaba 入门简介



Spring Cloud Alibaba 是阿里巴巴提供的微服务开发一站式解决方案，是阿里巴巴开源中间件与 Spring Cloud 体系的融合。

## 一、从 Spring Cloud 说起

提起微服务，不得不提 Spring Cloud 全家桶系列，SpringCloud 是若干个框架的集合，包括 spring-cloud-config、spring-cloud-bus 等近 20 个子项目，提供了服务治理、服务网关、智能路由、负载均衡、断路器、监控跟踪、分布式消息队列、配置管理等领域的解决方案。

Spring Cloud 通过 Spring Boot 风格的封装，屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、容易部署的分布式系统开发工具包。

一般来说，Spring Cloud 包含以下组件，主要以 Netflix 开源为主：

![](https://image.easyblog.top/v2-2d78afcc4e7019b9788478345059a9a3_720w.jpg)

## 二、Spring Cloud Alibaba

同 Spring Cloud 一样，Spring Cloud Alibaba 也是一套微服务解决方案，包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

作为 Spring Cloud 体系下的新实现，Spring Cloud Alibaba 跟官方的组件或其它的第三方实现如 Netflix, Consul，Zookeeper 等对比，具备了更多的功能:

![](https://image.easyblog.top/v2-3a132c503bb7cde19314acc8a4866c4b_720w.jpg)

## 三、Spring Cloud Alibaba 包含的组件

下图所示是 Spring Cloud Alibaba 系列组件，其中包含了阿里开源组件，阿里云商业化组件，以及集成Spring Cloud 组件。

![](https://image.easyblog.top/v2-46c0b9e0d41c441d222390c79a4cd53b_720w.jpg)

* **Nacos**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

* **Sentinel**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

* **RocketMQ**：开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
* **Dubbo**：这个就不用多说了，在国内应用非常广泛的一款高性能 Java RPC 框架。
* **Seata**：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
* **Arthas**：开源的Java动态追踪工具，基于字节码增强技术，功能非常强大。

作为一家商业公司，阿里巴巴推出 Spring Cloud Alibaba，很大程度上市希望通过抢占开发者生态，来帮助推广自家的云产品。所以在开源社区，夹带了不少私货：

* **Alibaba Cloud ACM**：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
* **Alibaba Cloud OSS**：阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的云存储服务。
* **Alibaba Cloud SchedulerX**：阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准的定时（基于 Cron 表达式）任务调度服务。

## 四、Spring Cloud Alibaba 功能

作为微服务解决方案， Spring Cloud Alibaba是如何支持微服务治理的各个功能的呢？

### （1）服务注册与发现

Spring Cloud Alibaba 基于 Nacos 提供 spring-cloud-alibaba-starter-nacos-discovery & spring-cloud-alibaba-starter-nacos-config 实现了服务注册 & 配置管理功能。依靠 @EnableDiscoveryClient 进行服务的注册，兼容 RestTemplate & OpenFeign 的客户端进行服务调用。

适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

### （2）服务限流降级

作为稳定性的核心要素之一，服务限流和降级是微服务领域特别重要的一环，Spring Cloud Alibaba 基于 Sentinel，对 Spring 体系内基本所有的客户端，网关进行了适配，

默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入。

Sentinel应用比较简单，只需引入 starter，即可生效，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

### （3）微服务消息驱动

支持为微服务应用构建消息驱动能力，基于 Spring Cloud Stream 提供 Binder 的新实现: Spring Cloud Stream RocketMQ Binder，

也新增了 Spring Cloud Bus 消息总线的新实现 Spring Cloud Bus RocketMQ。

### （4）分布式事务

使用 Seata 解决微服务场景下面临的分布式事务问题。

使用 @GlobalTransactional 注解，在微服务中传递事务上下文，可以对业务零侵入地解决分布式事务问题。

### （5）分布式任务调度

提供秒级，精准，高可靠，高可用的定时（基于Cron表达式）任务调度服务。同时停工分布式的任务执行模型，如网络任务。





