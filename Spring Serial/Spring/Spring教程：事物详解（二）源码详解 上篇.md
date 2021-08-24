## Spring教程：事物详解（二）源码详解



## 引言

Spring提供了两种事物管理实现方式：

1. **编程式事务管理：** 编程式事务管理使用**TransactionTemplate**可实现更细粒度的事务控制。

2. **声明式事务管理：** 基于**Spring AOP**实现。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

   声明式事务管理没有入侵代码，通过**@Transactional**就可以进行事务操作，更快捷而且简单（尤其是配合Spring Boot自动配置，可以说是精简至极！）且大部分业务都可以满足，推荐使用。

其实不管是编程式事务还是声明式事务，最终调用的底层核心代码是一致的。本章分上下两篇，分别从编程式、声明式入手，再进入核心源码贯穿式讲解。本片重点分析编程式事物的实现方式。Let's  go~~



## 一、编程式事务 TransactionTemplate

全路径名是：`org.springframework.transaction.support.TransactionTemplate`。看包名也知道了，这是Spring对事务的模板类。在Spring中模版模式使用非常广泛的～

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%885.56.19.png)

点进TransactionTemplate源码它是实现了`TransactionOperations、InitializingBean`这2个接口（熟悉Spring源码的小伙伴知道这个InitializingBean又是老套路），我们来看下接口源码如下：

**TransactionOperations接口**

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%886.53.09.png)

**InitializingBean**

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%886.51.09.png)

TransactionOperations这个接口用来执行事务的回调方法，InitializingBean这个是典型的Spring Bean初始化流程中的预留接口，专用用来在bean属性加载完毕时执行的方法（不熟悉的小伙伴去看看Spring Bean生命周期的流程）。



看到这里，有的小伙伴自然而然就有疑问了：TransactionTemplate的2个接口的方法做了什么？接着往下说！

![](https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%886.55.56.png)

如上图所示，实际上afterPropertiesSet方法只是校验了事务管理器不为空，execute()才是核心方法，execute主要步骤：

<img src="https://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%887.17.14.png" style="zoom:45%;" />

总结一下，核心步骤如下：

* **getTransaction()获取事务**，源码见 见下篇3.3.1节

* **doInTransaction()执行业务逻辑**，这里就是用户自定义的业务代码。如果是没有返回值的，就是doInTransactionWithoutResult()。

* **commit()事务提交**，根据事物执行状态，调用**AbstractPlatformTransactionManager**类的`commit() 提交事物`、`rollbackOnException() 异常回滚`或者`rollback() 事务提交回滚`，源码见下篇3.3.3节

