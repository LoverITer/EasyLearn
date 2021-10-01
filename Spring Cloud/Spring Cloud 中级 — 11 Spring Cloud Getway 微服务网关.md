## Spring Cloud 中级 — Spring Cloud Getway 微服务网关



## 一、Spring Cloud Getway简介

### 1.1 Spring Cloud Getway 是什么？

![](https://image.easyblog.top/image-20210925175419850.png)

首先看看官网是如何定义的：

![](https://image.easyblog.top/image-20210925175543447.png)



红框英文大意：

> 本项目提供了一个构建在 Spring 生态系统之上的 API 网关，包括：Spring 5、Spring Boot 2 和 Project Reactor。 Spring Cloud Gateway 旨在提供一种简单而有效的方式来路由到 API 并为它们提供交叉关注点，例如：安全性、监控/指标和弹性。 

实际上，SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。而为了提升网关的性能 **Spring Cloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty**。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

### 1.2、Spring Cloud Getway的特征

SpringCloud官方，对SpringCloud Gateway 特征介绍如下：

（1）基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0

（2）集成 Hystrix 断路器

（3）集成 Spring Cloud DiscoveryClient

（4）Predicates 和 Filters 作用于特定路由，易于编写的 Predicates 和 Filters

（5）具备一些网关的高级功能：动态路由、限流、路径重写

从以上的特征来说，和Zuul的特征差别不大。SpringCloud Gateway和Zuul主要的区别，还是在底层的通信框架上。

简单说明一下上文中的三个术语（Spring Cloud Getway三大核心概念）：

**（1）Filter（过滤器）**：

和Zuul的过滤器在概念上类似，可以使用它拦截和修改请求，并且对上游的响应，进行二次处理。过滤器为org.springframework.cloud.gateway.filter.GatewayFilter类的实例。

**（2）Route（路由）**：

网关配置的基本组成模块，和Zuul的路由配置模块类似。一个**Route模块**由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。

**（3）Predicate（断言）**：

这是一个 Java 8 新特性中提供的函数式接口，可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。**断言的**输入类型是一个 ServerWebExchange。

### 1.3 Spring Cloud Getway 与Zuul 的区别

![](https://image.easyblog.top/image-20210925181118032.png)



### **1.4 Zuul 1.x 的架构模型**

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

大家知道，servlet由servlet container进行生命周期管理。container启动时构造servlet对象并调用servlet init()进行初始化；container关闭时调用servlet destory()销毁servlet；container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service()。

弊端：servlet是一个简单的网络IO模型，当请求进入servlet container时，servlet container就会为其绑定一个线程，在并发不高的场景下这种模型是适用的，但是一旦并发上升，线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单的业务场景下，不希望为每个request分配一个线程，只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210925183153.png" style="zoom:80%;" />

所以Springcloud Zuul 是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servlet（DispatcherServlet），并由该servlet阻塞式处理处理。所以Springcloud Zuul无法摆脱servlet模型的弊端。虽然Zuul 2.0开始，使用了Netty，并且已经有了大规模Zuul 2.0集群部署的成熟案例，但是，Springcloud官方已经没有集成改版本的计划了。



### 1.5 Getway的架构模型

Spring在2017年下半年迎来了Webflux，Webflux的出现填补了Spring在响应式编程上的空白，Webflux的响应式编程不仅仅是编程风格的改变，而且对于一系列的著名框架，都提供了响应式访问的开发包，比如Netty、Redis等等。

SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。

Webflux模式替换了旧的Servlet线程模型。用少量的线程处理request和response io操作，这些线程称为Loop线程，而业务交给响应式编程框架处理，响应式编程是非常灵活的，用户可以将业务中阻塞的操作提交到响应式框架的work线程中执行，而不阻塞的操作依然可以在Loop线程中进行处理，大大提高了Loop线程的利用率。官方结构图：

![](https://image.easyblog.top/image-20210925181446108.png)

Webflux虽然可以兼容多个底层的通信框架，但是一般情况下，底层使用的还是Netty，毕竟，Netty是目前业界认可的最高性能的通信框架。而Webflux的Loop线程，正好就是著名的Reactor 模式IO处理模型的Reactor线程，如果使用的是高性能的通信框架Netty，这就是Netty的EventLoop线程。

关于Reactor线程模型，和Netty通信框架的知识，是Java程序员的重要、必备的内功，个中的原理，具体请参见尼恩编著的《Netty、Zookeeper、Redis高并发实战》一书，这里不做过多的赘述。

![](https://image.easyblog.top/image-20210925181526722.png)

### 1.6 Spring Cloud Gateway的工作流程

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。下图是Spring Cloud官网给出的Getway工作流程图：

<img src="https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/images/spring_cloud_gateway_diagram.png" style="zoom:80%;" />

![](https://image.easyblog.top/image-20210925183757671.png)

## 二、基于Getway配置路由

首先搭建起一个基本的Getway微服务模块，具体参考步骤（1）~ （）进行操作。

**（1）新建微服务模块cloud-gatway-9527**

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210925185131.png)

**（2）引入Spring Cloud Getway 的 POM 依赖**

```xml
<dependency>                                              
    <groupId>org.springframework.cloud</groupId>          
    <artifactId>spring-cloud-starter-gateway</artifactId> 
</dependency> 
```

**（3）新建启动类GatwayApplication**

```java
@SpringBootApplication
public class GatwayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatwayApplication.class, args);
    }


}
```

至此，Getway 微服务模块的基本搭建工作告一段落，接下来我们将使用3中方式来对服务Provider `cloud-provider-payment` 配置路由，可以对它所提供的服务搂一眼：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210925191214.png)

### 2.1  基于配置文件的路由配置方式

Getway是非常简单易容的，通常我们可以只用少量的配置就可以对一个服务进行路由的配置，比如对上面两个服务的路由配置文件示例如下：

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-getway-service
    gateway:
      routes:
        - id: payment-route            #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8009   #匹配后提供服务的路由地址
          predicates:                  #断言,会将路径相匹配的进行路由
            - Path=/payment/**

        - id: payment-route2
          uri: http://localhost:8009
          predicates:
            - Path=/payment/port
```

上面配置了两个路由规则，第一个可以路由到getPayment方法，第二个可以路由到getServerPort方法，配置中各字段具体含义如下：

* **id**：我们自定义的路由 ID，没有固定规则但要求唯一，建议配合服务名

* **uri**：目标服务地址，匹配后会路由到对应服务器

* **predicates**：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）

### 2.2 基于代码的路由配置方式

转发功能同样可以通过代码来实现，我们可以在模块中添加配置类 GatewayConfig 并提供方法customRouteLocator() 来定制转发规则。

```java
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder){
        return builder.routes()
                //一个route就是配置一条路由
                .route("payment-route ",r->r.path("/payment/**").uri("http://localhost:8009"))
                .route("payment-route2 ",r->r.path("/payment/port").uri("http://localhost:8009"))
                .build();
    }


}
```

我们在yaml配置文件中注销掉相关路由的配置，重启服务，访问链接：http://127.0.0.1:9527/port/1 和 http://127.0.0.1:9527/payment/port， 可以看到和上面一样的页面，证明我们测试成功。

![](https://image.easyblog.top/3.gif)

### 2.3 基于微服务名实现动态路由配置

默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。配置的方法值在uri的schema协议部分为Spring Cloud自定义的lb协议，表示从微服务注册中心（如Eureka、Consul）订阅服务，并且进行服务的路由。

需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能。`lb://serviceName` 是spring cloud gateway在微服务中自动为我们创建的负载均衡uri。如下是对2.1中示例配置方式的一个改进：

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-getway-service
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        instance-id: cloud-getway-service  #设置实例id，默认是项目的名称
        prefer-ip-address: true   #显示客户端的ip地址
    gateway:
      discovery:
         locator: 
             enbale: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment-route            #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://consul-provider-payment  #配置规则： lb://微服务名称
          predicates:                  #断言,会将路径相匹配的进行路由
            - Path=/payment/**

        - id: payment-route2
          uri: lb://consul-provider-payment
          predicates:
            - Path=/payment/port
```



## 三、路由匹配规则

Spring Cloud Gateway 的功能很强大，我们仅仅通过 Predicates 的设计就可以看出来，前面我们只是使用了 predicates 进行了简单的条件匹配，其实 Spring Cloud Gataway 帮我们内置了很多 Predicates 功能。

Spring Cloud Gateway 是通过 Spring WebFlux 的 HandlerMapping 做为底层支持来匹配到转发路由，Spring Cloud Gateway 内置了很多 Predicates 工厂，这些 Predicates 工厂通过不同的 HTTP 请求参数来匹配，多个 Predicates 工厂可以组合使用。

![](https://img-blog.csdnimg.cn/20200527213652534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NzIyNDc1,size_16,color_FFFFFF,t_70)

gateWay的主要功能之一是转发请求，转发规则的定义主要包含三个部分：

| 转发规则定义            | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Route（路由）           | 路由是网关的基本单元，由ID、URI、一组Predicate、一组Filter组成，根据Predicate进行匹配转发。 |
| Predicate（谓语、断言） | 路由转发的判断条件，目前SpringCloud Gateway支持多种方式，常见如：Path、Query、Method、Header等，写法必须遵循 key=vlue的形式 |
| Filter（过滤器）        | 过滤器是路由转发请求时所经过的过滤逻辑，可用于修改请求、响应内容 |

### 3.1 断言条件/转发规则（Predicate）

Predicate 来源于 Java 8，是 Java 8 中引入的一个函数，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。可以用于接口请求参数校验、判断新老数据是否有变化需要进行更新操作。

在 Spring Cloud Gateway 中 Spring 利用 Predicate 的特性实现了各种路由匹配规则，有通过 Header、请求参数等不同的条件来进行作为条件匹配到对应的路由。网上有一张图总结了 Spring Cloud 内置的几种 Predicate 的实现。

![](https://upload-images.jianshu.io/upload_images/19816137-bb046dbf19bee1b4.gif?imageMogr2/auto-orient/strip)

上面这些转发实现类分别对应以下转发规则（predicates），假设转发uri都设定为***[http://localhost:8001](http://localhost:9023/)***

| 规则    | 实例                                                         | 说明                                                         |
| :------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Path    | - Path=/gate/**,/rule/**                                     | 当请求的路径为gate、rule开头的时，转发到http://localhost:8001服务器上 |
| Before  | - Before=2017-01-20T17:42:47.789-07:00[America/Denver]       | 在某个时间之前的请求才会被转发到 http://localhost:8001服务器上 |
| After   | - After=2017-01-20T17:42:47.789-07:00[America/Denver]        | 在某个时间之后的请求才会被转发                               |
| Between | - Between=2017-01-20T17:42:47.789-07:00[America/Denver],2017-01-21T17:42:47.789-07:00[America/Denver] | 在某个时间段之间的才会被转发                                 |
| Cookie  | - Cookie=chocolate, ch.p                                     | 名为chocolate的表单或者满足正则ch.p的表单才会被匹配到进行请求转发 |
| Header  | - Header=X-Request-Id, \d+                                   | 携带参数X-Request-Id或者满足\d+的请求头才会匹配              |
| Host    | - Host=www.hd123.com                                         | 当主机名为www.hd123.com的时候直接转发到http://localhost:8001服务器上 |
| Method  | - Method=GET                                                 | 只有GET方法才会匹配转发请求，还可以限定POST、PUT等请求方式   |





### 3.2 过滤器规则（Filter）

![](https://image.easyblog.top/image-20210930233414321.png)注：当配置多个filter时，优先定义的会被调用，剩余的filter将不会生效

#### PrefixPath   对所有的请求路径添加前缀

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

访问/hello的请求被发送到https://example.org/mypath/hello。

#### RedirectTo  重定向

重定向，配置包含重定向的返回码和地址

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

#### RemoveRequestHeader   去掉请求头信息

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

去掉请求头信息 X-Request-Foo

#### RemoveResponseHeader   去掉响应头信息

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Request-Foo
```

#### RemoveRequestParameter   去掉请求参数信息

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

#### RewritePath   改写路径

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: rewrite_filter
        uri: http://localhost:8081
        predicates:
        - Path=/test/**
        filters:
        - RewritePath=/where(?<segment>/?.*), /test(?<segment>/?.*) 
```

将/where/... 改成 test/...

#### SetPath   设置请求路径

功能基本的和 RewritePath类似

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

#### SetRequestHeader   设置请求头信息

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

#### SetStatus  设置响应状态码

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

#### StripPrefix   跳过指定路径

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

#### RequestSize  请求大小

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

超过5M的请求会返回413错误。

#### Default-filters  对所有请求添加全局过滤器

```yml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```



### 3.3 通过代码的方式配置

通过代码进行配置，将路由规则设置为一个Bean即可：

```java
	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
			.route("path_route", r -> r.path("/get")
				.uri("http://httpbin.org"))
			.route("host_route", r -> r.host("*.myhost.org")
				.uri("http://httpbin.org"))
			.route("rewrite_route", r -> r.host("*.rewrite.org")
				.filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/${segment}"))
				.uri("http://httpbin.org"))
			.route("hystrix_route", r -> r.host("*.hystrix.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd")))
				.uri("http://httpbin.org"))
			.route("hystrix_fallback_route", r -> r.host("*.hystrixfallback.org")
				.filters(f -> f.hystrix(c -> c.setName("slowcmd").setFallbackUri("forward:/hystrixfallback")))
				.uri("http://httpbin.org"))
			.route("limit_route", r -> r
				.host("*.limited.org").and().path("/anything/**")
				.filters(f -> f.requestRateLimiter(c -> c.setRateLimiter(redisRateLimiter())))
				.uri("http://httpbin.org"))
			.build();
	}
```



## 四、Gatway 网关过滤器自定义开发

### 4.1 过滤器的执行次序

Spring-Cloud-Gateway 基于过滤器实现，同 zuul 类似，有**pre**和**post**两种方式的 filter,分别处理**前置逻辑**和**后置逻辑**。客户端的请求先经过**pre**类型的 filter，然后将请求转发到具体的业务服务，收到业务服务的响应之后，再经过**post**类型的 filter 处理，最后返回响应到客户端。

过滤器执行流程如下图所示，**order 越大，优先级越低**。

<img src="https://image.easyblog.top/spring-cloud-gateway-fliter-order.png" style="zoom:40%;" />

Filter同时又分为全局过滤器和局部过滤器

- **全局过滤器：**对所有路由生效

- **局部过滤器：**对指定路由生效



### 4.2 自定义全局过滤器

实现 `GlobalFilter` 接口和 `Ordered` 接口，重写相关方法，注册到到Spring容器管理即可，无需配置，全局过滤器对所有的路由都有效。

全局过滤器示例代码如下：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Configuration
public class FilterConfig{

    @Bean
    @Order(-1)
    public GlobalFilter a(){
        return new GlobalUserFilter();
    }

    @Bean
    @Order(0)
    public GlobalFilter b(){
        return new BFilter();
    }

    @Bean
    @Order(1)
    public GlobalFilter c(){
        return new CFilter();
    }


    @Slf4j
    public class GlobalUserFilter implements GlobalFilter, Ordered {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("执行 GlobalUserFilter 前置逻辑.....");
            String uname = exchange.getRequest().getQueryParams().getFirst("username");
            if (StringUtils.isEmpty(uname)) {
               log.info("*****用户名为Null 非法用户,(┬＿┬)");
               exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应
               return exchange.getResponse().setComplete();
            }
            return chain.filter(exchange).then(Mono.fromRunnable(() ->{
                log.info("AFilter后置逻辑");
            }));
        }

        //   值越小，优先级越高
        //    int HIGHEST_PRECEDENCE = -2147483648;
        //    int LOWEST_PRECEDENCE = 2147483647;
        @Override
        public int getOrder(){
            return HIGHEST_PRECEDENCE + 100;
        }
    }

    @Slf4j
    public class BFilter implements GlobalFilter, Ordered {
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("BFilter前置逻辑");
            return chain.filter(exchange).then(Mono.fromRunnable(() ->
            {
                log.info("BFilter后置逻辑");
            }));
        }


        //   值越小，优先级越高
        //    int HIGHEST_PRECEDENCE = -2147483648;
        //    int LOWEST_PRECEDENCE = 2147483647;
        @Override
        public int getOrder() {
            return HIGHEST_PRECEDENCE + 200;
        }
    }

    @Slf4j
    public class CFilter implements GlobalFilter, Ordered {

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            log.info("CFilter前置逻辑");
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                log.info("CFilter后置逻辑");
            }));
        }

        //   值越小，优先级越高
        //   int HIGHEST_PRECEDENCE = -2147483648;
        //   int LOWEST_PRECEDENCE = 2147483647;
        @Override
        public int getOrder() {
            return HIGHEST_PRECEDENCE + 300;
        }
    }
}
```



### 4.3 自定义局部过滤器

步骤：

1 需要实现GatewayFilter, Ordered，实现相关的方法

2 加入到过滤器工厂，并且注册到spring容器中。

3、在配置文件中进行配置，如果不配置则不启用此过滤器规则。

局部过滤器举例, 对请求头部的 user-id 进行校验，代码如下：

#### （1）需要实现GatewayFilter, Ordered 接口，实现相关的方法

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Slf4j
public class UserIdCheckGateWayFilter implements GatewayFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String url = exchange.getRequest().getPath().pathWithinApplication().value();
        log.info("请求URL:" + url);
        log.info("method:" + exchange.getRequest().getMethod());
         //获取param 请求参数
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        //获取header
        String userId = exchange.getRequest().getHeaders().getFirst("user-id");
        log.info("userId：" + userId);

        if (StringUtils.isBlank(userId)) {
            log.info("*****头部验证不通过，请在头部输入  user-id");
            //终止请求，直接回应
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    //   值越小，优先级越高
    //    int HIGHEST_PRECEDENCE = -2147483648;
    //    int LOWEST_PRECEDENCE = 2147483647;
    @Override
    public int getOrder() {
        return HIGHEST_PRECEDENCE;
    }
}
```

#### （2）加入到过滤器工厂，并且注册到spring容器中

```java
import com.crazymaker.cloud.nacos.demo.gateway.filter.UserIdCheckGateWayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.stereotype.Component;

@Component
public class UserIdCheckGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
    @Override
    public GatewayFilter apply(Object config) {
        return new UserIdCheckGateWayFilter();
    }
}
```

#### （3）在配置文件中进行配置

在配置文件中配置一下自定义的过滤器，如果不配置则不会启用此过滤器规则

```yml
spring:
  cloud: 
     gateway:
       rotues:
          - id: service_provider_demo_route_filter
          uri: lb://service-provider-demo
          predicates:
            - Path=/filter/**
          filters:
            - RewritePath=/filter/(?<segment>.*), /provider/$\{segment}
            - UserIdCheck
```





## 参考资料

* https://www.cnblogs.com/crazymakercircle/p/11704077.html