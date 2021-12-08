## Spring IOC源码阅读（七）解析Bean—解析并注册bean



### 一、解析bean标签获取BeanDefinition

import 标签解析完毕了，再看 Spring 中最复杂也是最重要的标签 bean 标签的解析过程。 在方法 `BeanDefinitionDocumentReader.parseDefaultElement()` 中，如果遇到标签 为 bean 则调用 `BeanDefinitionDocumentReader.processBeanDefinition()`方法进行 bean 标签解析，如下：

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {                
	BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);                             
	if (bdHolder != null) {                                                                               
		bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);                              
		try {                                                                                             
			// Register the final decorated instance.                                                     
			BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry()); 
		}                                                                                                 
		catch (BeanDefinitionStoreException ex) {                                                         
			getReaderContext().error("Failed to register bean definition with name '" +                   
					bdHolder.getBeanName() + "'", ele, ex);                                               
		}                                                                                                 
		// Send registration event.                                                                       
		getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));                
	}                                                                                                     
}                                                                                                         
```

整个过程分为四个步骤

* （1）调用 `BeanDefinitionParserDelegate.parseBeanDefinitionElement()` 进行元素解析，解析过程中如果失败，返回 null，错误由 `ProblemReporter` 处理。如果解析成功则返回` BeanDefinitionHolder` 实例 bdHolder。BeanDefinitionHolder 为持有 name 和 alias 的 BeanDefinition。
* （2）若实例 bdHolder 不为空，则调用 `BeanDefinitionParserDelegate.decorateBeanDefinitionIfRequired()` 进行自定义标签处理
* （3）解析完成后，调用 `BeanDefinitionReaderUtils.registerBeanDefiniation` 注册bean
* （4）发出响应事件，通知相关的监听器已经完成 Bean 标签解析

先看方法 `BeanDefinitionParserDelegate.parseBeanDefinitionElement()`，如下：

```java
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {   
  //解析ID属性
	String id = ele.getAttribute(ID_ATTRIBUTE);   
  //解析name属性
	String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);                                                          
  //解析aliases属性                                                                                                              
	List<String> aliases = new ArrayList<>();                                                                    
	if (StringUtils.hasLength(nameAttr)) {                                                                       
		String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);        
		aliases.addAll(Arrays.asList(nameArr));                                                                  
	}                                                                                                            
                                                                                                                 
	String beanName = id;                                                                                        
	if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {                                                  
		beanName = aliases.remove(0);                                                                            
		if (logger.isTraceEnabled()) {                                                                           
			logger.trace("No XML 'id' specified - using '" + beanName +                                          
					"' as bean name and " + aliases + " as aliases");                                            
		}                                                                                                        
	}                                                                                                            
                                                                                                                 
	if (containingBean == null) { 
    //检查name的唯一性
		checkNameUniqueness(beanName, aliases, ele);                                                             
	}                                                                                                            
                                                                                                                 
	AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);           
	if (beanDefinition != null) {                                                                                
		if (!StringUtils.hasText(beanName)) {                                                                    
			try {                                                                                                
				if (containingBean != null) {  
          //如果beanName不存在，则根据规则自动生成
					beanName = BeanDefinitionReaderUtils.generateBeanName(                                       
							beanDefinition, this.readerContext.getRegistry(), true);                             
				}                                                                                                
				else {                                                                                           
					beanName = this.readerContext.generateBeanName(beanDefinition);                              
					// Register an alias for the plain bean class name, if still possible,                       
					// if the generator returned the class name plus a suffix.                                   
					// This is expected for Spring 1.2/2.0 backwards compatibility.                              
					String beanClassName = beanDefinition.getBeanClassName();                                    
					if (beanClassName != null &&                                                                 
							beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&  
							!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {                  
						aliases.add(beanClassName);                                                              
					}                                                                                            
				}                                                                                                
				if (logger.isTraceEnabled()) {                                                                   
					logger.trace("Neither XML 'id' nor 'name' specified - " +                                    
							"using generated bean name [" + beanName + "]");                                     
				}                                                                                                
			}                                                                                                    
			catch (Exception ex) {                                                                               
				error(ex.getMessage(), ele);                                                                     
				return null;                                                                                     
			}                                                                                                    
		}                                                                                                        
		String[] aliasesArray = StringUtils.toStringArray(aliases);  
    //封装成BeanDefinitionHolder返回 ：BeanDefinitionHolder就是包含有Bean的beanName以及aliases的BeanDefinition
		return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);                                 
	}                                                                                                            
                                                                                                                 
	return null;                                                                                                 
}                                                                                                                
```

这个方法还没有直接上来就对 Bean 标签进行解析，只是在解析动作之前做了一些功能架构，主要的工作有：

- 解析 id、name 属性，确定 alias 集合，检测 beanName 是否唯一
- 对属性进行解析并封装成 AbstractBeanDefinition 实例 beanDefinition
- 根据所获取的信息（beanName、aliases、beanDefinition）构造 BeanDefinitionHolder 实例对象并返回。

这里有必要说下 beanName 的命名规则：如果 id 不为空，则 beanName = id；如果 id 为空，但是 alias 不空，则 beanName 为 alias 的第一个元素，如果两者都为空，则根据默认规则来设置 beanName。 上面三个步骤第二个步骤为核心方法，它主要承担解析 Bean 标签中所有的属性值。如下：

```java
@Nullable                                                                                        
public AbstractBeanDefinition parseBeanDefinitionElement(Element ele, String beanName, @Nullable BeanDefinition containingBean) {                 
                                                                                                 
	this.parseState.push(new BeanEntry(beanName));                                               
                                                                                                 
	String className = null;                                                                     
	if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
    //解析class属性
		className = ele.getAttribute(CLASS_ATTRIBUTE).trim();                                    
	}                                                                                            
	String parent = null;                                                                        
	if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
    //解析parent属性
		parent = ele.getAttribute(PARENT_ATTRIBUTE);                                             
	}                                                                                            
                                                                                                 
	try {                                  
    //创建用户存放bean定义的 GenericBeanDefinition 实例
		AbstractBeanDefinition bd = createBeanDefinition(className, parent);                     
    
    //解析默认bean的各种属性                                                                                            
		parseBeanDefinitionAttributes(ele, beanName, containingBean, bd); 
    
    //解析bean的description
		bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));     
     
    //解析元数据
		parseMetaElements(ele, bd);   
    
    //解析lookup-method
		parseLookupOverrideSubElements(ele, bd.getMethodOverrides()); 
    
    //解析replaced-method
		parseReplacedMethodSubElements(ele, bd.getMethodOverrides());                            
    
    //解析构造函数参数
		parseConstructorArgElements(ele, bd);  
    
    //解析property子元素
		parsePropertyElements(ele, bd); 
    
    //解析qualifier子元素
		parseQualifierElements(ele, bd);                                                         
                                                                                                 
		bd.setResource(this.readerContext.getResource());                                        
		bd.setSource(extractSource(ele));                                                        
                                                                                                 
		return bd;                                                                               
	}                                                                                            
	catch (ClassNotFoundException ex) {                                                          
		error("Bean class [" + className + "] not found", ele, ex);                              
	}                                                                                            
	catch (NoClassDefFoundError err) {                                                           
		error("Class that bean class [" + className + "] depends on not found", ele, err);       
	}                                                                                            
	catch (Throwable ex) {                                                                       
		error("Unexpected failure during bean definition parsing", ele, ex);                     
	}                                                                                            
	finally {                                                                                    
		this.parseState.pop();                                                                   
	}                                                                                            
                                                                                                 
	return null;                                                                                 
}                                                                                                
```

到这里，Bean 标签的所有属性我们都可以看到其解析的过程，也就说到这里我们已经解析一个基本可用的 **BeanDefinition**。



### 二、向IOC容器注册BeanDefinition

`DefaultBeanDefinitionDocumentReader.processBeanDefinition()` 完成 Bean 标签解析的核心工作，如下：

![](img/%E6%88%AA%E5%B1%8F2021-12-07%20%E4%B8%8B%E5%8D%887.09.31.png)

解析工作分为三步：

* 1、解析默认标签；
* 2、解析默认标签后下得自定义标签；
* 3、注册解析后的 BeanDefinition。经过前面两个步骤的解析，这时的 BeanDefinition 已经可以满足后续的使用要求了，那么接下来的工作就是将这些 BeanDefinition 进行注册，也就是完成第三步。 注册 BeanDefinition 由 `BeanDefinitionReaderUtils.registerBeanDefinition()` 完成。如下：

![](img/%E6%88%AA%E5%B1%8F2021-12-07%20%E4%B8%8B%E5%8D%887.14.47.png)



BeanDefinition 的注册由接口 BeanDefinitionRegistry 定义。 `BeanDefinitionRegistry.registerBeanDefinition()` 实现通过 beanName 注册 BeanDefinition的逻辑如下：

```java
@Override                                                                                               
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)                      
		throws BeanDefinitionStoreException {                                                           
  //参数校验                                                                                                      
	Assert.hasText(beanName, "Bean name must not be empty");                                            
	Assert.notNull(beanDefinition, "BeanDefinition must not be null");                                  
                                                                                                        
	if (beanDefinition instanceof AbstractBeanDefinition) {                                             
		try { 
      //主要是对属性 methodOverrides 进行校验
			((AbstractBeanDefinition) beanDefinition).validate();                                       
		} catch (BeanDefinitionValidationException ex) {                                                  
			throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,   
					"Validation of bean definition failed", ex);                                        
		}                                                                                               
	}                                          
  
  // 从缓存中获取指定 beanName 的 BeanDefinition 这里的beanDefinitionMap就是所谓的IOC容器                                                                                                     
	BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);                           
	if (existingDefinition != null) {
     // 如果存在但是不允许覆盖，抛出异常
		if (!isAllowBeanDefinitionOverriding()) {                                                       
			throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);    
		} else if (existingDefinition.getRole() < beanDefinition.getRole()) {                             
			// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE       
			if (logger.isInfoEnabled()) {                                                               
				logger.info("Overriding user-defined bean definition for bean '" + beanName +           
						"' with a framework-generated bean definition: replacing [" +                   
						existingDefinition + "] with [" + beanDefinition + "]");                        
			}                                                                                           
		} else if (!beanDefinition.equals(existingDefinition)) {  
       // 覆盖 beanDefinition 与 被覆盖的 beanDefinition 不是同类
			if (logger.isDebugEnabled()) {                                                              
				logger.debug("Overriding bean definition for bean '" + beanName +                       
						"' with a different definition: replacing [" + existingDefinition +             
						"] with [" + beanDefinition + "]");                                             
			}                                                                                           
		} else {                                                                                          
			if (logger.isTraceEnabled()) {                                                              
				logger.trace("Overriding bean definition for bean '" + beanName +                       
						"' with an equivalent definition: replacing [" + existingDefinition +           
						"] with [" + beanDefinition + "]");                                             
			}                                                                                           
		} 
    // 允许覆盖，直接覆盖原有的 BeanDefinition
		this.beanDefinitionMap.put(beanName, beanDefinition);                                           
	} else {     
     // 检测创建 Bean 阶段是否已经开启，如果开启了则需要对 beanDefinitionMap 进行并发控制
		if (hasBeanCreationStarted()) {                                                                 
			// 已经存在beanDefinition创建线程存在了吗，需要进行并发控制         
			synchronized (this.beanDefinitionMap) {                                                     
				this.beanDefinitionMap.put(beanName, beanDefinition);                                   
				List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1); 
				updatedDefinitions.addAll(this.beanDefinitionNames);                                    
				updatedDefinitions.add(beanName);                                                       
				this.beanDefinitionNames = updatedDefinitions;                                          
				removeManualSingletonName(beanName);                                                    
			}                                                                                           
		}	else {                                                                                          
			// 不会存在并发情况，直接设置：将beanDefiniation放到map中                                                    
			this.beanDefinitionMap.put(beanName, beanDefinition);                                       
			this.beanDefinitionNames.add(beanName);                                                     
			removeManualSingletonName(beanName);                                                        
		} 
		this.frozenBeanDefinitionNames = null;                                                          
	}                                                                                                   
                                                                                                        
	if (existingDefinition != null || containsSingleton(beanName)) { 
    // 重新设置 beanName 对应的缓存
		resetBeanDefinition(beanName);                                                                  
	} else if (isConfigurationFrozen()) {                                                                 
		clearByTypeCache();                                                                             
	}                                                                                                   
}                                                                                                       
```

**核心逻辑如下**：

- （1）首先 BeanDefinition 进行各种校验，该校验也是注册过程中的最后一次校验了，主要是对 AbstractBeanDefinition 的 methodOverrides 属性进行校验
- （2）根据 beanName 从缓存中获取 BeanDefinition，如果缓存中存在，则根据 allowBeanDefinitionOverriding 标志来判断是否允许覆盖，如果允许则直接覆盖，否则抛出 BeanDefinitionStoreException 异常
- （3）若缓存中没有指定 beanName 的 BeanDefinition，则判断当前阶段是否已经开始了 Bean 的创建阶段，如果是，则需要对 beanDefinitionMap 进行加锁控制并发问题，否则直接设置即可。对于 `hasBeanCreationStarted()` 方法后续做详细介绍，这里不过多阐述。
- （4）若缓存中存在该 beanName 或者 单例 bean 集合中存在该 beanName，则调用 `resetBeanDefinition()` 重置 BeanDefinition 缓存。

其实整段代码的核心就在于 `this.beanDefinitionMap.put(beanName, beanDefinition)` 。BeanDefinition 的缓存（IOC）也不是神奇的东西，就是定义了一个key 为 beanName，value 为 BeanDefinition 的 Map。 

**注册 alias**： `BeanDefinitionRegistry.registerAlias` 完成 alias 的注册，如下：

```java
@Override                                                                                                 
public void registerAlias(String name, String alias) {  
  //校验参数
	Assert.hasText(name, "'name' must not be empty");                                                     
	Assert.hasText(alias, "'alias' must not be empty");                                                   
	synchronized (this.aliasMap) {                                                                        
		if (alias.equals(name)) {  
      //bean的别名和beanName一样，则从aliasMap中删除此别名
			this.aliasMap.remove(alias);                                                                  
			if (logger.isDebugEnabled()) {                                                                
				logger.debug("Alias definition '" + alias + "' ignored since it points to same name");    
			}                                                                                             
		}else {   
      //bean的别名和beanName不一样，首先根据别名尝试从aliasMap获取此别名
			String registeredName = this.aliasMap.get(alias);                                             
			if (registeredName != null) {  
				if (registeredName.equals(name)) { 
          //已经存在的别名，不需要重复注册                                       
					return;                                                                               
				}                                                                                         
				if (!allowAliasOverriding()) {  
          //不允许覆盖冲突的别名，抛出异常
					throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +    
							name + "': It is already registered for name '" + registeredName + "'.");     
				}                                                                                         
				if (logger.isDebugEnabled()) {                                                            
					logger.debug("Overriding alias '" + alias + "' definition for registered name '" +    
							registeredName + "' with new target name '" + name + "'");                    
				}                                                                                         
			}                                                                                             
			checkForAliasCircle(name, alias);
      //允许重名的别名重写已存在的别名，在aliasMap中放入新的别名和beanName的映射关系
			this.aliasMap.put(alias, name);                                                               
			if (logger.isTraceEnabled()) {                                                                
				logger.trace("Alias definition '" + alias + "' registered for name '" + name + "'");      
			}                                                                                             
		}                                                                                                 
	}                                                                                                     
}                                                                                                         
```



### 总结

最后来一张时序图，把bean的解析和注册串起来：

