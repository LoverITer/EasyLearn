## Spring Cloud — Consul服务注册与发现



## 一、Consul简介

**是什么**

![](https://image.easyblog.top/image-20210909214605701.png)



**能干嘛**

* 服务发现：提供HTTP和DNS两种发现方式
* 健康监测：支持多种协议，HTTP、TCP、Docker、Shell脚本定制化
* KV存储：key , Value的存储方式（类似Redis）
* 多数据中心：Consul支持多数据中心
* 可视化Web界面



## 二、yum 安装并启动 Consul

**下载安装 Consul**

Consul官网下载地址：https://www.consul.io/downloads

Consul 中文文档：https://www.springcloud.cc/spring-cloud-consul.html

选择合适的版本喜进行下载安装，本教程选择使用Linux 发行版 CentOS 版本的，参照官网给的提示进行下载安装操作。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909220205.png)





**成功安装**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909221027.png)



**consul --version 查看版本 Consul 版本号**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909222013.png)



**启动Consul**

```shell
consul agent -dev    # 后台管理不能访问，没启动UI
consul agent -dev -ui -node=consul-dev-14 -client=0.0.0.0 # 0.0.0.0表示不绑定客户端IP地址，不然只能使用特定的IP访问
```

**常用参数**

```tex
-http-port 默认是8500
-client：客户端模式，http dns，默认127.0.0.1，回环令牌网址
-data-dir：状态数据存储文件夹，所有的节点都需要。文件夹位置需要不收consul节点重启影响，必须能够使用操作系统文件锁，unix-based系统下，文件夹文件权限为0600，注意做好账户权限控制，
-dev：开发模式，去掉所有持久化选项，内存服务器模式。
-ui：内置web ui界面。
-bind：绑定的内部通讯地址，默认0.0.0.0，即，所有的本地地址，会将第一个可用的ip地址散播到集群中，如果有多个可用的ipv4，则consul启动报错。[::]ipv6，TCP UDP协议，相同的端口。防火墙配置。
-bootstrap：启动模式，此模式下，节点可以选举自己为leader，一个数据中心只能有一个此模式启动的节点。机群启动后，新启动的节点不建议使用这种模式。
-bootstrap-expect：设定一个数据中心需要的服务节点数，可以不设置，设置的数字必须和实际的服务节点数匹配。consul会等待直到数据中心下的服务节点满足设定才会启动集群服务。初始化leader选举，不能和bootstrap混用。必须配合-server配置。
```



启动成功之后，在浏览量其中访问地址：http://47.99.161.205:8500/ui/

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909223851.png)



**关闭Consul命令**

```shell
consul leave
```



**重载Consul命令**

```shell
consul reload
```



## 三、Consul在微服务项目中使用

**新建模块 — cloud-providerconsul-payment8006**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909230054.png)



**POM**

在pom文件中引入Consul依赖（服务提供者和消费者都需要引入）

```xml
<!-- spring-cloud-starter-zookeeper-discovery -->                 
<dependency>                                                      
    <groupId>org.springframework.cloud</groupId>                  
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>                                                     
```



**application.yml 配置文件**

服务提供者，在application.properties文件配置Consul

```yml
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  cloud:
    consul:
      host: localhost
      port: 8500  #consul服务访问的端口，默认端口就是8500
      discovery:
        service-name: ${spring.application.name}
        instance-id: consul-provider-payment-8500  #设置实例id，默认是项目的名称
        prefer-ip-address: true   #显示客户端的ip地址
```





**编写启动类**

```java
package top.easyblog.prometheus;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * @author frank.huang
 * @since 2021/9/9 23:12
 */
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentConsulApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentConsulApplication.class,args);
    }
    
}
```





**服务提供者**

```java
package top.easyblog.prometheus.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * @author frank.huang
 * @since 2021/9/9 23:15
 */
@RestController
public class PaymentConsulController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/consul")
    public String paymentzk(){
        return "springcloud with consul:"+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}
```



**验证结果**

启动Service provider，刷新 Consul  http://127.0.0.1:8500/ui/dc1/services  如下图所示，上面创建的微服务Provider模块：consul-provider-payment 已经成功注册到 Consul ！

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210909233301.png)