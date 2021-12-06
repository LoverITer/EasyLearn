## Spring IOC源码阅读（三）Bean加载流程



```java
public class Main {

    public static void main(String[] args) {
        ClassPathResource resource = new ClassPathResource("application.xml");
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions(resource);
    }

}
```

这段代码是 Spring 中编程式使用 IOC 容器，通过这四段简单的代码，我们可以初步判断 IOC 容器的使用过程。

- **第一步**：获取资源，通过加载`xml`配置文件获取到具体需要加载的资源
- **第二步**：获取 `BeanFactory`
- **第三步**：根据新建的 BeanFactory 创建一个`BeanDefinitionReader`对象，该 Reader 对象为资源的解析器
- **第四步**：装载资源 ：整个过程又可以细分为三个步骤：**资源定位、装载、注册**
  - **资源定位**。我们一般用外部资源来描述 Bean 对象，所以在初始化 IOC 容器的第一步就是需要定位这个外部资源。PS：在上一篇博客（[Spring IOC源码阅读（二）Spring 资源加载策略](./Spring IOC源码阅读（二）Spring 资源加载策略)）已经详细说明了Spring加载资源的过程。
  - **装载**。装载就是 `BeanDefinition` 的载入。BeanDefinitionReader 读取、解析 Resource 资源后会将用户定义的 Bean 表示成 IOC 容器的内部数据结构：`BeanDefinition`。在 IOC 容器内部维护着一个 BeanDefinitionMap 的数据结构，在配置文件中每一个 `<bean>` 都对应着一个BeanDefinition对象。
  - **注册**。向IOC容器注册在第二步解析好的 BeanDefinition，这个过程是通过 `BeanDefinitionRegistry` 接口来实现的。在 IOC 容器内部其实是将第二个过程解析得到的 BeanDefinition 注入到一个 HashMap 容器中，IOC 容器就是通过这个 HashMap 来维护这些 BeanDefinition 的。在这里需要注意的一点是这个过程并没有完成依赖注入，依赖注册是发生在应用第一次调用 `getBean()` 向容器索要 Bean 时。当然我们可以通过设置预处理，即对某个 Bean 设置 lazyinit 属性，那么这个 Bean 的依赖注入就会在容器初始化的时候完成。



资源定位在前面已经分析了，下面我们直接分析加载，下图所示是`BeanDefinitionReader` 接口的定义：

```java
public interface BeanDefinitionReader {

	/**
	 * Return the bean factory to register the bean definitions with.
	 * 返回 bean 工厂以注册 bean 定义。
	 */
	BeanDefinitionRegistry getRegistry();

	/**
	 * Return the resource loader to use for resource locations.
	 * 返回用于资源位置的资源加载器。
	 */
	@Nullable
	ResourceLoader getResourceLoader();

	/**
	 * Return the class loader to use for bean classes.
	 * 返回用于 bean 类的类加载器。
	 */
	@Nullable
	ClassLoader getBeanClassLoader();

	/**
	 * Return the BeanNameGenerator to use for anonymous beans
	 * 返回用于匿名 bean 的 BeanNameGenerator
	 */
	BeanNameGenerator getBeanNameGenerator();


	/**
	 * Load bean definitions from the specified resource.
	 * 从指定的资源加载 bean 定义。
	 */
	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;

	/**
	 * Load bean definitions from the specified resources.
	 * 从指定的一个或多个资源加载 bean 定义。
	 */
	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;

	/**
	 * Load bean definitions from the specified resource location.
	 * 从指定的资源位置加载 bean 定义。
	 */
	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;

	/**
	 * Load bean definitions from the specified resource locations.
	 * 从指定的一个或多个资源位置加载 bean 定义。
	 */
	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;

}
```

接口有三个重要的实现类：`AbstractBeanDefinitionReader` 以及它的两个子类  `XmlBeanDefinitionReader` 和 `GroovyBeanDefinitionReader`。解析XML文件，程序会选择使用XmlBeanDefinitionReader， 在加载bean时会调用 `loadBeanDefinitions(Resource)`方法，它又会调用另一个真正干活的方法`loadBeanDefinitions(EncodedResource)`：

```java
/**                                                                                                   
 * 从指定的 XML 文件加载 bean 定义。   
 *
 * @param encodedResource  XML 文件的资源描述符，允许指定用于解析文件的编码                                       
 * @return 找到的 bean 定义数                                                     
 * @throws BeanDefinitionStoreException in case of loading or parsing errors                          
 */                                                                                                   
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException { 
	Assert.notNull(encodedResource, "EncodedResource must not be null");                              
	if (logger.isTraceEnabled()) {                                                                    
		logger.trace("Loading XML bean definitions from " + encodedResource);                         
	}                                                                                                 
  // 获取已经加载过的资源                                                                                                    
	Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();                 
  //检查没有加载过才会继续加载，否则就会报错                                                                                                    
	if (!currentResources.add(encodedResource)) {                                                     
		throw new BeanDefinitionStoreException(                                                       
				"Detected cyclic loading of " + encodedResource + " - check your import definitions!")
	}                                                                                                 
  // 从 EncodedResource 获取封装的 Resource 并从 Resource 中获取其中的 InputStream                                                                                                    
	try (InputStream inputStream = encodedResource.getResource().getInputStream()) {                  
		InputSource inputSource = new InputSource(inputStream);                                       
		if (encodedResource.getEncoding() != null) { 
      // 设置编码
			inputSource.setEncoding(encodedResource.getEncoding());                                   
		}
    // 核心逻辑部分
		return doLoadBeanDefinitions(inputSource, encodedResource.getResource());                     
	} catch (IOException ex) {                                                                          
		throw new BeanDefinitionStoreException(                                                       
				"IOException parsing XML document from " + encodedResource.getResource(), ex);        
	} finally { 
    // 从缓存中剔除该资源
		currentResources.remove(encodedResource);                                                     
		if (currentResources.isEmpty()) {                                                             
			this.resourcesCurrentlyBeingLoaded.remove();                                              
		}                                                                                             
	}                                                                                                 
}                                                                                                     
```

**代码核心逻辑**：

* （1）通过`resourcesCurrentlyBeingLoaded.get()` 来获取已经加载过的资源
* （2）判断是否可以将 encodedResource（XML 文件的资源描述符） 加入其中
  * （2.1）如果 resourcesCurrentlyBeingLoaded 中已经存在该资源，则抛出 BeanDefinitionStoreException 异常。
  * （2.2）resourcesCurrentlyBeingLoaded不存在该资源：
  * （2.2.1）将 encodedResource 加入其中
  * （2.2.2）从 encodedResource 获取封装的 Resource 资源并从 Resource 中获取相应的 InputStream ，
  * （2.2.3）将 InputStream 封装为 InputSource 调用 `doLoadBeanDefinitions()`。

方法 `doLoadBeanDefinitions()` 为从 xml 文件中加载 Bean Definition 的真正逻辑，如下:

```java
/**                                                                                                      
 * 实际的从指定的 XML 文件加载 bean 定义。                                         
 * @param inputSource 要从中读取的 SAX 输入源                                                
 * @param resource XML 文件的资源描述符                                           
 * @return 找到的 bean 定义数                                                            
 * @throws BeanDefinitionStoreException in case of loading or parsing errors                                                                                                     
 */                                                                                                      
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)                          
		throws BeanDefinitionStoreException {                                                            
                                                                                                         
	try {
    // 获取 Document 实例
		Document doc = doLoadDocument(inputSource, resource);  
    // 根据 Document 实例，注册 Bean信息
		int count = registerBeanDefinitions(doc, resource);                                              
		if (logger.isDebugEnabled()) {                                                                   
			logger.debug("Loaded " + count + " bean definitions from " + resource);                      
		}                                                                                                
		return count;                                                                                    
	} catch (BeanDefinitionStoreException ex) {                                                            
		throw ex;                                                                                        
	} catch (SAXParseException ex) {                                                                       
		throw new XmlBeanDefinitionStoreException(resource.getDescription(),                             
				"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex); 
	} catch (SAXException ex) {                                                                            
		throw new XmlBeanDefinitionStoreException(resource.getDescription(),                             
				"XML document from " + resource + " is invalid", ex);                                    
	} catch (ParserConfigurationException ex) {                                                            
		throw new BeanDefinitionStoreException(resource.getDescription(),                                
				"Parser configuration exception parsing XML from " + resource, ex);                      
	} catch (IOException ex) {                                                                             
		throw new BeanDefinitionStoreException(resource.getDescription(),                                
				"IOException parsing XML document from " + resource, ex);                                
	} catch (Throwable ex) {                                                                               
		throw new BeanDefinitionStoreException(resource.getDescription(),                                
				"Unexpected exception parsing XML document from " + resource, ex);                       
	}                                                                                                    
}                                                                                                        
```

核心部分就是 try 块的两行代码。

1.  `doLoadDocument()` ：根据 xml 文件获取 Document 实例。
2. `registerBeanDefinitions()`：根据获取的 Document 实例注册 Bean 实例。 

其中，`doLoadDocument()`方法内部还获取了 xml 文件的验证模式（`getValidationModeForResource(resource)`）。如下:

```java
protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
   return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
         getValidationModeForResource(resource), isNamespaceAware());
}
```

所以 `doLoadBeanDefinitions()`主要就是做了三件事情。

1. 调用 `getValidationModeForResource()` 获取 xml 文件的验证模式 
2. 调用 `loadDocument()` 根据 xml 文件获取相应的 Document 实例。 
3.  调用 `registerBeanDefinitions()` 注册 Bean 实例。