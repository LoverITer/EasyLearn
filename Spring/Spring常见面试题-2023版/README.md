# Spring 常见面试题-2023版



1.谈谈你使用过哪些设计模式
2.mybatis多级缓存是如何设计的
3.谈谈springmvc整体请求流程
4.谈谈SpringBean的生命周期
5.Spring底层如何解决循环问题
6.Spring Aop有哪些应用场景
7.Spring Aop底层是如何实现
8.Spring框架中用到了哪些设计模式
9.Spring 事务底层是如何实现的
10.使用Spring 事务有哪些注意事项
11.Spring 事务传播行为有哪些
12.Spring Boot有哪些注解
13.Spring Boot的原理
14.Spring Boot自动装配的原理  



spring中为什么会设计一个bean生命周期

目的 能够实现扩展性

# Applicationcontext与BeanFactory区别

applicationcontext对BeanFactory实现额外增强

1.applicationcontext主动添加Bean（工厂）处理器  来完成扫包

2.applicationcontext主动添加bean处理器 ----依赖注入@Autowired

3.applicationcontext主动初始化单例对象

BeanFactory----bean定义信息、单例池

# Bean工厂处理器与Bean后置处理器之间有哪些区别

1.Bean工厂处理器是在实例化Bean之前执行，例如 Spring中ConfigurationClassPostProcessor 解析 componentScan ， Bean ， Import ，ImportResource 注册Bean信息，可以通过工厂Bean处理器修改Bean的定义信息。

2.Bean后置处理器 是在 init前后执行 ，可以修改Bean对象为代理对象 从而对我们方法实现增强。

# 谈谈SpringBean的生命周期

为什么在Spring中设计 Bean生命周期?

1.Spring中的bean的生命周期主要包含四个阶段：实例化Bean --＞ Bean属性填充 --＞ 初始化Bean --＞销毁Bean

![img](https://cdn.nlark.com/yuque/0/2022/png/25438525/1660992033497-3dffed45-d4e6-490b-b959-bfb3aef8011d.png)





()

------

1.bean会被反射实例化（ABean），执行构造方法（默认执行无参构造方法）

ABean依赖BBean

2.走bean处理器 完成，bean属性填充（依赖----设计模式 循环依赖的问题）

3.执行bean处理器 初始化方法之前  执行

4.执行 初始化方法(init @PostConstruct )

5.执行bean处理器 初始化方法之后 执行  原生bean 代理bean

6.bean存入ioc容器

为什么spring bean生命周期设计bean后置处理呢？ 能够实现扩展功能

\-----



2.首先是实例化Bean，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚末初始化的依赖时，容器就会调用doCreateBean()方法进行实例化，实际上就是通过反射的方式创建出一个bean对象

3.Bean实例创建出来后，接着就是给这个Bean对象进行属性填充，也就是注入这个Bean依赖的其它bean对象

属性填充完成后，进行初始化Bean操作，初始化阶段又可以分为几个步骤：

 3.1执行Aware接口的方法

 3.2.Spring会检测该对象是否实现了xxxAware接口，通过Aware类型的接口，可以让我们拿到Spring容器的些资源。如实现

 3.3.BeanNameAware接口可以获取到BeanName，实现BeanFactoryAware接口可以获取到工厂对象BeanFactory等

3.4.执行BeanPostProcessor的前置处理方法postProcessBeforelnitialization()，对Bean进行一些自定义的前置处理

3.5.判断Bean是否实现了InitializingBean接口，如果实现了，将会执行lnitializingBean的afeterPropertiesSet()初始化方法；

3.6.执行用户自定义的初始化方法，如init-method等；

3.7.执行BeanPostProcessor的后置处理方法postProcessAfterinitialization()

初始化完成后，Bean就成功创建了，之后就可以使用这个Bean， 当Bean不再需要时，会进行销毁操作，

3.8首先判断Bean是否实现了DestructionAwareBeanPostProcessor接口，如果实现了，则会执行DestructionAwareBeanPostProcessor后置处理器的销毁回调方法

3.8其次会判断Bean是否实现了DisposableBean接口，如果实现了将会调用其实现的destroy()方法

3.9最后判断这个Bean是否配置了dlestroy-method等自定义的销毁方法，如果有的话，则会自动调用其配置的销毁方法；

# Spring底层如何解决循环问题

1.Spring一级缓存作用

第一级缓存〈也叫单例池）singletonObjects：存放已经经历了完整生命周期的Bean对象 成员属性都是有值的

2.Spring二级缓存作用

第二级缓存 earlySingletonObjects：存放早期暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完整）

3.Spring三级缓存作用

第三级缓存 Map<String, ObiectFactory<?>> singletonFactories：存放可以生成Bean的工厂

1.先执行 AService的构造方法----AService实例化 F8 一步一步分析代码如何走的

2.在为我们AService属性填充之前，会将我们的该对象 封装成ObjectFactory

存入到三级缓存中。

```java
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

调用ObjectFactory.getObject()..执行Lambda  执行该方法：getEarlyBeanReference

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```

3.为我们的AService中成员属性赋值，实例化BService 会执行到BService 无参构造方法

​     3.1为我们的BService 中成员属性赋值，AService （单例池中查找getBean（“AService ”））

​     3.2 调用getBean（“AService ”）获取  AService 对象赋值给我们的BService 依赖于AService属性。

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    ### 根据beanName 从单例池中获取bean对象
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
                ###从二级缓存中获取bean对象
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
                    // 从三级缓存中获取singletonFactory
					ObjectFactory<?> singletonFactory = this.singletonFactories
                        .get(beanName);
					if (singletonFactory != null) {
                        ##执行三级缓存对应的..执行Lambda  执行该方法：
                            getEarlyBeanReference
                        ## 判断如果开启了aop 则会生成代理类 AService代理类
                        ## 将我们的AService代理类 赋值给我们BService
                        ## 如果没有开启aop的情况下 则直接返回原始不完整bean对象
						singletonObject = singletonFactory.getObject();
                        ###将该代理类存入到二级缓存中 存入不完整对象
						this.earlySingletonObjects.put(beanName, singletonObject);
						###在删除三级缓存
                        this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

​    3.3 	singletonObject = singletonFactory.getObject(); 执行

```java
	protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```





如果我们开启了aop的情况下 则返回aop代理对象

```java
	@Override
	public Object getEarlyBeanReference(Object bean, String beanName) {
		Object cacheKey = getCacheKey(bean.getClass(), beanName);
		this.earlyProxyReferences.put(cacheKey, bean);
		return wrapIfNecessary(bean, beanName, cacheKey);
	}
```

如果我们没有开启aop的情况下 则返回原始对象

```java
@Override
	public Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
		return bean;
	}
```



A依赖B

B依赖A-----a对象存入二级缓存中不完整bean对象

B依赖C

C依赖A  ----getBean("a")  直接从二级缓存中查找到的。

```java
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
```



1.当我们正在创建A对象时，执行A类中无参构造方法

2.在执行属性赋值之前会先保存一个Lambda表达式在三级缓存中，key 为beanName value为该Lambda表达式

```java
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

3.在开始执行属性的赋值操作，A对象依赖B对象，会执行B对象的无参构造方法

4.B对象开始执行属性赋值，发现 B对象依赖A对象，但是A对象没有创建成功。

5.则会调用到（getSingleton）：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
         //  先从一级缓存查询该对象
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
                // 一级缓存如果没有该对象则 从二级缓存查找该对象
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
                    // 二级缓存如果没有该对象 则通过三级缓存中 保存的Lambda 执行 获取Bean对象
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
                        //则通过三级缓存中 保存的Lambda 执行 获取Bean对象
						singletonObject = singletonFactory.getObject();
                        //在保存到二级缓存中
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

singletonObject = singletonFactory.getObject();---重点代码：

6.当我们在调用singletonFactory.getObject(); 本质上在执行三级缓存中保存的

Lambda表达式。

```java
	protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```

![img](https://cdn.nlark.com/yuque/0/2022/png/25438525/1662293221064-dd2fd4f0-6820-4c87-a105-4851a20c12da.png)

如果我们使用了aop的情况下 ，则会基于aop生成代理类  判断我们的被代理类是否有

实现接口，如果有实现接口则采用jdk动态代理 如果没有实现接口则采用cglib代理。

如果我们没有使用aop的情况下，则返回的就是原始对象。

7.将singletonFactory.getObject(); 返回对象存入到二级缓存中 （代理对象、原始对象）

8.....继续执行生命周期后续流程。

# SpringBean配置方式有哪些

1.基于 xml 配置

2.java api方式

3.注解的的方式

# Spring你使用过哪些注解

@Controller - 用于 Spring MVC 项目中的控制器类。

@Service - 用于服务类。

@RequestMapping - 用于在控制器处理程序方法中配置 URI 映射。

@ResponseBody - 用于发送 Object 作为响应，通常用于发送 XML 或 JSON 数据作为响应。

@PathVariable - 用于将动态值从 URI 映射到处理程序方法参数。

@Autowired - 用于在 spring bean 中自动装配依赖项。

@Qualifier - 使用 @Autowired 注解，以避免在存在多个 bean 类型实例时出现混淆。

@Scope - 用于配置 spring bean 的范围。

@Configuration，@ComponentScan 和 @Bean - 用于基于 java 的配置。

@Aspect，@Before，@After，@Around，@Pointcut - 用于切面编程（AOP）。

# Spring Aop有哪些作用

可以解决我们代码重复性问题，对我们代码实现额外功能增强。

前置 后置 环绕 异常 等通知

aop 底层 通知执行 调用链---（代理、责任链（递归））

# Spring Aop有哪些应用场景

\1. 日志的采集

\2. 权限控制 aop 拦截器

\3. Mybatis mapper

\5. Spring的事务

\6. 全局捕获异常

\7. Rpc远程调用接口 （传递就是接口）

\8. 代理数据源

9.自定义注解

# Spring Aop底层是如何实现

底层基于动态代理模式实现，如果我们的目标类有实现接口的情况下则采用JDK动态代理，如果我们

的目标类没有实现接口的情况下则采用cglib动态代理。

# Spring框架中用到了哪些设计模式

1.工厂设计模式：Spring使用工厂模式通过BeanFactory和ApplicationContext创建bean对象。

2.代理设计模式：Spring AOP功能的实现。

3.单例设计模式：Spring中的bean默认都是单例的。

4.模板方法模式：Spring中的jdbcTemplate、hibernateTemplate等以Template结尾的对数据库操作的类，它们就使用到了模板模式。

5.包装器设计模式：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

6.观察者模式：Spring事件驱动模型就是观察者模式很经典的一个应用。

7.适配器模式：Spring AOP的增强或通知（Advice）使用到了适配器模式、Spring MVC中也是用到了适配器模式适配Controller。

# Spring 事务底层是如何实现的

Spring中的事务由声明事务与编程事务组成

声明事务：使用注解或者扫包方式配置事务 减少代码冗余性 代理模式或者aop实现

编程事务：手动begin、commit/rollback操作 代码重复

基于aop环绕通知或者bean生命周期处理器可以实现

# 使用Spring 事务有哪些注意事项

如果使用事务注解，方法中有异常往上抛 不要try，否则 外层aop没有

拦截到异常 导致事务注解会失效，如果非要try的情况下 可以采用手动事务处理。

 TransactionAspectSupport.

​        currentTransactionStatus()

​        .setRollbackOnly(); // 手动回滚

```java
@GetMapping("/updateUser2")
    @Transactional
    public String updateUser2(String name) {
    try {
    int result = userMapper.updateUser(name);
    if ("mayikt".equals(name)) {
        // 报错
        int j = 1 / 0;
    }
    return result > 0 ? "ok" : "fail";
} catch (Exception e) {
    log.error("e:{}", e);
    TransactionAspectSupport.
        currentTransactionStatus()
        .setRollbackOnly(); // 手动回滚
    return "fail";
}

}
```

# Spring 事务传播行为有哪些

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

1.TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。

2.TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前线程存在事务，则把当前事务挂起。

3.TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

4.TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。

5.TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

6.TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

7.TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。



# Spring MVC请求流程

1、用户向服务器发送请求，请求会先达到 SpringMVC 前端控制器 DispatcherServlet ;

2、DispatcherServlet 根据该 URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以 HandlerExecutionChain 执行链对象的形式返回

3、DispatcherServlet 根据获得的 Handler，获取对应的 HandlerAdapter;

5、获取到 HandlerAdapter，将开始执行拦截器的 preHandler(…)方法;

6、提取 Request 中的模型数据，填充 Handler 入参，开始执行 Handler（Controller)方法，处理请求，在填充 Handler 的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：

HttpMessageConveter：将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的类型信息

数据转换：对请求消息进行数据转换。如 String 转换成 Integer、Double 等

数据格式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等

数据验证：验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中

7、Handler 方法执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象。

8、开始执行拦截器的 postHandle(...)方

9、根据返回的 ModelAndView（此时会判断是否存在异常：如果存在异常，则执行 HandlerExceptionResolver 进行异常处理）选择一个适合的 ViewResolver 进行视图解析，根据 Model 和 View，来渲染视图

10、渲染视图完毕执行拦截器的 afterCompletion(…)方法

11、将渲染结果返回给客户端

# Spring Boot有哪些注解

1.@ SpringBootApplication

2.@ImportAutoConfiguration

3.@SpringBootConfiguration @Configuration

4.@EnableAutoConfiguration 实现自动装配



# Spring Boot的原理

\1. 封装了Maven常用依赖：能够快速的整合第三方框架

\2. 内置Tomcat(java)

\3. Web组件默认整合SpringMVC框架

\4. 使用注解化方式简化xml spring、springmvc（去除web.xml/xml）

# Spring Boot自动装配的原理

1.自动装配其实就是自动的把第三方的组件（框架）bean，装载到IOC容器中，不需要开发人员额外的去注入第三方bean相关配置，在springboot中启动类上加上@SpringBootApplication即可实现自动装配。

2.@SpringBootApplication组合注解，底层是@Enableautoconfiguration注解，通过引入第三方自定义的springboot starter 插件 中会有一个 加上了@configuration配置类，通过该配置类声明bean的信息，在通过springboot中约定配置思想，将该配置类的全路径放在 jar中的META-INF\spring.factories，这样的话 springboot就可以知道 第三方jar包对应的配置类位置，从而将第三方jar包配置类中的bean信息 加入到ioc容器中，底层会使用到spring.factoriesLoader完成。

引入第三方jar包的 不需要开发者额外配置组件jar包中bean信息 （自动实现装配）

# 谈谈Mybatis#与$区别

\#{参数} 预编译 防止sql注入问题

$ 被利用sql注入攻击问题

# 谈谈Mybatis分页原理

通过mybatis 拦截器改写sql语句 

sql语句后面加上  limit (0,5)

# 谈谈Mybatis一级和二级缓存



1.Mybatis 中有一级缓存和二级缓存，采用装饰设计模式；

2.默认情况下一级缓存是开启的，而且是不能关闭的 ，一级缓存是指 SqlSession 级别的缓存，当在同一个 SqlSession 中进行相同的 SQL 语句查询时，第二次以后的查询不会从数据库查询，而是直接从缓存中获取，一级缓存最多缓存 1024 条 SQL。

3.二级缓存是指可以跨 SqlSession 的缓存。 是 mapper 级别的缓存，对于 mapper 级别的缓存不同的sqlsession 是可以共享的，需要额外整合到第三方缓存 例如Redis、MongoDB、oscache、ehcache等。

一级、二级缓存 采用装饰模式设计封装。



# mybatis一级缓存优缺点

\1. Mybatis的一级缓存存放在SqlSession的生命周期，在同一个SqlSession中查询时，Mybatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。

如果同一个SqlSession中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map缓存对象中已经存在改键值时，则会返回缓存中的对象。(一个SqlSession连续两次查询 得到的是同一个java对象)

任何的insert update delete操作都会清空一级缓存(增删改任何记录都会清空当前SqlSession的缓存)。

2.如果服务器集群的时候，每个sqlSession有自己独立的缓存相互之间不存在共享，所以在服务器集群的时候容易产生查询数据冲突问题。



# 如何禁止使用mybatis一级缓存

方案1  在sql语句上 随机生成 不同的参数 存在缺点：map集合可能爆 内存溢出的问题

方案2  开启二级缓存（共享 依赖Redis实现）

方案3  使用sqlSession强制清除缓存

方案4  创建新的sqlSession连接。



# Spring整合Mybatis的时候一级缓存的问题

1.在未开启事务的情况之下，每次查询spring都会关闭旧的sqlSession而创建新的sqlSession,因此此时的一级缓存是没有启作用的

2.在开启事务的情况之下，spring模板使用threadLocal获取当前资源绑定同一个sqlSession，因此此时一级缓存是有效的



# 你会哪些设计模式



代码 设计模式 重构 设计模式提高代码扩展性

第八期结业项目聚合支付平台

常见设计模式：

1.策略 应用场景：减少if判断 例如聚合支付、联合登录（微信/QQ/钉钉）、（用户选择某一种）发送渠道通知(钉钉、短信、公众号、邮件)

2.工厂 应用场景：创建对象通过工厂获取到 不需要开发者自己new 例如SpringIOC bean工厂 从工厂获取

3.代理 应用场景：代理模式应用场景（减少代码冗余性问题）

例如每个方法 相同事情 （输出方法参数日志）

4.装饰器（mybatis一级、二级缓存）

mybatis一级、二级缓存

 jvm内置缓存（一级缓存）redis  （二级缓存） mysql  （三级缓存）

多级缓存架构模式考虑什么问题？数据一致性问题

不改变原有代码基础之上 额外实现增强。

5.适配器模式  springmvc请求控制

6.观察者模式 群发发送通知 （钉钉、短信、公众号、邮件）

7.单例 枚举单例是最安全 理论无法破解 但是修改jdk源码



**策略模式、模板方法模式、观察者模式、责任链模式**、**适配器模式、装饰器模式、代理模式（jdk与cglib）、外观模式、工厂方法模式、抽象工厂模式、单例模式、建造者模式、单例模式**

# 代理模式应用场景

\1. 日志的采集

\2. 权限控制

\3. 实现aop

\4. Mybatis mapper

\5. Spring的事务

\6. 全局捕获异常

\7. Rpc远程调用接口 （传递就是接口）

\8. 代理数据源

9.自定义注解



# 静态代理与动态代理区别

静态代理开发者自己写代理类 动态代理 不需要开发自己写代码

# 动态代理有哪些

cglib和jdk动态代理

jdk动态代理基于接口方式代理

cglib动态代理基于继承方式代理