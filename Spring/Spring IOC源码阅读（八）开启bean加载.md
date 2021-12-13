## Spring IOC源码阅读（八）开启bean加载



![](https://www.cmsblogs.com/images/group/sike-java/sike-java-spring-ioc/202105092051549271.png)

（此图来自《Spring 揭秘》) Spring IOC 容器所起的作用如上图所示，它会以某种方式加载 Configuration Metadata，将其解析注册到容器内部，然后会根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。 Spring 在实现上述功能中，将整个流程分为两个阶段：容器初始化阶段和加载 bean 阶段。

- **容器初始化阶段**：首先通过某种方式加载 Configuration Metadata (主要是依据 Resource、ResourceLoader 两个体系)，然后容器会对加载的 Configuration MetaData 进行解析和分析，并将分析的信息组装成 BeanDefinition，并将其保存注册到相应的 BeanDefinitionRegistry 中。至此，Spring IOC 的初始化工作完成。
- **加载 bean 阶段**：经过容器初始化阶段后，应用程序中定义的 bean 信息已经全部加载到系统中了，当我们显示或者隐式地调用 `getBean()` 时，则会触发加载 bean 阶段。在这阶段，容器会首先检查所请求的对象是否已经初始化完成了，如果没有，则会根据注册的 bean 信息实例化请求的对象，并为其注册依赖，然后将其返回给请求方。至此第二个阶段也已经完成。

第一个阶段前面（1~7）已经用了大量篇幅进行了深入分析了。所以从这篇开始分析第二个阶段：**加载 bean 阶段**。当我们显示或者隐式地调用 `BeanFactory.getBean()` 时，则会触发加载 bean 阶段。如下：

![](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%886.31.12.png)

方法内部调用 BeanFactory的抽象实现类 AbstractBeanFactory 的 `doGetBean()` 方法，其接受四个参数：

- `name`：要获取 bean 的名字
- `requiredType`：要获取 bean 的类型
- `args`：创建 bean 时传递的参数。这个参数仅限于创建 bean 时使用
- `typeCheckOnly`：是否为类型检查

`doGetBean` 实现如下：

```java
protected <T> T doGetBean(                                                                                      
		String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)           
		throws BeansException {                                                                                 
  // 获取 beanName，这里是一个转换动作，将 name 转换Wie beanName                                                                                                              
	String beanName = transformedBeanName(name);                                                                
	Object beanInstance;                                                                                        
                                                                                                                
	// 从缓存中或者实例工厂中获取 bean
  // *** 这里会涉及到解决循环依赖 bean 的问题                                        
	Object sharedInstance = getSingleton(beanName);                                                             
	if (sharedInstance != null && args == null) {                                                               
		if (logger.isTraceEnabled()) {                                                                          
			if (isSingletonCurrentlyInCreation(beanName)) {                                                     
				logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +               
						"' that is not fully initialized yet - a consequence of a circular reference");         
			}                                                                                                   
			else {                                                                                              
				logger.trace("Returning cached instance of singleton bean '" + beanName + "'");                 
			}                                                                                                   
		}                                                                                                       
		beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);                          
	}	else {                                                                                                      
		// 因为 Spring 只解决单例模式下得循环依赖，在原型模式下如果存在循环依赖则会抛出异常
    // **关于循环依赖后续会单独出文详细说明**                                                        
		if (isPrototypeCurrentlyInCreation(beanName)) {                                                         
			throw new BeanCurrentlyInCreationException(beanName);                                               
		}                                                                                                       
                                                                                                                
		// 如果容器中没有找到，则从父类容器中加载                                                  
		BeanFactory parentBeanFactory = getParentBeanFactory();                                                 
		if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {                                   
			// Not found -> check parent.                                                                       
			String nameToLookup = originalBeanName(name);                                                       
			if (parentBeanFactory instanceof AbstractBeanFactory) {                                             
				return ((AbstractBeanFactory) parentBeanFactory).doGetBean(                                     
						nameToLookup, requiredType, args, typeCheckOnly);                                       
			} else if (args != null) {                                                                            
				// Delegation to parent with explicit args.                                                     
				return (T) parentBeanFactory.getBean(nameToLookup, args);                                       
			} else if (requiredType != null) {                                                                    
				// No args -> delegate to standard getBean method.                                              
				return parentBeanFactory.getBean(nameToLookup, requiredType);                                   
			} else {                                                                                              
				return (T) parentBeanFactory.getBean(nameToLookup);                                             
			}                                                                                                   
		}                                                                                                       
                                                                                                                
		if (!typeCheckOnly) {                                                                                   
			markBeanAsCreated(beanName);                                                                        
		}                                                                                                       
                                                                                                                
		StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")                    
				.tag("beanName", name);                                                                         
		try {                                                                                                   
			if (requiredType != null) {                                                                         
				beanCreation.tag("beanType", requiredType::toString);                                           
			} 
      // 从容器中获取 beanName 相应的 GenericBeanDefinition，并将其转换为 RootBeanDefinition
			RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);     
      // 检查给定的合并的 BeanDefinition
			checkMergedBeanDefinition(mbd, beanName, args);                                                     
                                                                                                                
			// 处理所依赖的 bean                             
			String[] dependsOn = mbd.getDependsOn();                                                            
			if (dependsOn != null) {                                                                            
				for (String dep : dependsOn) {  
          // 若给定的依赖 bean 已经注册为依赖给定的bean
					if (isDependent(beanName, dep)) {                                                           
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,                 
								"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'")
					}
          // 缓存依赖调用
					registerDependentBean(dep, beanName);                                                       
					try {                                                                                       
						getBean(dep);                                                                           
					} catch (NoSuchBeanDefinitionException ex) {                                                  
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,                 
								"'" + beanName + "' depends on missing bean '" + dep + "'", ex);                
					}                                                                                           
				}                                                                                               
			}                                                                                                   
                                                                                                                
			// bean 实例化
      // 单例模式                                                                         
			if (mbd.isSingleton()) {                                                                            
				sharedInstance = getSingleton(beanName, () -> {                                                 
					try { 
            //实例化bean
						return createBean(beanName, mbd, args);                                                 
					} catch (BeansException ex) {                                                                               
						destroySingleton(beanName);                                                             
						throw ex;                                                                               
					}                                                                                           
				});                                                                                             
				beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);                   
			}	else if (mbd.isPrototype()) {                                                                       
				//bean实例化，原型模式                                                  
				Object prototypeInstance = null;                                                                
				try {                                                                                           
					beforePrototypeCreation(beanName);                                                          
					prototypeInstance = createBean(beanName, mbd, args);                                        
				} finally {                                                                                       
					afterPrototypeCreation(beanName);                                                           
				}                                                                                               
				beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);                
			} else {      
        //bean实例化： 从指定的 scope 下创建 bean
				String scopeName = mbd.getScope();                                                              
				if (!StringUtils.hasLength(scopeName)) {                                                        
					throw new IllegalStateException("No scope name defined for bean '" + beanName + "'");       
				}                                                                                               
				Scope scope = this.scopes.get(scopeName);                                                       
				if (scope == null) {                                                                            
					throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");  
				}                                                                                               
				try {                                                                                           
					Object scopedInstance = scope.get(beanName, () -> {                                         
						beforePrototypeCreation(beanName);                                                      
						try {                                                                                   
							return createBean(beanName, mbd, args);                                             
						}                                                                                       
						finally {                                                                               
							afterPrototypeCreation(beanName);                                                   
						}                                                                                       
					});                                                                                         
					beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);               
				}                                                                                               
				catch (IllegalStateException ex) {                                                              
					throw new ScopeNotActiveException(beanName, scopeName, ex);                                 
				}                                                                                               
			}                                                                                                   
		}                                                                                                       
		catch (BeansException ex) {                                                                             
			beanCreation.tag("exception", ex.getClass().toString());                                            
			beanCreation.tag("message", String.valueOf(ex.getMessage()));                                       
			cleanupAfterBeanCreationFailure(beanName);                                                          
			throw ex;                                                                                           
		}                                                                                                       
		finally {                                                                                               
			beanCreation.end();                                                                                 
		}                                                                                                       
	}                                                                                                           
  // 检查需要的类型是否符合 bean 的实际类型，如果类型符合就返回示实例                                                                                                              
	return adaptBeanInstance(name, beanInstance, requiredType);                                                 
}                                                                                                               
```

代码是相当的长，处理逻辑也是相当复杂，下面将其进行拆分阐述。

####  **1.获取 beanName**

![截屏2021-12-13 下午6.54.33](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%886.54.33.png)

这里调用`doGetBean()`的时候传递的参数 name，不一定就是 beanName，可能是 aliasName，也有可能是 FactoryBean，所以这里需要调用 `transformedBeanName()` 方法对 name 进行一番转换，主要如下：

```java
//doGetBean()方法里直接调用的transformedBeanName方法
protected String transformedBeanName(String name) {                  
	return canonicalName(BeanFactoryUtils.transformedBeanName(name));
}

//AbstractBeanFactory 类中 transformedBeanName 调用 BeanFactoryUtils 的 transformedBeanName方法
// 去除 FactoryBean 的修饰符
public static String transformedBeanName(String name) {                             
	Assert.notNull(name, "'name' must not be null");                                
	if (!name.startsWith(BeanFactory.FACTORY_BEAN_PREFIX)) { 
    //没有 ”&“ 前缀的name直接返回
		return name;                                                                
	}                                                                               
	return transformedBeanNameCache.computeIfAbsent(name, beanName -> {             
		do { 
      //处理，将name的"&"前缀去掉
			beanName = beanName.substring(BeanFactory.FACTORY_BEAN_PREFIX.length());
		}                                                                           
		while (beanName.startsWith(BeanFactory.FACTORY_BEAN_PREFIX));               
		return beanName;                                                            
	});                                                                             
} 

//AbstractBeanFactory 类中 transformedBeanName 调用的 canonicalName方法
//转换aliasName
public String canonicalName(String name) {               
	String canonicalName = name;                                                         
	String resolvedName;                                 
	do { 
    //循环地从bean别名Map中根据上一次的bean的别名获取对应的别名
		resolvedName = this.aliasMap.get(canonicalName); 
		if (resolvedName != null) { 
      //获取到的别名不为空，更新别名，方便下一次循环或者取值
			canonicalName = resolvedName;                
		}                                                
	}	while (resolvedName != null);                  
  //返回最终的beanName
	return canonicalName;                                
}                                                        
```

主要处理过程包括两步：

1. 去除 FactoryBean 的修饰符。如果 name 以 “&” 为前缀，那么会去掉该 "&"，例如，`name = "&studentService"`，则会是 `name = "studentService"`。
2. 取指定的 alias 所表示的最终 beanName。主要是一个循环获取 beanName 的过程，例如别名 A 指向名称为 B 的 bean 则返回 B，若别名 A 指向别名 B，别名 B 指向名称为 C 的 bean，则返回 C。

#### **2.从单例 bean 缓存中获取 bean**

![](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%887.08.10.png)

我们知道单例模式的 bean 在整个过程中只会被创建一次，第一次创建后会将该 bean 加载到缓存中，后面再获取 bean 就会直接从单例缓存中获取。如果从缓存中得到了 bean，则需要调用 `getObjectForBeanInstance()` 对 bean 进行实例化处理，因为缓存中记录的是最原始的 bean 状态，我们得到的不一定是我们最终想要的 bean。

#### **3.原型模式依赖检查与 parentBeanFactory**

![](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%887.12.00.png)

Spring 只处理单例模式下得循环依赖，对于原型模式下如果出现循环依赖则会直接抛出异常。主要原因还是在于 Spring 解决循环依赖的策略有关。对于单例模式 Spring 在创建 bean 的时候并不是等 bean 完全创建完成后才会将 bean 添加至缓存中，而是不等 bean 创建完成就会将创建 bean 的 ObjectFactory 提早加入到缓存中，这样一旦下一个 bean 创建的时候需要依赖 bean 时则直接使用 ObjectFactroy。但是原型模式我们知道是没法使用缓存的，所以 Spring 对原型模式的循环依赖处理策略则是不处理（关于循环依赖后面会有单独文章说明）。 如果容器缓存中没有相对应的 BeanDefinition 则会尝试从父类工厂（parentBeanFactory）中加载，然后再去递归调用 `getBean()`。

#### **4. 依赖处理**

![](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%887.15.23.png)

每个 bean 都不是单独工作的，它会依赖其他 bean，其他 bean 也会依赖它，对于依赖的 bean ，它会优先加载，所以在 Spring 的加载顺序中，在初始化某一个 bean 的时候首先会初始化这个 bean 的依赖。

#### **5.作用域处理**

Spring  bean 的作用域默认为 `singleton`，当然还有其他作用域，如prototype、request、session 等，不同的作用域会有不同的初始化策略。源码有点长，这里就不截图啦，相应代码段如下：

```java
      // bean 实例化
      // 单例模式                                                                         
			if (mbd.isSingleton()) {                                                                            
				sharedInstance = getSingleton(beanName, () -> {                                                 
					try { 
            //实例化bean
						return createBean(beanName, mbd, args);                                                 
					} catch (BeansException ex) {                                                                               
						destroySingleton(beanName);                                                             
						throw ex;                                                                               
					}                                                                                           
				});                                                                                             
				beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);                   
			}	else if (mbd.isPrototype()) {                                                                       
				//bean实例化，原型模式                                                  
				Object prototypeInstance = null;                                                                
				try {                                                                                           
					beforePrototypeCreation(beanName);                                                          
					prototypeInstance = createBean(beanName, mbd, args);                                        
				} finally {                                                                                       
					afterPrototypeCreation(beanName);                                                           
				}                                                                                               
				beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);                
			} else {      
        //bean实例化： 从指定的 scope 下创建 bean
				String scopeName = mbd.getScope();                                                              
				if (!StringUtils.hasLength(scopeName)) {                                                        
					throw new IllegalStateException("No scope name defined for bean '" + beanName + "'");       
				}                                                                                               
				Scope scope = this.scopes.get(scopeName);                                                       
				if (scope == null) {                                                                            
					throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");  
				}                                                                                               
				try {                                                                                           
					Object scopedInstance = scope.get(beanName, () -> {                                         
						beforePrototypeCreation(beanName);                                                      
						try {                                                                                   
							return createBean(beanName, mbd, args);                                             
						}                                                                                       
						finally {                                                                               
							afterPrototypeCreation(beanName);                                                   
						}                                                                                       
					});                                                                                         
					beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);               
				}                                                                                               
				catch (IllegalStateException ex) {                                                              
					throw new ScopeNotActiveException(beanName, scopeName, ex);                                 
				}                                                                                               
			}                                                                                                   
		}                                                                                                       
		catch (BeansException ex) {                                                                             
			beanCreation.tag("exception", ex.getClass().toString());                                            
			beanCreation.tag("message", String.valueOf(ex.getMessage()));                                       
			cleanupAfterBeanCreationFailure(beanName);                                                          
			throw ex;                                                                                           
		}                                                                                                       
		finally {                                                                                               
			beanCreation.end();                                                                                 
		}                                                                                                       
	}                     
```



####  **6.类型转换**

![](img/%E6%88%AA%E5%B1%8F2021-12-13%20%E4%B8%8B%E5%8D%887.19.07.png)

在调用 `doGetBean()` 方法时，有一个 `requiredType` 参数，该参数的功能就是将返回的 bean 转换为 `requiredType` 类型。当然就一般而言我们是不需要进行类型转换的，也就是 requiredType 为空（比如 `getBean(String name)`），但有可能会存在这种情况，比如我们返回的 bean 类型为 String，我们在使用的时候需要将其转换为 Integer，那么这个时候 requiredType 就有用武之地了。当然我们一般是不需要这样做的。



至此 `getBean()` 过程讲解完了。后续将会对该过程进行拆分，更加详细的说明，弄清楚其中的来龙去脉，所以这篇博客只能算是 Spring bean 加载过程的一个概览。拆分主要是分为三个部分：

* （1）分析从缓存中获取单例 bean，以及对 bean 的实例中获取对象
* （2）如果从单例缓存中获取 bean，Spring 是怎么加载的呢？所以第二部分是分析 bean 加载，以及 bean 的依赖处理
* （3）bean 已经加载了，依赖也处理完毕了，第三部分则分析各个作用域的 bean 初始化过程。