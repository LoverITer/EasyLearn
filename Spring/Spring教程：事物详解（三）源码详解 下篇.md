## Spring教程：事物详解（三）源码详解



## 引言

上篇我们着重对Spring编程式事物进行了源码阅读以及原理分析，了解了Spring事物的运行原理，但这种管理事务的方式的代码侵入行非常高，现在开发**基本**不会使用这个，而且现在Java开发基本都会使用SpringBoot，配合SpringBoot的自动配置，**声明式事物**简直不要太好用！

本篇就深入SpringBoot源码，看看`@EnableTransactionManagement配合@Transactional` 两个简单的注解（写法简单，但内在其实不简单）是如何帮助我们管理事务的。



## 一、 AOP有关概念回顾

声明式事务是依赖Spring AOP实现的，即面向切面编程。所谓AOP...一句话概括就是：把业务代码中重复代码做成一个切面，提取出来，并定义哪些方法需要执行这个切面。其它的自行百度吧...AOP核心概念如下：

- **通知（Advice**）:定义了切面(各处业务代码中都需要的逻辑提炼成的一个切面)做什么**what+when**何时使用。例如：**前置通知Before、后置通知After、返回通知After-returning、异常通知After-throwing、环绕通知Around.**
- 连接点（Joint point）：程序执行过程中能够插入切面的点，一般有多个。比如调用方式时、抛出异常时。
- 切点（Pointcut）:切点定义了连接点，切点包含多个连接点,即where哪里使用通知.通常指定类+方法 或者 正则表达式来匹配 类和方法名称。
- **切面（Aspect）**:切面=通知+切点，即**when+where+what**何时何地做什么。
- 引入（Introduction）:允许我们向现有的类添加新方法或属性。
- 织入（Weaving）:织入是把切面应用到目标对象并创建新的代理对象的过程。



##  二、声明式事物源码解读

由于采用声明式@Transactional这种注解的方式，那么我们从SpringBoot 启动时的自动配置载入开始看。在SpringBoot autoconfigure包下的/META-INF/spring.factories中配置文件中查找到有关声明式事物的自动配置，如下图：

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%887.37.13.png)

发现，SpringBoot载入2个关于事务的自动配置类： 

**org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration**,
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,

重点看一下`TransactionAutoConfiguration`这个自动配置类。

### 2.1  TransactionAutoConfiguration

TransactionAutoConfiguration这个类主要看：两个类注解 和 两个内部类

#### 2.1.1 两个类注解

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%887.52.08.png)

* `@ConditionalOnClass(PlatformTransactionManager.class)` 会在类路径下包含PlatformTransactionManager这个类时这个自动配置生效，这个类是Spring事务的核心包，肯定引入了。

* `@AutoConfigureAfter({ JtaAutoConfiguration.class, HibernateJpaAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class, Neo4jDataAutoConfiguration.class })`，这个配置在括号中的4个配置类后才生效。

#### 2.1.2 两个类内部类

**TransactionTemplateConfiguration 事务模板配置类**

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%887.55.00.png)

有两个注解，含义分别如下：

* `@ConditionalOnSingleCandidate(PlatformTransactionManager.class)`  当能够唯一确定一个PlatformTransactionManager Bean实例时才生效。

* `@ConditionalOnMissingBean`  如果没有定义TransactionTemplate Bean生成一个。

**EnableTransactionManagementConfiguration  开启事务管理器配置类**

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%887.58.46.png)

在源码中我们还可以看到，EnableTransactionManagementConfiguration支持2种代理方式：**JdkDynamicAutoProxyConfiguration** 和 **CglibAutoProxyConfiguration**

- **JdkDynamicAutoProxyConfiguration**

  `@EnableTransactionManagement(proxyTargetClass = false)`，表示是JDK动态代理，支持面向接口的代理。

  `@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)`，即`spring.aop.proxy-target-class=false`时生效，且没有这个配置不生效。



- **CglibAutoProxyConfiguration**

  `@EnableTransactionManagement(proxyTargetClass = true)`，表示使用Cglib代理，支持的是子类继承代理。
  `@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)`，即`spring.aop.proxy-target-class=true`时生效，且没有这个配置默认生效。**也就是说，如果开始事物后，没有做任何配置，那么SpringBoot会默认使用Cglib代理，那么也就是说，@Transactional注解默认情况下支持直接加在类上。**

看到这里，我们需要对`@EnableTransactionManagement`注解做一个更深的认识，下图所示是其源码。

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-05%20%E4%B8%8A%E5%8D%889.11.22.png)

* **proxyTargetClass** ：值为false表示是JDK动态代理支持接口代理。true表示是Cglib代理支持子类继承代理。
* **mode**：事务通知模式(切面织入方式)，默认代理模式（同一个类中方法互相调用拦截器不会生效），可以选择增强型AspectJ
* **order**：连接点上有多个通知时用于优先级排序，默认最低。值越大优先级越低。

属性先说这么多，这个注解重点还是看它头上的`@Import(TransactionManagementConfigurationSelector.class)` 这个注解。@Import注解的功能不用多说，它就是用来导入配置的，重点关注配置类**TransactionManagementConfigurationSelector**，类图如下：

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-05%20%E4%B8%8A%E5%8D%889.58.18.png)

如上图所示，TransactionManagementConfigurationSelector继承自AdviceModeImportSelector实现了ImportSelector接口。

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-05%20%E4%B8%8A%E5%8D%8810.00.22.png)

如上图，最终会执行selectImports方法导入需要加载的类，我们只看**PROXY**模式下，载入了AutoProxyRegistrar、ProxyTransactionManagementConfiguration2个类。

- **AutoProxyRegistrar**：给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；
- **ProxyTransactionManagementConfiguration**：一个配置类，定义了事务增强器。









## 三、 声明式事物工作时序图

最后，整理了一下声明式事物工作时序图，建议收藏！

![](https://image.easyblog.top/584866-20180925191041500-2119850677.jpg)
