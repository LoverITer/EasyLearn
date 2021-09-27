## Spring Cloud — Eureka 服务注册与发现



## 一、服务治理、服务注册与发现介绍

### 1.1 什么是服务治理

Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理

**在传统的RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服服务之间依赖关系，可以实现服务调用、负载均衡、容错等**，实现服务发现与注册。



### 1.2 什么是服务注册与发现

Eureka采用了CS的设计架构，Eureka server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用**Eureka的客户端连接到Eureka Sever并维持心跳连接**。这样系统的维护人员就可以通过Eureka Server 来监控系统中各个微服务是否正常运行。

在服务注册与发现中有一个**注册中心**。当服务（**生产者服务，Service Provider**）启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以别名方式注册到注册中心上。另一方（**消费者服务,Service Consumer**）以该别名去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程润用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何RPC远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址)



### 1.3 Eureka 两大组件介绍

Eureka包含两个组件: **Eureka Server** 和 **Eureka Client** ，下图是Eureka组织架构示意图：

![](https://image.easyblog.top/image-20210905154342509.png)

#### Eureka Server提供服务注册服务

各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点i信息，服务节点的信息可以在界面中直观看到。

#### EurekaClient通过注册中心进行访问

EurekaClient是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳,EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)



## 二、单机 Eureka 服务注册中心安装与配置



#### 2.1 服务注册中心模块的构建和配置

（1）新建模块 `cloud-eureka-server7001`

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905160014.png)

（2）导入Eureka相关依赖包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-learning</artifactId>
        <groupId>top.easyblog.prometheus</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-eureka-server7001</artifactId>

    <dependencies>
        <!-- spring cloud eureka 依赖包 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
</project>
```

（3）Eureka Server相关配置

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost  #eureka服务端的实例名字
  client:
    register-with-eureka: false    #表识不向注册中心注册自己
    fetch-registry: false   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
      service-url:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/    #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
```

（4）编写Eureka注册中心启动类 EurekaServerApplication

特别注意 不仅要使用`@SpringBootApplication` 注解标记它是启动类，还需要使用 `@EnableEurekaServer` 注解标记他是Eureka服务端，与之对应的是 `@EnableEurekaClient` 注解，他表示 EureKa 客户端 ，即服务生产者（Service provider）。

```java
package top.easyblog.promethuns;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * @author frank.huang
 * @since 2021/9/5 16:18
 */
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }
}
```

（5）启动 Eureka Server

启动 EurekaServerApplication ，打开浏览器访问 http://localhost:7001，如果看得如下截图所示页面，那么恭喜你，单机版Eureka服务已经搭建成功！

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905163657.png)



#### 2.2 Service Provider将服务注册到服务注册中心

2.1 小节中我们新建并配置好了一个单机版的 Eureka 服务注册中心，但是还没有 Service Provider 将服务注册到 Eureka 注册中心上，接下来我将本小节我将会使用上一篇教程： [Spring Cloud — 04 微服务多模块工程架构搭建](http://localhost) 中搭建的 Service Provider ：`cloud-provider-payment8001` 为例继续给大家演示如何将自己写的服务注册到 Eureka 注册中心上去。



（1）在原工程中添加 Eureka Client 的依赖包

```xml
<!-- spring-cloud-eureka-client -->                                    
<dependency>                                                           
    <groupId>org.springframework.cloud</groupId>                       
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>                                                          
```

（2）Eureka Client 相关配置

```yml
eureka:
  client:
    register-with-eureka: true #Service Provider需要自己注册到Eureka
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka/    #服务将会按照这个路径注册到eureka server中
```

（3）在原工程启动类上添加 `@EnableEurekaClient`  注解

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905165457.png)

（4）启动服务，验证结果

保持Eureka Server 服务处于运行状态，我们接着启动Eureka Client 服务 `PaymentApplication`，如下截图所示 `cloud-payment-service` 服务已经成功被注册到 Eureka Server  上去。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905173041.png)

另外，在上面截图中我们发现 Eureka 给出了一串红字警告，这是Eureka的自我保护机制，可以在 Service Provider  的配置文件中增加如下配置：

```yml
spring:
   security:
    user:
      password: root
      name: root
```

其实，当我们在点击注册实例 `Status` 栏下方的链接的时候就会发现如下图所示Spring要求我们填写用户名和密码的界面，而这个用户名和密码就是我们上面配置`spring.security.user.name` 和 `spring.security.user.password` 

#### 2.3 Service Consumer在服务注册中心发现并远程调用服务

2.2 小节中我们将我在上一篇教程： [Spring Cloud — 04 微服务多模块工程架构搭建](https://127.0.0.1) 中创建的 Service Provider ：`cloud-provider-payment8001` 已经成功注册到Eureka 服务注册中心。本小节，我们继续以上篇教程中的 Service Consumer：`cloud-consumer-order80` 为基础，继续给大家演示如何消费自己注册到 Eureka 注册中心上的服务。

（1）Eureka先关配置

首先，还是需要对Eureka进行相关的必要配置，包括：【导入spring-cloud-eureka-starter-client 依赖包】-> 【配置Eureka相关参数】-> 【在启动类添加 `@EnableEurekaClient` 注解】，具体方式请参考2.2. 小节相关步骤。

配置完成之后启动服务，在Eureka管理页面刷新，我们将会看到如下图所示页面，消费者也成功被注册到Eureka Server上！

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905180326.png)

（2）测试服务

随后，在postman输入：http://127.0.0.1/consumer/payment?id=8 查看ID为8的支付单详请，如下图所示服务发现和远程服务调用均可正常工作！

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905183328.png)



## 三、集群 Eureka 服务注册中心安装与配置

线上Eureka高可用集群，至少三个节点组成一个集群，推荐部署在不同的服务器上，IP用域名绑定，端口保持一致。本教程使用的三台线上服务器地址分别是：`47.99.161.205`、`101.34.31.122`、`110.42.154.13`

（1）修改本机host文件配置

编辑 C:\Windows\System32\drivers\etc\hosts 文件，添加 127.0.0.1 的本地域名

```tex
127.0.0.1   euraka7001.com
127.0.0.1   euraka7002.com
```



（2）分别在在本地7001端口和7002端口启动两个的Eureka Server 服务

如法炮制第二节搭建单 Eureka 注册中心的方法，这里不再赘述！

有差别的地址地方：一是配置文件这里的 `euraka.instance.hostname` 它是唯一标识 euraka 在集群中身份的值，为了方便之后测试查看结果，需要配置为上面 hosts 文件中的域名地址；二是 `euraka.client.service-url.defaultZone` 的值 ，之前在单机模式中配置的地址是Service Provider 或 Service Consumer 可以找到 Euraka 服务的地址，现在不是了，现在需要配置为 Euraka 集群中其他   Euraka 服务的地址，以达到让所有 Euraka 服务间的相互注册。

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com   #配置Eureka的主机名
  client:
    register-with-eureka: false #不把自己注册到Eureka
    fetch-registry: false   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/     # 集群配置：这里 7001 端口的 eureka 需要关注7002端口的 eureka
```



同理，在本机7002端口再启动一个 Euraka 服务，它的配置参数如下：

```yml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com   #配置Eureka的主机名
  client:
    register-with-eureka: false #不把自己注册到Eureka
    fetch-registry: false   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/  # 集群配置：这里 7002 端口的 eureka 需要关注7001端口的 eureka
```





如下图1所示是分别在本地7001和7002端口启动的两个Euraka 服务：

<center><img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905223020.png" style="zoom:67%;" /><br/>图1 在本地7001和7002端口启动的两个Euraka 服务</center>



<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905223305.png" style="zoom:50%;" />



<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905223305.png" style="zoom:50%;" />

可以看到图上registered-replicas和available-replicas分别有了对方的地址。

（3）配置并启动 Service Provider 和 Service Consumer

Service Provider 也需要少许配置和修改，具体就是 `euraka.client.service-url.defaultZone`  参数的值，现在可以可以Euraka 服务地址了，当然你也可以只配置一个，详细配置如下所示：

Service Provider ：`cloud-provider-payment8001`

```yml
spring:
  application:
    name: cloud-payment-service
eureka:
  client:
    register-with-eureka: true #Service Provider需要自己注册到Eureka
    fetch-registry: true
    service-url:
      defaultZone: http://euraka7001.com:7001/eureka/,http://euraka7002.com:7002/eureka/
```



 Service Consumer：`cloud-consumer-order80`

```yml
spring:
  application:
    name: cloud-consumer-order
eureka:
  client:
    register-with-eureka: true #Service Provider需要自己注册到Eureka
    fetch-registry: true
    service-url:
      defaultZone: http://euraka7001.com:7001/eureka/,http://euraka7002.com:7002/eureka/    #服务将会按照这个路径注册到eureka server中
```



配置完成之后分别启动两个服务，浏览器地址栏访问：http://euraka7001.com:7001 发现两个服务都已经注册上来，再访问地址：http://euraka7002.com:7002 发现另一个Euraka注册中心同样也有两个服务注册上来。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905225831.png" style="width:90%;" />



<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905225906.png" style="width:90%;" />



此时我们在postaman中访问Service Provider 查询接口 ，服务可以正常工作：

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210905183328.png" style="width:90%;" />



## 四、Euraka 服务发现 Discovery

* DiscoveryClient 类  
* @EnableDiscovery注解



## 五、Euraka 自我保护机制

![](https://image.easyblog.top/image-20210908214022456.png)

Euraka 自我保护机制：某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存

