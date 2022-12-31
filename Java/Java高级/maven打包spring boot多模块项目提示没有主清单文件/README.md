# maven打包Spring Boot多模块项目提示没有主清单文件



项目打包为Jar后，通过java -jar xxxxx.jar运行时提示xxxxx.jar中没有主清单属性，如下：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220404214013.png)

打开jar包，META-INF目录下的MANIFEST.MF，内容如下：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220404214124.png)



我们发现这里没有主类等信息，是什么原因导致的呢？网上大多数资料指出需要在pom.xml中配置[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)插件，如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

这种解决方案通常可以解决大部分问题，但这种方案只在使用 `spring-boot-starter-parent` 为 `<parent/>` 标签内容时才有效，当我们使用自定义的`<parent/>`节点时按如上所述的方式配置maven插件则是无效的，这是为什么呢？让我们一起看一看 spring-boot-starter-parent 中的配置。

**spring-boot-starter-parent** 中maven插件的配置如下：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220404214545.png)

我们可以看到这里配置了主类信息以及一个重要的标签`<goal>`，对repackage的描述如下：`Repackages existing JAR and WAR archives so that they can be executed from the command line using  java -jar. ` 看到这里我们就清楚了，当使用自定义的 parent 时，我们需要自行配置maven插件的`<goal>`属性，如下：

```xml
<build>                                                      
    <plugins>                                                
        <plugin>                                             
            <groupId>org.springframework.boot</groupId>      
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.2.6.RELEASE</version>                 
            <executions>                                     
                <execution>                                  
                    <goals>                                  
                        <goal>repackage</goal>               
                    </goals>                                 
                </execution>                                 
            </executions>                                    
        </plugin>                                            
    </plugins>                                               
</build>                                                     
```

或者指定 `mainClass`，如下：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.2.6.RELEASE</version>
    <configuration>
        <mainClass>top.easyblog.titan.Application</mainClass>
    </configuration>
    <executions>
        <execution>
            <id>repackage</id>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

此时我们再次使用`mvn clean package`指令打包jar包后看一下清单文件，内容如下：

![](C:/Users/HuangXin/Desktop/QQ%E6%88%AA%E5%9B%BE20220404222305.png)

这样项目就打包成功了，通过`java -jar`也可以正确运行了。

### 参考

* [maven打包spring boot多模块项目提示没有主清单文件](https://blog.csdn.net/xiaolegeaizy/article/details/108597125)

