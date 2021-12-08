## Spring IOC源码阅读（六）解析Bean—解析import标签

在上一篇：[Spring IOC源码阅读（五）注册 BeanDefinition](./Spring IOC源码阅读（五）注册 BeanDefinition) 中最后分析知道，Spring 有两种解析 Bean 的方式：

* （1）如果根节点或子节点采用默认命名空间的话，那么会调用 `BeanDefinitionDocumentReader.parseDefaultElement(Element, BeanDefinitionParserDelegate)` 来进行默认标签解析
* （2）如果根节点或子节点不是采用默认命名空间的话，则调用 `BeanDefinitionParserDelegate.parseCustomElement(root)` 方法进行自定义解析。

本文就围绕这两个方法，分别分析不同状态下标签的解析过程，先从默认的解析过程开始：

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
  if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {                               
    //对import标签进行解析
		importBeanDefinitionResource(ele);                                            
	} else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) { 
    //对alias标签进行解析
		processAliasRegistration(ele);                                                
	} else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) { 
    //对bean标签进行解析
		processBeanDefinition(ele, delegate);                                         
	} else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {                    
		// 对嵌套的bean标签进行解析                                                                   
		doRegisterBeanDefinitions(ele);                                               
	}                                                                                 
}                                                                                     
```

方法的功能一目了然，分别是对四种不同的标签进行解析，分别是 import、alias、bean、beans四种标签进行解析。咱门从第一个标签 import 开始。



## import标签的处理

使用过 Spring 配置文件的小伙伴都知道，如果工程比较大，配置文件的维护会让人觉得恐怖，文件太多了，想象将所有的配置都放在一个 application.xml 配置文件中，哪种后怕感是不是很明显？所有针对这种情况 Spring 提供了一个分模块的思路，利用 `import `标签，例如我们可以构造一个这样的 application.xml。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
    
   <import resource="spring-student.xml"/>
   <import resource="spring-student-dtd.xml"/>
</beans>
```

我们可以在配置文件中使用 import 标签的方式导入其他模块的配置文件，如果有配置需要修改直接修改相应配置文件即可，若有新的模块需要引入直接增加 import 即可，这样大大简化了配置后期维护的复杂度，同时也易于管理。 Spring 利用 `importBeanDefinitionResource()` 方法完成对 import 标签后面resource指定资源的导入。

```java
protected void importBeanDefinitionResource(Element ele) {
   String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
   if (!StringUtils.hasText(location)) {
      getReaderContext().error("Resource location must not be empty", ele);
      return;
   }

   // Resolve system properties: e.g. "${user.dir}"
   location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

   Set<Resource> actualResources = new LinkedHashSet<>(4);

   // Discover whether the location is an absolute or relative URI
   boolean absoluteLocation = false;
   try {
      absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
   }
   catch (URISyntaxException ex) {
      // cannot convert to an URI, considering the location relative
      // unless it is the well-known Spring prefix "classpath*:"
   }

   // Absolute or relative?
   if (absoluteLocation) {
      try {
         int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
         if (logger.isTraceEnabled()) {
            logger.trace("Imported " + importCount + " bean definitions from URL location [" + location + "]");
         }
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error(
               "Failed to import bean definitions from URL location [" + location + "]", ele, ex);
      }
   }
   else {
      // No URL -> considering resource location as relative to the current file.
      try {
         int importCount;
         Resource relativeResource = getReaderContext().getResource().createRelative(location);
         if (relativeResource.exists()) {
            importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
            actualResources.add(relativeResource);
         }
         else {
            String baseLocation = getReaderContext().getResource().getURL().toString();
            importCount = getReaderContext().getReader().loadBeanDefinitions(
                  StringUtils.applyRelativePath(baseLocation, location), actualResources);
         }
         if (logger.isTraceEnabled()) {
            logger.trace("Imported " + importCount + " bean definitions from relative location [" + location + "]");
         }
      }
      catch (IOException ex) {
         getReaderContext().error("Failed to resolve current resource location", ele, ex);
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error(
               "Failed to import bean definitions from relative location [" + location + "]", ele, ex);
      }
   }
   Resource[] actResArray = actualResources.toArray(new Resource[0]);
   getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
}
```

解析 import 过程较为清晰，整个过程如下：

* （1）获取 **resource** 属性的值，获取该值表示资源的路径，如果属性的值为空直接返回
* （2）解析路径中的用户定义的动态属性，如"${user.dir}"
* （3）判断资源路径 location 是绝对路径还是相对路径
  * （3.1）如果是**绝对路径**，则调递归调用 Bean 的解析过程，进行另一次的解析
  * （3.2）如果是**相对路径**，则先计算出绝对路径得到 Resource，然后进行解析
* （4）通知监听器，完成解析



Spring是如何判断获得的location是相对路径还是绝对路径的呢？

```java
absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();

//以 `classpath*: `或者 `classpath: `开头为绝对路径
public static boolean isUrl(@Nullable String resourceLocation) {                             
	return (resourceLocation != null &&                                                      
			(resourceLocation.startsWith(ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX) ||
					ResourceUtils.isUrl(resourceLocation)));                                 
} 

//能够通过该 `location` 构建出 `java.net.URL` 为绝对路径
public static boolean isUrl(@Nullable String resourceLocation) {
	if (resourceLocation == null) {                             
		return false;                                           
	}                                                           
	if (resourceLocation.startsWith(CLASSPATH_URL_PREFIX)) {    
		return true;                                            
	}                                                           
	try {                                                       
		new URL(resourceLocation);                              
		return true;                                            
	}                                                           
	catch (MalformedURLException ex) {                          
		return false;                                           
	}                                                           
}                                                               
```

判断绝对路径的规则如下：

- 以 `classpath*: `或者 `classpath: `开头为绝对路径
- 能够通过该 `location` 构建出 `java.net.URL` 为绝对路径
- 根据 `location` 构造 `java.net.URI` 判断调用 `isAbsolute()` 判断是否为绝对路径



如果 location 为绝对路径则调用 `AbstractBeanDefinitionReader.loadBeanDefinitions()`，注意该方法在 `AbstractBeanDefinitionReader` 中定义，它是前面提到的`XmlBeanDefinitionReader` 的父类。

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
	ResourceLoader resourceLoader = getResourceLoader();                                                                  
	if (resourceLoader == null) {                                                                                         
		throw new BeanDefinitionStoreException(                                                                           
				"Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");            
	}                                                                                                                     
                                                                                                                          
	if (resourceLoader instanceof ResourcePatternResolver) {                                                              
		// Resource pattern matching available.                                                                           
		try {                                                                                                             
			Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);                     
			int count = loadBeanDefinitions(resources);                                                                   
			if (actualResources != null) {                                                                                
				Collections.addAll(actualResources, resources);                                                           
			}                                                                                                             
			if (logger.isTraceEnabled()) {                                                                                
				logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");           
			}                                                                                                             
			return count;                                                                                                 
		}                                                                                                                 
		catch (IOException ex) {                                                                                          
			throw new BeanDefinitionStoreException(                                                                       
					"Could not resolve bean definition resource pattern [" + location + "]", ex);                         
		}                                                                                                                 
	}                                                                                                                     
	else {                                                                                                                
		// Can only load single resources by absolute URL.                                                                
		Resource resource = resourceLoader.getResource(location);                                                         
		int count = loadBeanDefinitions(resource);                                                                        
		if (actualResources != null) {                                                                                    
			actualResources.add(resource);                                                                                
		}                                                                                                                 
		if (logger.isTraceEnabled()) {                                                                                    
			logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");                       
		}                                                                                                                 
		return count;                                                                                                     
	}                                                                                                                     
}                                                                                                                         
```

方法整体逻辑是比较清晰的：

* （1）首先获得资源的ResourceLoader，这个在之前的文章有分析过 [Spring IOC源码阅读（二）Spring 资源加载策略](./Spring IOC源码阅读（二）Spring 资源加载策略)
* （2）如果资源的定位器为空，那么资源就无法被加载，抛出异常
* （3）否则根据ResourceLoader的具体类型执行对应操作：
  * （3.1）如果是 ResourcePatternResolver ，表示给定的路径是根据模式匹配的批量路径，则调用批量资源的方法批量加载
  * （3.2）否则表示给定的路径就是一个资源，使用根据一个路径加载资源的方法加载资源

至此，import 标签解析完毕，整个过程比较清晰明了：**获取 source 属性值，得到正确的资源路径，然后调用 `loadBeanDefinitions()` 方法进行递归的 BeanDefinition 加载**。下节我们来分析[bean标签的解析和注册](./Spring IOC源码阅读（七）解析Bean—解析bean标签)。

