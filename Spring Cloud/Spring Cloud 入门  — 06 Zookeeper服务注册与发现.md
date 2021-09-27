## Spring Cloud —  Zookeeper 服务注册于发现.md





## 一、安装并启动Zookeeper

TODO：zk学习和安装



## 二、新建 Service Provider

新建模块cloud-provider-payment8004

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210908221120.png)



**pom**

引入zk服务注册有关依赖

```xml
 <!-- spring-cloud-starter-zookeeper-discovery -->                    
 <dependency>                                                         
     <groupId>org.springframework.cloud</groupId>                     
     <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
 </dependency>                                                        
```



application.yml 配置文件

```yml
server:
  port: 8004

spring:
  application:
    name: cloud-provider-payment:8004
  cloud:
    zookeeper:
      connect-string: 47.99.161.205:2181   #zk服务地址
```





启动类上标注 `@EnableDiscoveryClient`  开启服务注册和发现功能

```java
package top.easyblog.prometheus;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @author frank.huang
 * @since 2021/9/8 22:17
 */
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class,args);
    }
    
}
```



**编写  controller**

```java
package top.easyblog.prometheus.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * @author frank.huang
 * @since 2021/9/9 21:42
 */
@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/zk")
    public String paymentzk(){
        return "springcloud with zookeeper:"+serverPort+"\t"+ UUID.randomUUID().toString();
    }

}
```



（2）测试服务在zk上创建的节点类型（ 服务节点是临时节点还是持久节点）

关闭server服务，断开心跳，发现注册在zk上的服务服务是**临时节点**



## 三、新建 Service Consumer

新建订单服务消费者：cloud-consumerzk-order80  ，通过zk服务发现的方式消费 Service Provider 提供的接口



![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909212744.png)



**POM**

同样除了需要引入Spring Cloud相关依赖之外，还需要引入zk服务注册有关依赖

```xml
 <!-- spring-cloud-starter-zookeeper-discovery -->                    
 <dependency>                                                         
     <groupId>org.springframework.cloud</groupId>                     
     <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
 </dependency> 
```



**application.yml 配置**

```yml
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  cloud:
    zookeeper:
      connect-string: 47.99.161.205:2181
```



**主启动类**

 标注 `@EnableDiscoveryClient`  开启服务注册和发现功能

```java
package top.easyblog.promethues;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @author frank.huang
 * @since 2021/9/9 21:29
 */
@SpringBootApplication
@EnableDiscoveryClient
public class OrderZkConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderZkConsumerApplication.class,args);
    }
}
```



**配置 RestTemplate**

使用 @LoadBalanced 注解赋予RestTemplate负载均衡的能力

```java
package top.easyblog.promethues.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @author frank.huang
 * @since 2021/9/5 0:57
 */
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```



**编写  controller**

```java
package top.easyblog.promethues.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author frank.huang
 * @since 2021/9/9 21:36
 */
@RestController
public class OrderConsumer {


    public static final String BASE_URL = "http://cloud-provider-payment";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/zk")
    public String payment (){
        String result = restTemplate.getForObject(BASE_URL+"/payment/zk",String.class);
        return result;
    }
    
}
```



**验证和测试**

浏览器中访问地址：http://localhost/consumer/payment/zk  可以出现

