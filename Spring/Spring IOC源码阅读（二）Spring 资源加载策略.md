## Spring IOC源码阅读（二）Spring 资源加载策略



在程序的世界里，资源的存在形式多种多样。比如，网络形式的资源、二进制形式存在的、以文件形式存在的、以字节流形式存在的等等。而且它可以存在于任何场所，比如网络、文件系统、应用程序中。因此加载这些资源的方式也各不相同，基于Java对资源操作的局限性迫使 Spring 必须实现自己的资源加载策略，该资源加载策略需要满足如下要求：

1. 职能划分清楚。资源的定义和资源的加载应该要有一个清晰的界限；
2. 统一的抽象。统一的资源定义和资源加载策略。资源加载后要返回统一的抽象给客户端，客户端要对资源进行怎样的处理，应该由抽象资源接口来界定

### Resource：统一资源

`org.springframework.core.io.Resource` 是 Spring 框架所有资源的抽象和访问接口，它继承 `org.springframework.core.io.InputStreamSource` 接口。作为所有资源的统一抽象，Resource 定义了一些通用的方法，并由子类 `AbstractResource` 提供统一的默认实现。

![](https://www.cmsblogs.com/images/group/sike-java/sike-java-spring-ioc/202105092051297881.png)

从上图可以看到，Resource 根据资源的不同类型提供不同的具体实现，如下：

- **FileSystemResource**：对 `java.io.File` 类型资源的封装，只要是跟 File 打交道的，基本上与 FileSystemResource 也可以打交道。支持文件和 URL 的形式，实现 WritableResource 接口，且从 Spring Framework 5.0 开始，FileSystemResource 使用NIO.2 API进行读/写交互
- **ByteArrayResource**：对字节数组提供的数据的封装。如果通过 InputStream 形式访问该类型的资源，该实现会根据字节数组的数据构造一个相应的 ByteArrayInputStream。
- **UrlResource**：对 `java.net.URL`类型资源的封装。内部集合 URI/URL 进行具体的资源操作。
- **ClassPathResource**：class path 类型资源的实现。使用给定的 ClassLoader 或者给定的 Class 来加载资源。
- **InputStreamResource**：将给定的 InputStream 作为一种资源的 Resource 的实现类。

AbstractResource 为 Resource 接口的默认实现，它实现了 Resource 接口的大部分的公共实现，是 Resource 体系的重中之重，其定义如下：

```java
package org.springframework.core.io;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.nio.channels.Channels;
import java.nio.channels.ReadableByteChannel;
import org.springframework.core.NestedIOException;
import org.springframework.lang.Nullable;
import org.springframework.util.ResourceUtils;

public abstract class AbstractResource implements Resource {
    public AbstractResource() {
    }

    //判断资源是否存在
    public boolean exists() {
        try {
            return this.getFile().exists();
        } catch (IOException var4) {
            try {
                this.getInputStream().close();
                return true;
            } catch (Throwable var3) {
                return false;
            }
        }
    }

    //资源是否可读
    public boolean isReadable() {
        return true;
    }

    //资源是否被打开
    public boolean isOpen() {
        return false;
    }

    //是否是文件，加载的资源不一定是文件，具体是否是文件需要其子类来定义
    public boolean isFile() {
        return false;
    }

    //需要子类实现
    public URL getURL() throws IOException {
        throw new FileNotFoundException(this.getDescription() + " cannot be resolved to URL");
    }

    //构建URL
    public URI getURI() throws IOException {
        URL url = this.getURL();

        try {
            return ResourceUtils.toURI(url);
        } catch (URISyntaxException var3) {
            throw new NestedIOException("Invalid URI [" + url + "]", var3);
        }
    }

    //需要子类实现
    public File getFile() throws IOException {
        throw new FileNotFoundException(this.getDescription() + " cannot be resolved to absolute file path");
    }

   //根据getInputStream()返回结构构建读取数据的channal
    public ReadableByteChannel readableChannel() throws IOException {
        return Channels.newChannel(this.getInputStream());
    }

    //获取资源长度，这里获得的就是资源实际的字节长度
    public long contentLength() throws IOException {
        InputStream is = this.getInputStream();

        try {
            long size = 0L;

            int read;
            for(byte[] buf = new byte[256]; (read = is.read(buf)) != -1; size += (long)read) {
            }

            long var6 = size;
            return var6;
        } finally {
            try {
                is.close();
            } catch (IOException var14) {
            }

        }
    }

    //返回资源最后修改的时间
    public long lastModified() throws IOException {
        File fileToCheck = this.getFileForLastModifiedCheck();
        long lastModified = fileToCheck.lastModified();
        if (lastModified == 0L && !fileToCheck.exists()) {
            throw new FileNotFoundException(this.getDescription() + " cannot be resolved in the file system for checking its last-modified timestamp");
        } else {
            return lastModified;
        }
    }

    protected File getFileForLastModifiedCheck() throws IOException {
        return this.getFile();
    }

    //需要子类实现
    public Resource createRelative(String relativePath) throws IOException {
        throw new FileNotFoundException("Cannot create a relative resource for " + this.getDescription());
    }

    @Nullable
    public String getFilename() {
        return null;
    }

    public boolean equals(Object other) {
        return this == other || other instanceof Resource && ((Resource)other).getDescription().equals(this.getDescription());
    }

    public int hashCode() {
        return this.getDescription().hashCode();
    }

    public String toString() {
        return this.getDescription();
    }
}
```

如果我们想要实现自定义的 Resource，记住不要实现 Resource 接口，而应该继承 AbstractResource 抽象类，然后根据当前的具体资源特性覆盖相应的方法即可。



### ResourceLoader：统一资源定位

![](https://www.cmsblogs.com/images/group/sike-java/sike-java-spring-ioc/202105092051301142.png)

Spring 通过 `Resource` 接口完成了对程序世界各种资源的定义，现在资源定义好了，就该有个东西去加载资源。为此，Spring 定义了`org.springframework.core.io.ResourceLoader` ，它抽象了加载资源的具体行为，具体的资源加载则由相应的实现类来完成，所以我们可以将 ResourceLoader 称作为统一资源定位器。如下是ResourceLoader源码定义：

```java
package org.springframework.core.io;

import org.springframework.lang.Nullable;

public interface ResourceLoader {
    //定义classpath的路径
    String CLASSPATH_URL_PREFIX = "classpath:";

    Resource getResource(String var1);

    @Nullable
    ClassLoader getClassLoader();
}
```

ResourceLoader 接口提供两个方法：`getResource()`、`getClassLoader()`。

* `getResource()` 根据所提供资源的路径 location 返回 Resource 实例，但是它不确保该 Resource 一定存在，需要调用 `Resource.exist()`方法判断。该方法支持以下模式的资源加载：
  * URL位置资源，如”file:C:/test.java”
  * ClassPath位置资源，如”classpath:test.java”
  * 相对路径资源，如”WEB-INF/test.java”，此时返回的Resource实例根据实现不同而不同

  该方法的主要实现是在其子类 DefaultResourceLoader 中实现，具体过程我们在分析 DefaultResourceLoader 时做详细说明。

* `getClassLoader()` 返回 ClassLoader 实例，对于想要获取 ResourceLoader 使用的 ClassLoader 用户来说，可以直接调用该方法来获取，

在分析 Resource 时，提到了一个类 ClassPathResource ，这个类是可以根据指定的 ClassLoader 来加载资源的。

作为 Spring 统一的资源加载器，它提供了统一的抽象，具体的实现则由相应的子类来负责实现，其类的类结构图如下：

#### DefaultResourceLoader

DefaultResourceLoader 是 ResourceLoader 的默认实现，它可以接收 ClassLoader 作为构造函数或者使用不带参数的构造函数，在使用带参构造函数时，需要传入一个ClassLoader，默认一般是 `Thread.currentThead().getContextClassLoader()`。源码细节如下：

```java
public class DefaultResourceLoader implements ResourceLoader {
    @Nullable
    private ClassLoader classLoader;
    private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet(4);
    private final Map<Class<?>, Map<Resource, ?>> resourceCaches = new ConcurrentHashMap(4);

    public DefaultResourceLoader() {
        this.classLoader = ClassUtils.getDefaultClassLoader();
    }

    public DefaultResourceLoader(@Nullable ClassLoader classLoader) {
        this.classLoader = classLoader;
    }
 ......   
}

public abstract class ClassUtils{
  @Nullable
    public static ClassLoader getDefaultClassLoader() {
        ClassLoader cl = null;

        try {
            //获取当前类线程的加载器
            cl = Thread.currentThread().getContextClassLoader();
        } catch (Throwable var3) {
        }

        if (cl == null) {
            cl = ClassUtils.class.getClassLoader();
            if (cl == null) {
                try {
                    cl = ClassLoader.getSystemClassLoader();
                } catch (Throwable var2) {
                }
            }
        }

        return cl;
    }
}
```

ResourceLoader 中最核心的方法为 `getResource()`,它根据提供的 location 返回相应的 Resource，而 DefaultResourceLoader 对该方法提供了核心实现(它的两个子类都没有提供覆盖该方法，所以可以断定ResourceLoader 的资源加载策略就封装 DefaultResourceLoader中)，如下：

```java
public class DefaultResourceLoader implements ResourceLoader {
    public Resource getResource(String location) {
        Assert.notNull(location, "Location must not be null");
        Iterator it = this.protocolResolvers.iterator();

        Resource resource;
        do {
            if (!it.hasNext()) {
                if (location.startsWith("/")) {
                    //此时location表示本地文件路径，将从文件路径获取资源
                    return this.getResourceByPath(location);
                }

                if (location.startsWith("classpath:")) {
                    //此时location特指classpath路径，将从类路径下获取资源
                    return new ClassPathResource(location.substring("classpath:".length()), this.getClassLoader());
                }

                try {
                    //此时location表示URL,将从网络获取资源
                    URL url = new URL(location);
                    return (Resource)(ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
                } catch (MalformedURLException e) {
                    return this.getResourceByPath(location);
                }
            }

            ProtocolResolver protocolResolver = (ProtocolResolver)it.next();
            resource = protocolResolver.resolve(location, this);
        } while(resource == null);

        return resource;
    }
}
```



源码中我们看到程序会首先尝试通过 ProtocolResolver 来加载资源，成功返回 Resource，否则调用如下逻辑：

- 若 location 以 / 开头，则调用 `getResourceByPath()`构造 ClassPathContextResource 类型资源并返回。
- 若 location 以 classpath: 开头，则构造 ClassPathResource 类型资源并返回，在构造该资源时，通过 `getClassLoader()`获取当前的 ClassLoader。
- 构造 URL ，尝试通过它进行资源定位，若没有抛出 MalformedURLException 异常，则判断是否为 FileURL , 如果是则构造 FileUrlResource 类型资源，否则构造 UrlResource。若在加载过程中抛出 MalformedURLException 异常，则委派 `getResourceByPath()` 实现资源定位加载。



`ProtocolResolver `，用户自定义协议资源解决策略，作为 DefaultResourceLoader 的 SPI，它允许用户自定义资源加载协议，而不需要继承 ResourceLoader 的子类。其定义如下：

```java
@FunctionalInterface
public interface ProtocolResolver {
    @Nullable
    Resource resolve(String var1, ResourceLoader var2);
}
```

在介绍 Resource 时，提到如果要实现自定义 Resource，我们只需要继承 DefaultResource 即可，但是有了 ProtocolResolver 后，我们不需要直接继承 DefaultResourceLoader，改为实现 ProtocolResolver 接口也可以实现自定义的 ResourceLoader。 ProtocolResolver 接口，仅有一个方法 `Resource resolve(String location, ResourceLoader resourceLoader)`，该方法接收两个参数：资源路径location，指定的加载器 ResourceLoader，返回为相应的 Resource 。在 Spring 中你会发现该接口并没有实现类，它需要用户自定义，自定义的 Resolver 如何加入 Spring 体系呢？调用 `DefaultResourceLoader.addProtocolResolver()` 即可，

#### ResourcePatternResolver

ResourceLoader 的 `Resource getResource(String location)` 每次只能根据 location 返回一个 Resource，当需要加载多个资源时，我们除了多次调用 `getResource()` 外别无他法。ResourcePatternResolver 是 ResourceLoader 的扩展，它支持根据指定的资源路径匹配模式每次返回多个 Resource 实例，其定义如下：

```java
public interface ResourcePatternResolver extends ResourceLoader {
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

    Resource[] getResources(String locationPattern) throws IOException;
}
```

ResourcePatternResolver 在 ResourceLoader 的基础上增加了 `getResources(String locationPattern)`，以支持根据路径匹配模式返回多个 Resource 实例，同时也新增了一种新的协议前缀 `classpath*:`，该协议前缀由其子类负责实现。

PathMatchingResourcePatternResolver 为 ResourcePatternResolver 最常用的子类，它除了支持 ResourceLoader 和 ResourcePatternResolver 新增的 classpath*: 前缀外，还支持 Ant 风格的路径匹配模式（类似于 `**/*.xml`）。

PathMatchingResourcePatternResolver 在实例化的时候，可以指定一个 ResourceLoader，如果不指定的话，它会在内部默认构造一个 DefaultResourceLoader。

```java
public class PathMatchingResourcePatternResolver implements ResourcePatternResolver {
    private final ResourceLoader resourceLoader;
  
    public PathMatchingResourcePatternResolver() {
        this.resourceLoader = new DefaultResourceLoader();
    }

    public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {
        Assert.notNull(resourceLoader, "ResourceLoader must not be null");
        this.resourceLoader = resourceLoader;
    }

    public PathMatchingResourcePatternResolver(@Nullable ClassLoader classLoader) {
        this.resourceLoader = new DefaultResourceLoader(classLoader);
    }
}  
```

重点看一下它的 `getResources(String locationPattern)` 方法是如何实现的：

```java
public Resource[] getResources(String locationPattern) throws IOException {
    //要求输入的路径匹配模式不能为空
    Assert.notNull(locationPattern, "Location pattern must not be null");
    //以 classpath*: 开头 
    if (locationPattern.startsWith("classpath*:")) {
        return this.getPathMatcher().isPattern(locationPattern.substring("classpath*:".length())) ? this.findPathMatchingResources(locationPattern) : this.findAllClassPathResources(locationPattern.substring("classpath*:".length()));
    } else {
        int prefixEnd = locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 : locationPattern.indexOf(58) + 1;
        return this.getPathMatcher().isPattern(locationPattern.substring(prefixEnd)) ? this.findPathMatchingResources(locationPattern) : new Resource[]{this.getResourceLoader().getResource(locationPattern)};
    }
}
```

程序执行流程如下：

![](https://www.cmsblogs.com/images/group/sike-java/sike-java-spring-ioc/202105092051327943.png)

**findAllClassPathResources**

当 locationPattern 以 `classpath*:` 开头但是不包含通配符，则调用`findAllClassPathResources()` 方法加载资源。该方法返回 classes 路径下和所有 jar 包中的所有相匹配的资源。

```java
 protected Resource[] findAllClassPathResources(String location) throws IOException {          
     String path = location;                                                                   
     if (location.startsWith("/")) {                                                           
         path = location.substring(1);                                                         
     }                                                                                         
                                                                                               
     Set<Resource> result = this.doFindAllClassPathResources(path);                            
     if (logger.isDebugEnabled()) {                                                            
         logger.debug("Resolved classpath location [" + location + "] to resources " + result);
     }                                                                                         
                                                                                               
     return (Resource[])result.toArray(new Resource[0]);                                       
 }                                                                                             
```

真正执行加载的 `doFindAllClassPathResources()`方法如下：

```java
protected Set<Resource> doFindAllClassPathResources(String path) throws IOException {                    
    Set<Resource> result = new LinkedHashSet(16);                                                        
    ClassLoader cl = this.getClassLoader();                                                              
    Enumeration resourceUrls = cl != null ? cl.getResources(path) : ClassLoader.getSystemResources(path);
                                                                                                         
    while(resourceUrls.hasMoreElements()) {                                                              
        URL url = (URL)resourceUrls.nextElement();                                                       
        result.add(this.convertClassLoaderURL(url));                                                     
    }                                                                                                    
                                                                                                         
    if ("".equals(path)) {                                                                               
        this.addAllClassLoaderJarRoots(cl, result);                                                      
    }                                                                                                    
                                                                                                         
    return result;                                                                                       
}                                                                                                        
```

`doFindAllClassPathResources()` 根据 ClassLoader 加载路径下的所有资源。在加载资源过程中如果，在构造 PathMatchingResourcePatternResolver 实例的时候如果传入了 ClassLoader，则调用其 `getResources()`，否则调用`ClassLoader.getSystemResources(path)`。如下是类加载器中的`getResources(String)`

```java
 public Enumeration<URL> getResources(String name) throws IOException {
     @SuppressWarnings("unchecked")
     Enumeration<URL>[] tmp = (Enumeration<URL>[]) new Enumeration<?>[2];
     if (parent != null) {
         tmp[0] = parent.getResources(name);
     } else {
         tmp[0] = getBootstrapResources(name);
     }
     tmp[1] = findResources(name);

     return new CompoundEnumeration<>(tmp);
 }
```

通过上面的分析，我们知道 `findAllClassPathResources()` 其实就是利用 ClassLoader 来加载指定路径下的资源，不管它是在 class 路径下还是在 jar 包中。如果我们传入的路径为空或者 `/`，则会调用 `addAllClassLoaderJarRoots()` 方法加载所有的 jar 包。

**findPathMatchingResources**

当 locationPattern 以 `classpath*:` 开头且当中包含了通配符，则调用该方法进行资源加载：

https://www.cmsblogs.com/article/1391375268060467201

