## Spring Cloud 中级 — Spring Cloud Config 分布式配中心



## 一、Spring Cloud Config 简介

### 1.1 分布式微服务带来的痛点

配置文件是我们再熟悉不过的了，尤其是 Spring Boot 项目，除了引入相应的 maven 包之外，剩下的工作就是完善配置文件了，例如 mysql、redis 、security 相关的配置。除了项目运行的基础配置之外，还有一些配置是与我们业务有关系的，比如说七牛存储、短信相关、邮件相关，或者一些业务上的开关。

对于一些简单的项目来说，我们一般都是直接把相关配置放在单独的配置文件中，以 properties 或者 yml 的格式出现，更省事儿的方式是直接放到 `application.properties` 或` application.yml` 中。但是这样的方式有个明显的问题，那就是，当修改了配置之后，必须重启服务，否则配置无法生效。此外每个微服务都需要必要配置信息才能运行，当有大量微服务系统的之后，光维护这些配置文件就是一件令人头疼的事情，所以一套集中式、动态的配置管理设施必不可少。

Spring Cloud提供了Config Server来解决这个问题，此外目前有一些用的比较多的开源的配置中心，比如携程的 Apollo、蚂蚁金服的 disconf 等，对比 Spring Cloud Config，这些配置中心功能更加强大。有兴趣的可以拿来试一试。

**接下来**，我们开始在 Spring Boot 项目中集成 Spring Cloud Config，并以 github 作为配置存储。除了 git 外，还可以用数据库、svn、本地文件等作为存储。主要从以下三块来说一下 Config 的使用。

1. **基础版的配置中心（不集成 Eureka）**;

2. **结合 Eureka 版的配置中心**;

3. **实现配置的自动刷新**；



### 1.2 Config 是什么？

![](https://image.easyblog.top/image-20211001085505597.png)

Spring Cloud Config（以下简称Config） 是一个解决分布式系统的配置管理方案。它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用。如下是 Config 配置中心提供的核心功能：

- 提供服务端和客户端支持
- 集中管理各环境的配置文件
- 配置文件修改之后，可以快速的生效
- 可以进行版本管理
- 支持大的并发查询
- 支持各种语言



## 二、构建Spring Cloud Config配置中心

构建一个 spring cloud config 配置中心按照如下方式进行：

### 1、创建一个通用的 Spring Boot 项目

### 2、在 pom.xml 文件中添加如下依赖

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 3、在 Boot 启动类上添加注解 @EnableConfigServer

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableDiscoveryClient
@EnableConfigServer   //开启Config Server
public class ConfigCenterApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterApplication.class,args);
    }
    
}
```



### 4、在 application.properties 中配置 git 仓库信息

在 application.properties 中配置一下 git 仓库信息，此处我们使用 GitHub( 也可以使用码云 gitee ) ， 首先在我的 Github 上创建一个名为 `spring-cloud-config-center-server` 的项目，创建之后，再做如下配置：

```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center-center-server
  cloud:
    config:
      server:
        git:
          uri:  https://github.com/myspring/sprin #填写你自己的github路径
          username: myusername   #填写你自己的github用户名
          password: 12345678     #填写你自己的github密码
          search-paths:
            - springcloud-config-center-server
      label: master
```

至此我们的配置中心服务端就创建好了。



## 三、构建Spring Cloud config配置中心仓库

接下来我们需要在 github 上设置好配置中心，首先在本地创建一个文件夹叫 config-center，然后在 config-center中创建四个配置文件，如下：

```txt
application-dev.yml
application-test.yml
application-stg.yml
application.yml
```

在四个文件里面分别写上要测试的内容：

```yml
server:
   port: 8001
url:
  http://dev.wkcto.com
---
server:
   port: 8005
url: 
  http://test.wkcto.com
---  
server:
   port: 80
url:
  http://stg.wkcto.com
---
server:
   port: 80
url:
  http://www.wkcto.com
```

完成之后，我们在git客户端执行git命令将配置信息推送到远程github上

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211001095531.png)

此时启动我们的配置中心，通过 `/{application}/{profile}/{label}` 就能访问到我们的配置文件了

其中：

* `{application}`：表示配置文件的名字，对应的配置文件即application
* `{profile}`：表示环境，有dev、test、stg及默认
* `{label}`：表示一组“版本化”的配置文件，一般就是分支名称

通过浏览器上访问 http://localhost:3344/application/dev/master  ，返回如下JSON 格式的数据：

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211001102809.png" style="width:80%;" />

`name` 表示配置文件名 application 部分，`profiles` 表示环境，`label` 表示分支，`version` 表示 GitHub 上提交时产生的版本号，同时当我们访问成功后，在控制台会打印了相关的日志信息；当访问成功后配置中心会通过 git clone 命令将远程配置文件在本地也保存一份，以确保在 git 仓库故障时我们的应用还可以继续正常使用



## 四、构建Spring cloud config配置中心客户端

前面已经搭建好了配置中心的服务端，接下来我们来看看如何在客户端应用中使用。构建一个 spring cloud config 配置中心客户端按照如下方式进行：

### 1、创建一个通用的 Spring Boot 项目

### 2、在 pom.xml 文件中添加如下依赖

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 3、创建 bootstrap.yml 文件，用于获取配置信息

![](https://image.easyblog.top/image-20211001104952297.png)

```yml
server: 
   port: 3355
spring:
   application:
       name: application
    cloud:
       config:
          name: application     #配置文件名称 
          profile: dev          #环境
          label: master         #分支名
          uri: http://127.0.0.1:3344/  #配置中心地址
```

其中：

* `name` ，对应配置文件中的 application 部分
* `profile` ，对应了 profile 部分 
* `label`， 对应了 label 部分
* `uri` ，表示配置中心的地址。

### 4、创建一个 Controller 进行测试

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigController {

    @Value("${url}")
    private String url;

    @Value("${server.port}")
    private Integer port;

    @Autowired
    private Environment env;

    @RequestMapping("/cloud/url")
    public String url () {
        return this.url+":"+port;
    }

    @RequestMapping("/cloud/url2")
    public String url2 () {
        return env.getProperty("url")+":"+env.getProperty("server.port");
    }
}
```

启动服务之后访问浏览器 http://127.0.0.1:8001/url 以及 http://127.0.0.1:8001/url2 均得到如下界面：

![](https://image.easyblog.top/4.gif)



通过上面的测试发现Spring Cloud Config 的功能确实很强大，实现了完全从远程获取配置并将配置应用到客户端本地的功能。





## 五、Spring Cloud Config 动态刷新配置

之所以引入这个配置热刷新的机制，主要目的是为了减少运维的工作量。如果没有这个配置热刷新的功能，每一个服务的配置变更后，运维人员都要登录对应的服务器来手动停止并重新启动服务，不仅耗费人力，也降低了服务稳定性。通过这种热刷新机制，通过调用配置中心的端点就可以实现对所有服务配置的自动刷新，在配置有较大修改的时候可以极大省去运维工作。

### 5.1 方案一：手动刷新

#### 1、在Config Server 的pom.xml文件中新增配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



#### 2、在Config server 项目的application.yml中增加配置

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



#### 3、在Config client 项目的Controller上增加`@RefreshScope` 注解

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope   //让其可以有自动刷新的能力
public class ConfigController {

    @Value("${url}")
    private String url;

    @Value("${server.port}")
    private Integer port;

    @Autowired
    private Environment env;

    @RequestMapping("/cloud/url")
    public String url () {
        return this.url+":"+port;
    }

    @RequestMapping("/cloud/url2")
    public String url2 () {
        return env.getProperty("url")+":"+env.getProperty("server.port");
    }
}
```



#### 4、向Config client 手动发送刷新POST请求

在修改配置文件后需要同时请求以下地址告知Config client刷新配置。注意这里必须是POST请求！

```shell
curl -X POST "http://localhost:8001/actuator/refresh"
```



### 5.2 方案二：基于Webhook触发的自动刷新

此方法基于 5.1 的原理，只是把手动请求url的一步操作通过我们在github上配置提交自动触发操作之后，github会自动在我们提交之后请求url自动刷新，一定程度上进一步解放了运维的难度。接下来着重介绍一下如何配置webhook。



### 5.3 方案二：基于Spring Cloud Bus的全自动动态刷新

参考 [Spring Cloud 中级— Spring Cloud Bus 消息总线]()



## 六、Spring Cloud Config的安全保护

生产环境中我们的配置中心肯定是不能随随便便被人访问的，我们可以加上适当的保护机制，由于微服务是构建在 Spring Boot 之上，所以整合 Spring Security是最方便的方式。

### 1、在 Config server 项目添加依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2、在  Config server 项目的 application.yml 中增加配置用户名密码

```yml
spring: 
  security:
    user:
      name: admin
      password: 123456
```

### 3、在 Config client 项目的 bootstrap.yml 中增加配置用户名和密码

```yml
spring:
    cloud:
       config:
          username: admin
          password: 123456
```

