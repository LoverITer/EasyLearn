## Spring Cloud — OpenFeign服务接口调用



## 一、OpenFeign简介

### 1.1 OpenFeign是什么？

![](https://image.easyblog.top/image-20210919103254547.png)



![](https://image.easyblog.top/image-20210919103322809.png)



OPenFegin 项目 GitHub地址：https://github.com/spring-cloud/spring-cloud-openfeign



### 1.2 OpenFeign能干什么？

![](https://image.easyblog.top/image-20210919103547882.png)



### 1.3 OpenFeign和 Feign的区别

![](https://image.easyblog.top/image-20210919103625590.png)



## 二、使用OpenFeign调用微服务接口

#### 2.1 在服务Consumer 项目的 `pom.xml` 中引入OpenFeign的依赖

```xml
<dependency>                                               
    <groupId>org.springframework.cloud</groupId>           
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>                                              
```

#### 2.2 在服务Consumer 项目的 `application.yml` 中做如下配置

```yml
server:
  port: 80

spring:
  application:
    name: payment-consul-consumer
  cloud:
    # 这里使用consul作为服务注册与发现组件
    consul:
        host: localhost
        port: 8500
        discovery:
          service-name: ${spring.application.name}

url:
  payment: consul-provider-payment   #配置服务名称
```

#### 2.3 启动类加注@EnableFeignClients开启Feign

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;


@SpringBootApplication
@EnableFeignClients  //开启Feign，会自动扫描标记有@FeignClient注解的接口成为组件
public class FeignOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignOrderApplication.class,args);
    }
    
}
```

#### 2.4 编写Feign客户端

Feign 客户端一般是一个接口，因此我们需要提供一个接口，并在接口定义上使用 `@FeignClient` 注解，然后在接口中可以像编写Spring MVC 控制器（Controller）一样使用各种Mapping注解以及参数类注解来调用目标方法。


```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;


@FeignClient(name = "${url.payment}")
public interface PaymentClient {

    //这里直接写调用目标服务的接口路径
    @GetMapping(value = "/payment/consul")
    String getPayment();

}
```

上面的代码中使用到了 `@FeignClient` 注解的 `name` 属性。除此之外，`@FeignClient` 注解的常用属性如下：
                                             
* `name`：指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现
* `url`: url一般用于调试，可以手动指定@FeignClient调用的地址
* `decode404`:当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException
* `configuration`: Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract
* `fallback`: 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口，通常配合服务熔断（Hystrix）进行使用
* `fallbackFactory`: 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码
* `path`: 定义当前FeignClient的统一前缀

#### 2.5 提供Controller用于测试

这里业务比较简单，直接在controller

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import top.easyblog.promethues.client.PaymentClient;


@RestController
public class PaymentOrderController {

    @Autowired
    private PaymentClient paymentClient;

    @GetMapping("/consumer/payment")
    public String getPayment(){
        return paymentClient.getPayment();
    }
}
```

完成之后，首先在本地8500端口启动consul，之后在本地8006~8009启动4个provider，最后在本地80端口启动上面编写的消费者，在浏览器访问：http://127.0.0.1/consumer/payment ，执行结果如下GIF动图所示：

![](https://image.easyblog.top/2.gif)



#### 2.6 总结

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919120219.png" style="width:70%;" />

## 三、OpenFeign超时防控

### 3.1 OpenFeign调用超时演示

基于第二节的配置，我们在服务Provider中增加以下接口方法，模拟一个比较耗时的业务逻辑：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
public class PaymentConsulController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/timeput")
    public String timeout(){
        try {
            Thread.sleep(5000);
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
        return String.format("%s",serverPort);
    }
}
```



同时，在消费者端FeignClient中也增加对应方法的调用：

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient(name = "${client.payment}")
public interface PaymentClient {

    @GetMapping(value = "/payment/consul")
    String getPayment();

    //模拟超时业务
    @GetMapping(value = "/payment/timeput")
    String timeout();

}
```



提供controller，调用FeignClient的方法：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import top.easyblog.promethues.client.PaymentClient;


@RestController
public class PaymentOrderController {

    @Autowired
    private PaymentClient paymentClient;

    @GetMapping("/consumer/payment")
    public String getPayment(){
        return paymentClient.getPayment();
    }

    @GetMapping(value = "/consumer/timeout")
    public String timeout(){
        return paymentClient.timeout();
    }
}
```



在浏览器输入：http://127.0.0.1/consumer/timeout ，执行结果如下图所示，可以看到发生了调用超时。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919165609.png" style="width:60%;" />





### 3.2 解决OpenFeign调用超时报错

在feign.Request里面有一个内部类 `Options`，如果不配置超时，外部会调用下面红色框所标记的构造函数，连接超时10s，读超时60s

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919172205.png" style="width:60%;" />

可是这和我们的实现结果并不一致，据需寻找原因发现在`RibbonClientConfiguration`类定义中，请求连接时间和超时时间，默认为1秒，被覆盖后也是会读超时的。

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919173014.png" style="width:60%;" />

并且在`FeignLoadBalancer`类中发现，如果我们在程序中没有显示的配置feign超时时间，默认的 `Options`上面的时间也会被ribbon覆盖，覆盖超时时间设置的代码如下图所示：

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919173704.png" style="width:80%;" />

知道了原因，解决问题就简单了，在程序中新增如下Feign配置类：

```java
import feign.Request;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
public class CommonFeignConfig {

    @Value("${ribbon.read-timeout:6000}")
    private int readTimeout;

    @Value("${ribbon.connect-timeout:3000}")
    private int connectTimeout;


    @Bean
    public Request.Options options() {
        return new Request.Options(connectTimeout, readTimeout);
    }
}
```



配置好之后重新启动客户端消费者的服务，在浏览器中访问地址：http://127.0.0.1/consumer/timeout，执行发现可以正常访问服务了。



## 四、OpenFeign 日志打印

### 4.1 OpenFeign日志概述

![](https://image.easyblog.top/image-20210919175913531.png)

OpenFeign提供了一下四个级别的日志：

* **NONE**：默认的，不显示任何日志。

* **BASIC**：仅仅记录请求方法、url、响应状态码、执行时间

* **HEADERS**：除了BASIC信息还添加了请求和响应的头信息

* **PULL**：除了HEADERS信息还添加了请求和响应的正文和元信息

### 4.2 配置日志级别

在Feign配置类配置日志级别：

```java
import feign.Logger;
import feign.Request;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CommonFeignConfig {

    @Bean
    public Logger.Level logger(){
        return Logger.Level.FULL;
    }

}
```

### 4.3 在配置文件中声明需要输出日志的接口

在 `application.yml` 中声明需要开启日志打印的FeignClient接口，如下配置会打开项目client包下所有FeignClient接口的日志打印功能：

```java
logging:
  level:
    top.easyblog.promethues.client.*: debug
```

#### 4.4 测试

在浏览器访问地址：http://127.0.0.1/consumer/payment ，打开控制台看到Feign执行过程中打印的日志如下：

<img src="https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210919180224.png" style="width:70%;" />