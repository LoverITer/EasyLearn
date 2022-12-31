## Spring Cloud 高级 — Spring Cloud Alibaba Nacos 服务注册和配置中心



## 一、Spring Cloud Alibaba Nacos 简介

### 1.1 Spring Cloud Alibaba Nacos 是什么？

Spring Cloud Alibaba Nacos（以下简称 Nacos） 是阿里巴巴推出的一个开源项目，这是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。


#### Nacos提供四大功能

- **服务发现和服务健康检查**

  Nacos使服务更容易注册自己并通过DNS或HTTP接口发现其他服务。Nacos还提供服务的实时健康检查，以防止向不健康的主机或服务实例发送请求。

- **动态配置管理**

  动态配置服务允许您在所有环境中以集中和动态的方式管理所有服务的配置。Nacos消除了在更新配置时重新部署应用程序和服务的需要，这使配置更改更加高效和灵活。

- **动态DNS服务**

  动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务。动态DNS服务还能让您更容易地实现以 DNS 协议为基础的服务发现，以帮助您消除耦合到厂商私有服务发现 API 上的风险。

- **服务和元数据管理**

  Nacos提供易于使用的服务仪表板，可帮助您管理服务元数据，配置，kubernetes DNS，服务运行状况和指标统计。

### 1.2 为什么是 Nacos？

**Nacos和其他注册中心的对比**

![](https://image.easyblog.top/image-20211010174641442.png)

相对于 Spring Cloud Eureka 来说，Nacos 更强大。

**Nacos = Spring Cloud Eureka + Spring Cloud Config**

Nacos 可以与 Spring、Spring Boot、Spring Cloud 集成，并能代替 Spring Cloud Eureka, Spring Cloud Config。

* 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-config 实现配置的动态变更。

* 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现。

## 二、Nacos 服务的安装和运行

参考 [Nacos 官网快速开始教程](https://nacos.io/zh-cn/docs/quick-start.html) 

在[Nacos release node](https://github.com/alibaba/nacos/releases) 下载并解压之后可以看到如下目录结构

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211010132009.png)

然后如果是Liunx/MAC/Unix平台则在`bin`目录下执行如下命令启动Nacos：

```shell
./startup.sh -m standalone
```

如果是Windows平台则在`bin`目录下执行如下命令启动Nacos：

```shell
startup.cmd -m standalone
```

* 启动参数中 `standalone` 代表让 Nacos 以单机非集群模式运行



启动之后如果看到控制台窗口输出类似如下日志代表启动成功：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211010131937.png)



还可以访问：http://127.0.0.1:8848/nacos 查看Nacos的管理界面（需要登陆，默认的用户名和密码都是 `nacos`）：

**登陆Nacos**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211010133306.png)

**Nacos管理界面**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211010133322.png)



## 三、Nacos 服务注册与服务发现配置和使用



### 3.1 Nacos 服务注册

新建一个多么模块 Spring Boot 项目（可以参考我的另一篇Spring Cloud系列教程：[Spring Cloud —  微服务多模块工程架构搭建]()），然后新建模块 `cloudalibaba-provider-payment9001` 

#### 1、在pom.xml 中引入依赖

首先需要在父工程的pom.xml文件中引入 spring-cloud-alibaba-dependencies 有关依赖的声明：

```xml
 <!--spring cloud alibaba-->                                   
 <dependency>                                                  
     <groupId>com.alibaba.cloud</groupId>                      
     <artifactId>spring-cloud-alibaba-dependencies</artifactId>
     <version>${spring.cloud.alibaba.version}</version>        
     <type>pom</type>                                          
     <scope>import</scope>                                     
 </dependency>                                                 
```

之后在新建的模块中引入如下spring-cloud-alibaba-nacos-discovery 的依赖：

```xml
<dependency>                                                             
    <groupId>com.alibaba.cloud</groupId>                                 
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>                                                            
```

#### 2、在服务Provider的配置文件 `application.yml` 中做如下配置

```yml
server:
  port: 9001

spring:
  application:
    name: cloud-nacos-payment-provider-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 #配置nacos的主机地址
management:
  endpoints:
    web:
      exposure:
        include: '*'
```



#### 3、在服务Provider的启动类上使用 `@EnableDiscoveryClient` 开启服务注册

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient  //开启服务注册
public class PaymentProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentProviderApplication.class, args);
    }

}

```

#### 4、编写controller用于测试

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;


@RestController
@RequestMapping(value = "/test")
public class PaymentController {


    @GetMapping(value = "/payment/{id}")
    public String getPayment(@PathVariable Integer id){
        return String.format("spring cloud with nacos: {id: %d,payment_no: %s}",id,UUID.randomUUID().toString());
    }
    
}
```



完成之后启动服务，此时我们首先访问：http://127.0.0.1:9001/test/payment/10010 ，不出意外可以看到正常响应的页面如下：

![](https://image.easyblog.top/5.gif)

此时去Nacos【服务管理】->【服务列表】会发现我们刚才创建的微服务也已经注册到nacos上了：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211010164525.png)



### 3.2 Nacos 服务发现

完成3.1节的步骤，接下来我们创建服务的消费者，新建模块 `cloudalibaba-consumer-nacos-order80` 

#### 1、在服务Consumer的 pom.xml 文件中引入nacos相关依赖

```xml
<!--Spring Cloud Alibaba-->
<dependency>                                                             
    <groupId>com.alibaba.cloud</groupId>                                 
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>  
<!--待会儿使用Feign做服务调用-->
<dependency>                                               
    <groupId>org.springframework.cloud</groupId>           
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>                                              
```

#### 2、在服务Consumer的 `application.yml` 文件中做如下配置

```yml
server:
  port: 80

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

url:
  payment-service: http://cloud-nacos-payment-provider-service #这里配置服务provider的服务名称
```

#### 3、在服务Consumer的启动类上使用 `@EnableDiscoveryClient` 开启服务发现以及 `@EnableFeignClients` 开启Feign

```java
@SpringBootApplication 
@EnableDiscoveryClient  //开启服务发现
@EnableFeignClients     //开启Feign
public class PaymentConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentConsumerApplication.class,args);
    }

}
```

#### 4、编写controller用于测试

```java
@RestController
@RequestMapping(value = "/consumer")
public class PaymentConsumerController {

    @Autowired
    private PaymentFeignClient paymentFeignClient;

    @GetMapping(value = "/pay/{id}")
    public String getPayment(@PathVariable Integer id){
        return paymentFeignClient.getPaymentById(id);
    }

}
```

`PaymentFeignClient` 如下：

```java
@FeignClient(name = "${url.payment-service}")
public interface PaymentFeignClient {

    @GetMapping(value = "/payment/{id}")
    String getPaymentById(@PathVariable(value= "id") Integer id);

}
```

完成之后启动在本地80端口启动服务消费者，然后在浏览器输入地址：http://127.0.0.1/consumer/pay/1000 ，如果看到如下界面表示基于Nacos搭建的单节点服务注册和服务发现实验成功。

![](https://image.easyblog.top/6.gif)



### 3.3  Nacos 集群和持久化配置

#### 1、Nacos集群搭建原理简介



#### 2、配置Nacos持久化数据库为MySQL



#### 3、



## 四、Nacos 服务配置中心的配置和使用



接上节 [Nacos服务注册于服务发现配置使用]()，本节我们继续来了解学习一下Nacos做分布式的配置中心如何使用/配置。

首先新建配置中心模块 `cloudalibaba-config-nacos-client3377` 

#### 1、在 pom.xml 中引入Naocs相关依赖

```xml
 <!--nacos-config-->                                                      
 <dependency>                                                             
     <groupId>com.alibaba.cloud</groupId>                                 
     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>   
 </dependency>                                                            
 <!--nacos-discovery-->                                                   
 <dependency>                                                             
     <groupId>com.alibaba.cloud</groupId>                                 
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
 </dependency>                                                            
```

#### 2、新建 bootstrap.yml 配置文件

```yml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yml   #配置文件后缀名
```

#### 3、新建application.yml 配置文件

```yml
spring:
  profiles:
    active: dev   #表示开发环境
```

#### 4、新建主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientApplication.class,args);
    }

}
```



#### 5、在nacos中添加配置信息

（1）登陆Nacos管理后台新建配置列表

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211016125209.png)

（2）填写配置信息

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211016125739.png)

Nacos配置列表`dataId`填写规则：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211016124849.png)



（3）发布配置信息

完成配置之后点击下方发布按钮保存并发布配置信息，成功之后会弹框提示。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211016125830.png" style="zoom:60%;" />



#### 6、编写controller用于测试

```java
@Slf4j
@RestController
@RefreshScope   //实现自动更新配置
public class NacosConfigController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }

}
```



编写完成之后，启动项目，访问地址：http://http://127.0.0.1:3377/config/info ，可以看到程序读取到配置了：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211016131432.png)

