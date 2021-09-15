## Spring Cloud — Ribbon负载均衡与服务调用



## 一、Ribbon简介

**是什么**

![](https://image.easyblog.top/image-20210912221846300.png)

官网资料：https://github.com/Netflix/ribbon/wiki/Getting-Started



**能干嘛**

* 集中式LB

  ![](https://image.easyblog.top/image-20210912222157705.png)

* 进程内LB

  ![](https://image.easyblog.top/image-20210912222210588.png)

**总结**：Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。



## 二、Ribbon 负载均衡算法

Ribbon核心组件IRule接口提供了Ribbon实现负载均衡七大算法，类图如下所示：

![](https://image.easyblog.top/image-20210912222530759.png)

### 2.1 com.netflix.loadbalancer.RoundRobinRule

轮询，Ribbon默认的负载均衡算法

### 2.2 com.netflix.loadbalancer.RandomRule

随机

### 2.3 com.netflix.loadbalancer.RetryRule

先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试

### 2.4 WeightedResponseTimeRule 

对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择

### 2.5 BestAvailableRule 

会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

### 2.6 AvailabilityFilteringRule 

先过滤掉故障实例，再选择并发较小的实例

### 2.7 ZoneAvoidanceRule

默认规则，复合判断server所在区域的性能和server的可用性选择服务器



## 三、Ribbon负载均衡算法的切换



**配置细节**：Ribbon配置类不要放在和启动类可以扫描到的路径，原因如下图官方说明所示：

![](https://image.easyblog.top/image-20210912223512731.png)