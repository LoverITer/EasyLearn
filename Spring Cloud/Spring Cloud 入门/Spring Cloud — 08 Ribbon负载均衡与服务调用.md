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



### 3.1 RestTemplate

（1）配置IRule新的实现Bean

RestTemplate 更改负载就均衡策略 只要在Ribbon配置类 RibbonConfig 添加代码即可，代码如下：

```java
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RibbonConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule ribbonRule(){
        //重新定义为 Random 随机策略
        return new RandomRule();
    }

}
```



（2）测试结果：

在本地8006~8009 四个端口分别启动四个Provider，启动之后在浏览器访问地址：http://127.0.0.1/consumer/payment/consul ，执行结果如下GIF动图所示：

![](https://image.easyblog.top/1.gif)

## 四、自定义Ribbon复杂均衡算法

定义 MyCustomRule 类 继承 AbstractLoadBalancerRule 类，重写父类方法。

```java
import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;
import java.util.List;
import java.util.concurrent.ThreadLocalRandom;
 
/**
 * 要求每台服务器被调用5次才能轮询下一个，也就是说以前每台机器一次，现在每台机器5次。
 */
public class MyCustomRule extends AbstractLoadBalancerRule {
    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {
 
    }
 
    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }
 
    /**
     * Randomly choose from all living servers
     */
    //@edu.umd.cs.findbugs.annotations.SuppressWarnings(value = "RCN_REDUNDANT_NULLCHECK_OF_NULL_VALUE")
    //从服务清单中随机选择一个服务实例
    @SuppressWarnings(value = "RCN_REDUNDANT_NULLCHECK_OF_NULL_VALUE")
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;
 
        int total = 0; // 总共被调用的次数，目前要求每台被调用5次
        int currentIndex = 0; // 当前提供服务的机器号
 
        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            //负载均衡来获得可用实例列表upList和allList
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();
            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }
            if (total < 5) {
                server = upList.get(currentIndex);
                total++;
            } else {
                total = 0;
                currentIndex++;
                if (currentIndex >= upList.size()) {
                    currentIndex = 0;
                }
            }
            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }
            if (server.isAlive()) {
                return (server);
            }
            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }
        //正常情况下，每次都应该可以选择一个服务实例
        return server;
    }
 
 
    //随机获取一个随机数
    protected int chooseRandomInt(int serverCount) {
        return ThreadLocalRandom.current().nextInt(serverCount);
    }
}
```



在RestTemplate，配置自定义的策略，代码如下：

```java
import com.netflix.loadbalancer.IRule;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RibbonConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule ribbonRule(){
        //自定义的Rule
        return new MyCustomRule();
    }

}
```

