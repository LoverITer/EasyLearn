## Spring IOC源码阅读（五）注册 BeanDefinition

接上一篇：[Spring IOC源码阅读（三）Bean加载流程](./Spring IOC源码阅读（三）Bean加载流程)，在`XmlBeanDefinitionReader.doLoadBeanDefinitions()`方法中，当获取到 Document 对象后，Spring会根据 Document 对象和 Resource 资源对象调用 `XmlBeanDefinitionReader.registerBeanDefinitions()` 方法，开始注册 BeanDefinitions 之旅，如下：

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   int countBefore = getRegistry().getBeanDefinitionCount();
   //核心方法
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

核心的逻辑在**`BeanDefinitionDocumentReader.registerBeanDefinitions()`**中，这个方法是`BeanDefinitionDocumentReader`接口唯一的一个方法，主要作用是==从给定的 DOM 文档中读取 bean 定义，并将它们注册到给定阅读器上下文中的注册表中==，在Spring中有个默认实现类 `DefaultBeanDefinitionDocumentReader`

```java
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   this.readerContext = readerContext;
   //真正干活的哥们~~
   doRegisterBeanDefinitions(doc.getDocumentElement());
}
```

在默认实现类中它又包装了一个方法，真正干活的方法是**`BeanDefinitionDocumentReader.doRegisterBeanDefinitions()`**，如下：

```java
/**                                                                             
 * Register each bean definition within the given root {@code <beans/>} element.
 */                                                                             
protected void doRegisterBeanDefinitions(Element root) {                                                                              
	BeanDefinitionParserDelegate parent = this.delegate;                                                       
	this.delegate = createDelegate(getReaderContext(), root, parent);                                          
                                                                                                               
	if (this.delegate.isDefaultNamespace(root)) { 
    //处理profile
		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);                                             
		if (StringUtils.hasText(profileSpec)) {                                                                
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(                                    
					profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);                                                                   
			if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {                     
				if (logger.isDebugEnabled()) {                                                                 
					logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
							"] not matching: " + getReaderContext().getResource());                            
				}                                                                                              
				return;                                                                                        
			}                                                                                                  
		}                                                                                                      
	}                                                                                                          
  //解析前处理                                                                                                             
	preProcessXml(root);  
  //解析
	parseBeanDefinitions(root, this.delegate);  
  //解析后处理
	postProcessXml(root);                                                                                      
                                                                                                               
	this.delegate = parent;                                                                                    
}                                                                                                              
```

通过Spring源码的注释，我们可以大概知道这个方法的作用是==在给定的根 \<beans/> 元素中注册每个 bean 定义==。

* （1）程序首先处理 profile属性，profile主要用于我们切换环境，比如切换开发、测试、生产环境等。
* （2）然后调用 **`BeanDefinitionDocumentReader.parseBeanDefinitions()`** 进行解析动作，不过在该方法前后分别调用 `BeanDefinitionDocumentReader.preProcessXml()` 和 `BeanDefinitionDocumentReader.postProcessXml()` 方法来进行前、后处理，不过这两个方法都是空实现，交由子类来实现进行扩展性配置。

<img src="http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-11-30%20%E4%B8%8B%E5%8D%885.43.57.png" style="width:80%;" />

**注册BeanDefinitions**：`BeanDefinitionDocumentReader.parseBeanDefinitions()` 定义如下：

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   if (delegate.isDefaultNamespace(root)) {
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element ele) {
            if (delegate.isDefaultNamespace(ele)) {
              //解析默认标签
               parseDefaultElement(ele, delegate);
            }
            else {
              //解析自定义标签
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
     //解析自定义标签
      delegate.parseCustomElement(root);
   }
}
```

方法有两个入参：

* `Element root` ：文档的 DOM 根元素

*  `BeanDefinitionParserDelegate delegate` ：用于解析 XML Bean 定义的有状态委托类。说白了，这个类里面就存的是xml文件解析之后的元数据，点进去就可以看到类中定义了许多我们熟悉的常量标签字符串：

  ![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-11-30%20%E4%B8%8B%E5%8D%885.51.56.png)

最终解析动作落地在两个方法上：`BeanDefinitionDocumentReader.parseDefaultElement(ele, delegate)` 和 `BeanDefinitionParserDelegate.parseCustomElement(root)`。我们知道在 Spring 有两种 Bean 声明方式：

- **配置文件式声明**：`<bean id="studentService" class="org.springframework.core.StudentService"/>`
- **自定义注解方式**：`<tx:annotation-driven>`

两种方式的读取和解析都存在较大的差异，所以采用不同的解析方法：

* （1）如果根节点或者子节点采用默认命名空间的话，则调用 `BeanDefinitionDocumentReader.parseDefaultElement()` 进行解析
* （2）根节点或者子节点不是采用默认命名空间的话，则调用 `BeanDefinitionParserDelegate.parseCustomElement()` 方法进行自定义解析。 

至此，`doLoadBeanDefinitions()` 中做的三件事情已经全部分析完毕，下期将对 Bean 的解析过程做详细分析说明。

