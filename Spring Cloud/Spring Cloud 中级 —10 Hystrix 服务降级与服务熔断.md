## Spring Cloud 中级 — Hystrix 服务降级与服务熔断



## 一、背景：分布式系统面临的问题

几个微服务之间的相互调用，肯定会导致链路越来越长。一旦某个服务不可用就会导致整条链路上的服务都出事。 所以分布式系统实际面临一个问题：复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败：网络故障，出错，宕机，断电...... 都可能会导致服务雪崩。    

![](https://image.easyblog.top/image-20210919181837154.png)

###  服务雪崩      

 多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”。        如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓雪崩效应。     

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒内饱和。 比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加、备份队列、线程和其他系统资源紧张， 导致整个系统发生更多的级联故障。 这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不会破坏整个应用程序或系统。 

所以，通常当你发现一个模块的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。    



## 二、Hystrix 简介

**Hystrix**是一个用于处理分布式系统**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等。  Hystrix能够保证在一个依赖出问题的情况下，**不会导致整个服务失败，避免级联故障，以提高分布式系统的弹性**。        

“断路器”本身就是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个符合预期的，可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间，不必要的占用，从而避免了故障在分布式系统中的蔓延乃至雪崩。      

Hystrix的主要功能：       

* 服务降级    

* 服务熔断    

* 接近实时的监控        

* 服务限流    

* 隔离 

  ......

官方文档：https://github.com/Netflix/Hystrix/wiki/How-To-Use



## 三、Hystrix 重要概念

### 3.1 Hystrix设计理念

Hystrix是基于 **命令模式** 设计的，通过UML图先直观的认识一下这一设计模式。

![](https://image.easyblog.top/8057a36c6ac84660b4f6b86b38f20971_th.jpg)

可见，Command是在Receiver和Invoker之间添加的中间层，Command实现了对Receiver的封装。那么Hystrix的应用场景如何与上图对应呢？

API既可以是Invoker又可以是Reciever，通过继承Hystrix核心类 **`HystrixCommand`** 或者 **`@HystrixCommand`** 来封装这些API（例如，远程接口调用，数据库查询之类可能会产生延时的操作）。就可以为API提供弹性保护了。

### 3.2 Hystrix如何解决依赖隔离

1. Hystrix使用命令模式HystrixCommand(Command)包装依赖调用逻辑，每个命令在单独线程中/信号授权下执行。
2. 可配置依赖调用超时时间，超时时间一般设为比99.5%平均时间略高即可.当调用超时时，直接返回或执行fallback逻辑。
3. Hystrix为每个依赖提供一个小的线程池（或信号），如果线程池已满调用将被立即拒绝，默认不采用排队，加快失败判定时间。
4. 依赖调用结果分:成功，失败（抛出异常），超时，线程拒绝，短路。 请求失败(异常，拒绝，超时，短路)时执行fallback(降级)逻辑。
5. 提供熔断器组件,可以自动运行或手动调用,停止当前依赖一段时间(10秒)，熔断器默认错误率阈值为50%,超过将自动运行。
6. 提供近实时依赖的统计和监控。

### 3.3 服务降级

服务降级是指上游依赖的某服务响应结果延迟，导致整理服务质量不高，为了提高服务吞吐量，将一些不重要或者服务质量较差的服务延迟使用或暂停使用。

Hystrix通过将依赖的服务调用封装在Command中，监控统计服务调用情况，分析响应结果，包含：成功、失败、拒绝、超时。在服务窗口期内，当请求量超过指定值，错误率超过阀值的服务调用，将会开启熔断器，进行服务降级，直接返回友好信息或者使用简单的逻辑进行处理，缓解依赖服务的压力，减少请求的积压，避免问题的蔓延。

**Hystrix实现服务降级主要从三个方面实施：**

**●隔离（线程池隔离和信号量隔离）：**限制调用分布式服务的资源使用，某一个调用的服务出现问题不会影响其他服务调用。

**●优雅的降级机制：**超时降级、资源不足时(线程或信号量)降级，降级后可以配合降级接口返回托底数据。

**●融断：**当失败率达到阀值自动触发降级(如因网络故障/超时造成的失败率高)，熔断器触发的快速失败会进行快速恢复。



### 3.4 服务熔断

**（1）服务熔断基本概念**

服务熔断的概念是由大神 ***Martin Fowler*** 在其技术博客 [CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html) 一文中提出的。

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210921120737.png)

**（2）熔断类型**

* 熔断打开（Open）：请求不再进行调用当前服务，内部设置始终一般为MTTP（平均故障处理时间），当打开时长达到时钟所设定的时间就进入版熔断状态（Half Open）
* 熔断关闭（Close）:熔断关闭不会对服务进行熔断
* 熔断半开（Half Open）：部分请求根据规则调用当前服务，如果请求成功且符合规则，那么就认为当前服务已经恢复正常，关闭熔断。

每个熔断器默认维护10个bucket,每秒一个bucket,每个bucket记录成功,失败,超时,拒绝的状态。

默认错误超过50%且10秒内超过20个请求进行中断拦截。

<img src="http://image.mamicode.com/info/201805/20180512110618344184.png" style="zoom:67%;" />

**（3）断路器在什么情况下开始起作用**

![](https://image.easyblog.top/image-20210925160354409.png)

**（4）断路器开启或关闭的的条件**

1. 当满足一定阀值的时候（默认10秒内超过20个请求次数）
2. 当失败率达到一定的时候（默认10秒内超过50%请求失败）
3. 到达以上阀值，断路器将会开启
4. 当开启的时候，所有请求都不会进行转发
5. 一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。重复4和5

**（5）断路器如何恢复主调用链路的**

![](https://image.easyblog.top/image-20210925160641793.png)

### 3.5 服务限流

服务限流就容易理解多了，顾名思义，这是对访问的流量进行限制，就比如上边的场景，我们还可能通过服务限流的方法来解决高并发以及秒杀等问题。主流的限流算法主要有：**漏桶算法**和**令牌算法**



## 四、使用Hystrix构建健壮的服务



首先在已有项目中添加以下Hystrix依赖（本节演示的采用的项目是一个多模块项目，依赖版本已经自父工程中定义好）：

```xml
<dependency>                                                     
    <groupId>org.springframework.cloud</groupId>                 
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>  
```



### 4.1 服务降级配置

服务降级有两种解决思路：可以分别从服务调用者和服务提供者进行服务降级，也就是进行错误的“兜底”

#### 4.1.1 从服务提供者方进行服务降级

  （1）首先在启动类上添加注解：`@EnableCircuitBreaker`

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker    //开启provider侧服务降级
public class PaymentConsulApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentConsulApplication.class,args);
    }
}
```

（2）提供一个模拟超时的方法

```java
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

（3）给方法提供服务降级处理逻辑

```java
@RestController
public class PaymentConsulController {
    @Value("${server.port}")
    private String serverPort;

    @HystrixCommand(fallbackMethod = "timeoutFallBack",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    @GetMapping(value = "/payment/timeput")
    public String timeout(){
        try {
            Thread.sleep(5000);
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
        return String.format("%s",serverPort);
    }

    //降级处理逻辑
    private String timeoutFallBack(){
        return "服务繁忙，请稍后重试！";
    }
}
```

上边这个注解要注意三点：

* `fallCallbackMethod` 中的这个参数就是“兜底”逻辑方法
* `fallCallbackMethod` 中的这个方法的声明要和本方法一致
* `commandProperties` 属性中可以写多个`@HystrixProperty` 注解，其中的name和value就是配置对应的属性，上例中的这个就是配置响应超时，更多的配置可以参考文首给出的Hystrix官方文档。

#### 4.1.2 从消费者方进行服务降级

和在服务提供方进行服务降级相比，在服务调用方（客户端、消费者）进行服务降级是更常用的方法。这两者相比，前者是要让服务提供者对自己可发生的错误进行“预处理”，这样，一定要保证调用者访问到我才会调用这个“兜底”方法。但是，大家想一下，如果我这个服务宕机了呢？客户端根本就调用不到我，它怎么可能接收到我的“兜底”方法呢？所以，在客户端进行服务降级是更常用的方法。

（1）在配置文件 `application.yml` 中增加 配置：`feign.hystrix.enbale=true`

（2）在启动类上添加注解： `@EnableHystrix`

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix   //开启Hystrix断路器保护
public class HystrixOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixOrderApplication.class,args);
    }

}
```

（3）使用@HystrixCommand在消费者controller对应方法上设置降级属性

```java
@RestController
public class OrderController {

    @Autowired
    private PaymentClient paymentClient;

    @GetMapping("/consumer/payment")
    public String getPayment(){
        return paymentClient.getPayment();
    }

    @HystrixCommand(fallbackMethod = "timeoutFallBack",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    @GetMapping(value = "/consumer/timeout")
    public String timeout(){
        //方法正常逻辑
        return paymentClient.timeout();
    }

    //用于处理timeout方法服务降级时的逻辑
    public String timeoutFallBack(){
        return "服务繁忙，请稍候重试！";
    }
}
```



#### 4.1.3 改进现有方案

以上的两种方案看似可行， 但是，实际呢？这是一个合格程序员应该做的事吗？每个接口我们都要写一个fallback方法？然后和我们的业务代码要写在一起？说好的“低耦合，高内聚”呢？我们有必要对已有方案进行改进

* 第一种解决方案：使用`@DefaultProperties` 注解在整个Controller类上，顾名思义，就是给它一个默认的“兜底”方法，就不用每一个需要降级的方法进行设置fallbackMethod了，我们只需要加上@HystrixCommand好了。这个方法太过简单，不做代码演示

* 第二种解决方法：我们在客户端不是通过Feign调用的吗？是有一个Feign的本地接口类，我们直接对这个类进行设置就好了。直接上代码。

```java
@FeignClient(name = "${client.payment}",fallback = PaymentClientFallBack.class)
public interface PaymentClient {

    @GetMapping(value = "/payment/consul")
    String getPayment();

    @GetMapping(value = "/payment/timeput")
    String timeout();

}
```

```java
@Component
public class PaymentClientFallBack implements PaymentClient {

    @Override
    public String getPayment() {
        return this.getClass().getTypeName()+"：服务繁忙或服务异常，请稍后重试！";
    }

    @Override
    public String timeout() {
        return this.getClass().getTypeName()+"：服务繁忙，请稍后重试！";
    }
}
```

以上两种方法的对比：

* 第一种和我们的业务类进行了耦合，而且如果要对每个方法进行fallback，就要多写一个方法，代码太过臃肿。但是，它提供了一个DefaultProperties注解，可以提供默认的方法，这个后者是没有的。这种方法适合直接使用Ribbon结合RestTemplate进行调用的方法。
* 第二种提供了一个Feign接口的实现类来处理服务降级问题，将所有的fallback方法写到了一起，和我们的业务代码完全解耦了。对比第一个，我们可以定义一个统一的方法来实现DefalutPropeties。这种方法适合Feign作为客户端的调用，比较推荐这种。



### 4.2 服务熔断配置

当启用服务降级时，会默认启用服务熔断机制，我们只需要对一些参数进行配置就可以了，就是在上边的@HystrixCommand中的一些属性，比如：

```java
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),  //是否开启断路器
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),   //请求次数
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),  //时间范围
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //失败率达到多少后跳闸
})
```

这些配置可以在官方文档中去了解，也可以打开查看HystrixCommandProperties类中的属性(使用idea一搜索就有)，全部都有默认配置



## 5、Hystrix工作流程



下图所示是Hystrix的工作流程图：

![](https://yqfile.alicdn.com/ca838df2f5fcd368456493811e0b1ce5a96534c4.png)

**Hystrix主要有4种调用方式**：

* `toObservable()`  ：未做订阅，只是返回一个Observable 。
* `observe()`  ：调用 #toObservable() 方法，并向 Observable 注册rx.subjects.ReplaySubject 发起订阅。
* `queue()`  ：调用 #toObservable() 方法的基础上，调用：Observable#toBlocking() 和 BlockingObservable#toFuture() 返回 Future 对象
* `execute() ` ：调用 #queue() 方法的基础上，调用 Future#get() 方法，同步返回 #run() 的执行结果。



**主要的执行逻辑**：

1. 每次调用创建一个新的HystrixCommand,把依赖调用封装在run()方法中.

2. 执行execute()/queue做同步或异步调用.

3. 判断熔断器(circuit-breaker)是否打开,如果打开跳到步骤8,进行降级策略,如果关闭进入步骤.

4. 判断线程池/队列/信号量是否跑满，如果跑满进入降级步骤8,否则继续后续步骤.

5. 调用HystrixCommand的run方法运行依赖逻辑。依赖逻辑调用超时,进入步骤8.

6. 判断逻辑是否调用成功。返回成功调用结果；调用出错，进入步骤8.

7. 计算熔断器状态,所有的运行状态(成功, 失败, 拒绝,超时)上报给熔断器，用于统计从而判断熔断器状态.

8. getFallback()降级逻辑。以下四种情况将触发getFallback调用：

   * run()方法抛出非HystrixBadRequestException异常。
   * run()方法调用超时
   * 熔断器开启拦截调用
   * 线程池/队列/信号量是否跑满

   没有实现getFallback的Command将直接抛出异常，fallback降级逻辑调用成功直接返回，降级逻辑调用失败抛出异常.

9. 返回执行成功结果：Hystrix执行命令成功，它将以可观察到的形式返回响应或响应给调用者



## 6、Hystrix图形化服务监控

**（1）新建图形化Hystrix图形化监控服务模块**

**（2）引入Hystrix图形化服务监控依赖**

```xml
<!--新增hystrix dashboard-->   
<dependency>                                                               
    <groupId>org.springframework.cloud</groupId>                           
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>                                                              
```

**（3）在 `application.yml` 配置文件中配置服务模块服务端口为 9002**

**（4）在启动类上使用 `@EnableHystrixDashboard` 注解开启图形化监控服务**

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class,args);
    }
    
}
```

**（5）启动Hystrix图形化监控服务**

启动Hystrix图形化监控服务，在浏览器地址栏输入：http://127.0.0.1:9002/hystrix，我们将会看到如下Hystrix Dashboard监控首页：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210925171132.png)

**（6）启动服务查看监控数据**

在本地80端口启动消费者服务：`cloud-consumer-payment`  ，它会通过注册中心发现并调用服务提供者 `cloud-provider-payment` 提供的服务 ，我们在 Hystrix Dashboard 进行如下配置：

![](https://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20210925171830.png)

最后点击 `Monitor Stream` 。我们将会看到类似如下监控界面，并且随着我们不断访问，监控界面的各项指标会实时变化。

![](https://image.easyblog.top/image-20210925171439971.png)



监控界面各项指标及其含义如下所示：

![](https://image.easyblog.top/image-20210925171500645.png)

