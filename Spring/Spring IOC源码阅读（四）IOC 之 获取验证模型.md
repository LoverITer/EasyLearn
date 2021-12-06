## Spring IOC源码阅读（四）IOC 之 获取验证模型

在上篇博客[Spring IOC源码阅读（三）Bean加载流程](./Spring IOC源码阅读（三）Bean加载流程) 中提到，Bean加载过程最终落在了在核心逻辑方法 `doLoadBeanDefinitions()`，而它又主要是做三件事情：

1. 调用 `getValidationModeForResource()` 获取 xml 文件的验证模式
2. 调用 `loadDocument()` 根据 xml 文件获取相应的 Document 实例。
3. 调用 `registerBeanDefinitions()` 注册 Bean 实例。

这篇博客主要分析获取 xml 文件的验证模式。注册Bean示例我们放到下篇来讲解。

>  **Q**：什么是xml 文件的验证模式？
>
> **A**：XML 文件的验证模式保证了 XML 文件的正确性



## DTD 与 XSD 的区别

**DTD(Document Type Definition)**，即文档类型定义，为 XML 文件的验证机制，属于 XML 文件中组成的一部分。DTD 是一种保证 XML 文档格式正确的有效验证方式，它定义了相关 XML 文档的元素、属性、排列方式、元素的内容类型以及元素的层次结构。其实 DTD 就相当于 XML 中的 “词汇”和“语法”，我们可以通过比较 XML 文件和 DTD 文件 来看文档是否符合规范，元素和标签使用是否正确。 要在 Spring 中使用 DTD，需要在 Spring XML 文件头部声明：

```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC  "-//SPRING//DTD BEAN//EN"  "http://www.springframework.org/dtd/spring-beans.dtd">
```

DTD 在一定的阶段推动了 XML 的发展，但是它本身存在着一些缺陷：

1. 它没有使用 XML 格式，而是自己定义了一套格式，相对解析器的重用性较差；而且 DTD 的构建和访问没有标准的编程接口，因而解析器很难简单的解析 DTD 文档。
2. DTD 对元素的类型限制较少；同时其他的约束力也叫弱。
3. DTD 扩展能力较差。
4. 基于正则表达式的 DTD 文档的描述能力有限。

针对 DTD 的缺陷，W3C 在 2001 年推出 XSD。**XSD（XML Schemas Definition）**即 XML Schema 语言。XML Schema 本身就是一个 XML文档，使用的是 XML 语法，因此可以很方便的解析 XSD 文档。相对于 DTD，XSD 具有如下优势：

- XML Schema基于XML,没有专门的语法
- XML Schema可以象其他XML文件一样解析和处理
- XML Schema比DTD提供了更丰富的数据类型.
- XML Schema提供可扩充的数据模型。
- XML Schema支持综合命名空间
- XML Schema支持属性组。



## getValidationModeForResource() 分析

```java
protected int getValidationModeForResource(Resource resource) {
   int validationModeToUse = getValidationMode();
   if (validationModeToUse != VALIDATION_AUTO) {
      return validationModeToUse;
   }
   int detectedMode = detectValidationMode(resource);
   if (detectedMode != VALIDATION_AUTO) {
      return detectedMode;
   }
   // Hmm, we didn't get a clear indication... Let's assume XSD,
   // since apparently no DTD declaration has been found up until
   // detection stopped (before finding the document's root tag).
   return VALIDATION_XSD;
}
```

**代码核心逻辑**：

* （1）调用`getValidationMode()`尝试获取指定的验证模式
* （2）如果是显示配置的验证模式，直接返回
* （3）否则从给定的`resource`通过`detectValidationModef`方法探测出验证模式，如果检测发生异常，那么会默认返回XSD验证模式，这部分逻辑如下：

```java
protected int detectValidationMode(Resource resource) {                                                  
	if (resource.isOpen()) {                                                                             
		throw new BeanDefinitionStoreException(                                                          
				"Passed-in Resource [" + resource + "] contains an open stream: " +                      
				"cannot determine validation mode automatically. Either pass in a Resource " +           
				"that is able to create fresh streams, or explicitly specify the validationMode " +      
				"on your XmlBeanDefinitionReader instance.");                                            
	}                                                                                                    
                                                                                                         
	InputStream inputStream;                                                                             
	try {                                                                                                
		inputStream = resource.getInputStream();                                                         
	}                                                                                                    
	catch (IOException ex) {                                                                             
		throw new BeanDefinitionStoreException(                                                          
				"Unable to determine validation mode for [" + resource + "]: cannot open InputStream. " +
				"Did you attempt to load directly from a SAX InputSource without specifying the " +      
				"validationMode on your XmlBeanDefinitionReader instance?", ex);                         
	}                                                                                                    
                                                                                                         
	try {
    //核心方法
		return this.validationModeDetector.detectValidationMode(inputStream);                            
	}                                                                                                    
	catch (IOException ex) {                                                                             
		throw new BeanDefinitionStoreException("Unable to determine validation mode for [" +             
				resource + "]: an error occurred whilst reading from the InputStream.", ex);             
	}                                                                                                    
}                                                                                                        
```

前面一大堆的代码对于理解Spring IOC原理不重要，核心在于 `this.validationModeDetector.detectValidationMode(inputStream)`，validationModeDetector 定义为 `XmlValidationModeDetector`，所以验证模式的获取委托给 `XmlValidationModeDetector` 的 `detectValidationMode()` 方法。

```java
public int detectValidationMode(InputStream inputStream) throws IOException {
   // Peek into the file to look for DOCTYPE.
   try (BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
      boolean isDtdValidated = false;
      String content;
      while ((content = reader.readLine()) != null) {
         content = consumeCommentTokens(content);
         if (this.inComment || !StringUtils.hasText(content)) {
            continue;
         }
         //判断是哪种种验证模型类型的核心
         if (hasDoctype(content)) {
            isDtdValidated = true;
            break;
         }
         if (hasOpeningTag(content)) {
            // End of meaningful data...
            break;
         }
      }
      return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
   }
   catch (CharConversionException ex) {
      // Choked on some character encoding...
      // Leave the decision up to the caller.
      return VALIDATION_AUTO;
   }
}
```

其实都不用看方法的逻辑，只需要看`XmlValidationModeDetector` 这个类的两个常量就知道大致判断逻辑是什么样的了：

![](img/%E6%88%AA%E5%B1%8F2021-11-30%20%E4%B8%8B%E5%8D%885.08.30.png)

通过源码注释得知，判断的语句主要是判断文档中是否存在`DOCTYPE` 这个字符串，如果发现这个字符串就认定为DTD，否则就是XSD。知道这一点，再去看源码就会发现代码逻辑还是比较清晰的~~