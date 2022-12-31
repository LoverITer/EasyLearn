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

**（1）value, name**

value和name的作用一样，如果没有配置url那么配置的值将作为服务名称，用于服务发现。反之只是一个名称。

**（2）serviceId**

serviceId已经废弃了，直接使用name即可。

**（3）contextId**

比如我们有个user服务，但user服务中有很多个接口，我们不想将所有的调用接口都定义在一个类中，比如：

Client 1

```less
@FeignClient(name = "optimization-user")
public interface UserRemoteClient {
    @GetMapping("/user/get")
    public User getUser(@RequestParam("id") int id);
}
```

Client 2

```less
@FeignClient(name = "optimization-user")
public interface UserRemoteClient2 {
    @GetMapping("/user2/get")
    public User getUser(@RequestParam("id") int id);
}
```

这种情况下启动就会报错了，因为Bean的名称冲突了，具体错误如下：

```mipsasm
Description:
The bean 'optimization-user.FeignClientSpecification', defined in null, could not be registered. A bean with that name has already been defined in null and overriding is disabled.
Action:
Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

解决方案可以增加下面的配置，作用是允许出现beanName一样的BeanDefinition。

```ini
spring.main.allow-bean-definition-overriding=true
```

另一种解决方案就是为每个Client手动指定不同的contextId，这样就不会冲突了。

上面给出了Bean名称冲突后的解决方案，下面来分析下contextId在Feign Client的作用，在注册Feign Client Configuration的时候需要一个名称，名称是通过getClientName方法获取的：

```applescript
String name = getClientName(attributes);

registerClientConfiguration(registry, name,
attributes.get("configuration"));
private String getClientName(Map<String, Object> client) {
    if (client == null) {
      return null;
    }
    String value = (String) client.get("contextId");
    if (!StringUtils.hasText(value)) {
      value = (String) client.get("value");
    }
    if (!StringUtils.hasText(value)) {
      value = (String) client.get("name");
    }
    if (!StringUtils.hasText(value)) {
      value = (String) client.get("serviceId");
    }
    if (StringUtils.hasText(value)) {
      return value;
    }


    throw new IllegalStateException("Either 'name' or 'value' must be provided in @"
        + FeignClient.class.getSimpleName());
  }
```

可以看到如果配置了contextId就会用contextId，如果没有配置就会去value然后是name最后是serviceId。默认都没有配置，当出现一个服务有多个Feign Client的时候就会报错了。

其次的作用是在注册FeignClient中，contextId会作为Client 别名的一部分，如果配置了qualifier优先用qualifier作为别名。

```dart
private void registerFeignClient(BeanDefinitionRegistry registry,
      AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
    String className = annotationMetadata.getClassName();
    BeanDefinitionBuilder definition = BeanDefinitionBuilder
        .genericBeanDefinition(FeignClientFactoryBean.class);
    validate(attributes);
    definition.addPropertyValue("url", getUrl(attributes));
    definition.addPropertyValue("path", getPath(attributes));
    String name = getName(attributes);
    definition.addPropertyValue("name", name);
    String contextId = getContextId(attributes);
    definition.addPropertyValue("contextId", contextId);
    definition.addPropertyValue("type", className);
    definition.addPropertyValue("decode404", attributes.get("decode404"));
    definition.addPropertyValue("fallback", attributes.get("fallback"));
    definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
    definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

    // 拼接别名
    String alias = contextId + "FeignClient";
    AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();


    boolean primary = (Boolean) attributes.get("primary"); // has a default, won't be
                                // null


    beanDefinition.setPrimary(primary);

    // 配置了qualifier优先用qualifier
    String qualifier = getQualifier(attributes);
    if (StringUtils.hasText(qualifier)) {
      alias = qualifier;
    }


    BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
        new String[] { alias });
    BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
  }
```

**（4）url**

url用于配置指定服务的地址，相当于直接请求这个服务，不经过Ribbon的服务选择。像调试等场景可以使用。

**（5）decode404**

当调用请求发生404错误时，decode404的值为true，那么会执行decoder解码，否则抛出异常。

解码也就是会返回固定的数据格式给你：

```json
{"timestamp":"2020-01-05T09:18:13.154+0000","status":404,"error":"Not Found","message":"No message available","path":"/user/get11"}
```

抛异常的话就是异常信息了，如果配置了fallback那么就会执行回退逻辑：
![xxx.png](https://segmentfault.com/img/bVbCpAe)

**（6）configuration**

configuration是配置Feign配置类，在配置类中可以自定义Feign的Encoder、Decoder、LogLevel、Contract等。

configuration定义

```typescript
public class FeignConfiguration {
    @Bean
    public Logger.Level getLoggerLevel() {
        return Logger.Level.FULL;
    }
    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
    
    @Bean
    public CustomRequestInterceptor customRequestInterceptor() {
        return new CustomRequestInterceptor();
    }
    // Contract,feignDecoder,feignEncoder.....
}
```

使用示列

```less
@FeignClient(value = "optimization-user", configuration = FeignConfiguration.class)
public interface UserRemoteClient {
    
    @GetMapping("/user/get")
    public User getUser(@RequestParam("id")int id);
    
}
```

**（7）fallback**

定义容错的处理类，也就是回退逻辑，fallback的类必须实现Feign Client的接口，无法知道熔断的异常信息。

fallback定义

```aspectj
@Component
public class UserRemoteClientFallback implements UserRemoteClient {
    @Override
    public User getUser(int id) {
        return new User(0, "默认fallback");
    }
    
}
```

使用示列

```less
@FeignClient(value = "optimization-user", fallback = UserRemoteClientFallback.class)
public interface UserRemoteClient {
    
    @GetMapping("/user/get")
    public User getUser(@RequestParam("id")int id);
    
}
```

**（8）fallbackFactory**

也是容错的处理，可以知道熔断的异常信息。

fallbackFactory定义

```aspectj
@Component
public class UserRemoteClientFallbackFactory implements FallbackFactory<UserRemoteClient> {
    private Logger logger = LoggerFactory.getLogger(UserRemoteClientFallbackFactory.class);
    
    @Override
    public UserRemoteClient create(Throwable cause) {
        return new UserRemoteClient() {
            @Override
            public User getUser(int id) {
                logger.error("UserRemoteClient.getUser异常", cause);
                return new User(0, "默认");
            }
        };
    }
}
```

使用示列

```less
@FeignClient(value = "optimization-user", fallbackFactory = UserRemoteClientFallbackFactory.class)
public interface UserRemoteClient {
    
    @GetMapping("/user/get")
    public User getUser(@RequestParam("id")int id);
    
}
```

**（9）path**

path定义当前FeignClient访问接口时的统一前缀，比如接口地址是/user/get, 如果你定义了前缀是user, 那么具体方法上的路径就只需要写/get 即可。

使用示列

```less
@FeignClient(name = "optimization-user", path="user")
public interface UserRemoteClient {
    
    @GetMapping("/get")
    public User getUser(@RequestParam("id") int id);
}
```

 **（10）primary**

primary对应的是@Primary注解，默认为true，官方这样设置也是有原因的。当我们的Feign实现了fallback后，也就意味着Feign Client有多个相同的Bean在Spring容器中，当我们在使用@Autowired进行注入的时候，不知道注入哪个，所以我们需要设置一个优先级高的，@Primary注解就是干这件事情的。

**（11）qualifier**

qualifier对应的是@Qualifier注解，使用场景跟上面的primary关系很淡，一般场景直接@Autowired直接注入就可以了。

如果我们的Feign Client有fallback实现，默认@FeignClient注解的primary=true, 意味着我们使用@Autowired注入是没有问题的，会优先注入你的Feign Client。

如果你鬼斧神差的把primary设置成false了，直接用@Autowired注入的地方就会报错，不知道要注入哪个对象。

解决方案很明显，你可以将primary设置成true即可，如果由于某些特殊原因，你必须得去掉primary=true的设置，这种情况下我们怎么进行注入，我们可以配置一个qualifier，然后使用@Qualifier注解进行注入，示列如下：

Feign Client定义

```less
@FeignClient(name = "optimization-user", path="user", qualifier="userRemoteClient")
public interface UserRemoteClient {
    
    @GetMapping("/get")
    public User getUser(@RequestParam("id") int id);
}
```

Feign Client注入

```less
@Autowired
@Qualifier("userRemoteClient")
private UserRemoteClient userRemoteClient;
```

   

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