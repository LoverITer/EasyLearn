### Spring MVC—HTTP数据序列化和反序列化组件：HttpMessageConvertor

Java Web 开发人员经常要设计 RESTful API ，通过 json 数据进行交互。那么前端传入的 json 数据如何被解析成 Java 对象作为 API入参？API 返回结果又如何将 Java 对象解析成 json 格式数据返回给前端？在转换的过程我们可以加入哪些定制化内容？其实在整个数据流转过程中，**HttpMessageConverter** 起到了重要作用。



### 一、HttpMessageConverter 介绍

`org.springframework.http.converter.HttpMessageConverter` 是一个策略接口，源码中对此接口说明如下图所示：

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8A%E5%8D%889.46.33.png)

简单说就是 HTTP request (请求)和 response  (响应)的转换器。该接口有6个方法，简单来说就是获取支持的 MediaType（application/json之类），接收到请求时判断是否能读（canRead），能读则读（read）；返回结果时判断是否能写（canWrite），能写则写（write）。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8A%E5%8D%889.50.11.png)



### 二、 HttpMessageConertor 的缺省配置

我们写 Demo 时没有配置任何 MessageConverter，但是数据前后传递依旧好用，是因为 SpringMVC 启动时会自动配置一些HttpMessageConverter，在 `org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport` 类中添加了缺省 MessageConverter：

```java
protected final void addDefaultHttpMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
   messageConverters.add(new ByteArrayHttpMessageConverter());
   messageConverters.add(new StringHttpMessageConverter());
   messageConverters.add(new ResourceHttpMessageConverter());
   messageConverters.add(new ResourceRegionHttpMessageConverter());
   if (!shouldIgnoreXml) {
      try {
         messageConverters.add(new SourceHttpMessageConverter<>());
      }
      catch (Throwable ex) {
         // Ignore when no TransformerFactory implementation is available...
      }
   }
   messageConverters.add(new AllEncompassingFormHttpMessageConverter());

   if (romePresent) {
      messageConverters.add(new AtomFeedHttpMessageConverter());
      messageConverters.add(new RssChannelHttpMessageConverter());
   }

   if (!shouldIgnoreXml) {
      if (jackson2XmlPresent) {
         Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.xml();
         if (this.applicationContext != null) {
            builder.applicationContext(this.applicationContext);
         }
         messageConverters.add(new MappingJackson2XmlHttpMessageConverter(builder.build()));
      }
      else if (jaxb2Present) {
         messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
      }
   }

   if (kotlinSerializationJsonPresent) {
      messageConverters.add(new KotlinSerializationJsonHttpMessageConverter());
   }
   if (jackson2Present) {
      Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json();
      if (this.applicationContext != null) {
         builder.applicationContext(this.applicationContext);
      }
      messageConverters.add(new MappingJackson2HttpMessageConverter(builder.build()));
   } else if (gsonPresent) {
      messageConverters.add(new GsonHttpMessageConverter());
   } else if (jsonbPresent) {
      messageConverters.add(new JsonbHttpMessageConverter());
   }

   if (jackson2SmilePresent) {
      Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.smile();
      if (this.applicationContext != null) {
         builder.applicationContext(this.applicationContext);
      }
      messageConverters.add(new MappingJackson2SmileHttpMessageConverter(builder.build()));
   }
   if (jackson2CborPresent) {
      Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.cbor();
      if (this.applicationContext != null) {
         builder.applicationContext(this.applicationContext);
      }
      messageConverters.add(new MappingJackson2CborHttpMessageConverter(builder.build()));
   }
}
```

我们看到很熟悉的 **MappingJackson2HttpMessageConverter** ，如果我们引入 jackson 相关包，Spring 就会自动为我们添加该 MessageConverter，但是我们通常在搭建框架的时候还是会手动添加配置 MappingJackson2HttpMessageConverter，为什么？  答案还得在源码中寻找，如下还是请看WebMvcConfigurationSupport 类，类中有另一个方法： `getMessageConverters()` 

```java
protected final List<HttpMessageConverter<?>> getMessageConverters() { 
	if (this.messageConverters == null) {                              
		this.messageConverters = new ArrayList<>();
    //我们手动配置的MessageConverters
		configureMessageConverters(this.messageConverters);            
		if (this.messageConverters.isEmpty()) { 
      //如果手动配置的MessageConverters不存在，添加默认的
			addDefaultHttpMessageConverters(this.messageConverters);   
		}                                                              
		extendMessageConverters(this.messageConverters);               
	}                                                                  
	return this.messageConverters;                                     
}                                                                      
```

方法主要做的事情，从方法名就可以清楚得知——获取MessageConverters，同时方法逻辑也不难理解，就是首先会尝试加载我们手动配置的MessageConverters，如果不存在，则Spring MVC就不会再调用 `addDefaultHttpMessageConverters` 方法添加默认的MessageConverter了。



### 三、HtppMessageConvertor 数据流转解析

首先我们可以看一个例子：

```java
@RequestMapping("/test")
@ResponseBody
public String test(@RequestBody String param) {
  return "param '" + param + "'";
}
```

这是一段经典的Handler方法，在请求进入test方法前，会根据`@RequestBody`注解选择对应的`HttpMessageConverter`实现类来将请求参数解析到param变量中，因为这里的参数是String类型的，所以这里将会使用`StringHttpMessageConverter`，它的canRead()方法返回true，然后read()方法会从请求中读出请求参数，绑定到test()方法的param变量中。

同理当执行test方法后，由于返回值标识了`@ResponseBody`，SpringMVC / SpringBoot将使用`StringHttpMessageConverter`的write()方法，将结果作为String值写入响应报文，当然，此时canWrite()方法返回true。借用下图简单描述整个过程：

<img src="http://image.easyblog.top/src=http%253A%252F%252Fgmem.site%252Fwp-content%252Fuploads%252F2012%252F08%252Fhttpconv.png&refer=http%253A%252F%252Fgmem.site&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg" style="width:87%;" />



在Spring的处理过程中，一次请求报文和一次响应报文，分别被抽象为一个请求消息`HttpInputMessage`和一个响应消息`HttpOutputMessage`。

处理请求时，由合适的消息转换器将请求报文绑定为方法中的形参对象，在这里同一个对象就有可能出现多种不同的消息形式，如json、xml。同样响应请求也是同样道理。

在Spring中，针对不同的消息形式，有不同的HttpMessageConverter实现类来处理各种消息形式，至于各种消息解析实现的不同，则在不同的HttpMessageConverter实现类中。



接下来，我们翻看源码验证一下上述过程。

**（1）请求过程解析**

看 doDispatch 方法中的关键代码：

```java
// 这里的 Adapter 实际上是 RequestMappingHandlerAdapter
HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler()); 
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
    return;
}
// 实际处理的handler
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());            mappedHandler.applyPostHandle(processedRequest, response, mv);
```

从进入handle之后我将调用栈粘贴在此处：

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8A%E5%8D%8811.08.59.png)

这里重点说明调用栈最顶层 `readWithMessageConverters` 方法中内容：

```java
// 遍历 messageConverters
for (HttpMessageConverter<?> converter : this.messageConverters) {
    Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
    // 判断 converter 是否是 GenericHttpMessageConverter 类型
    if (converter instanceof GenericHttpMessageConverter) {
        GenericHttpMessageConverter<?> genericConverter = (GenericHttpMessageConverter<?>) converter;
        //判断是否可读 canRead
        if (genericConverter.canRead(targetType, contextClass, contentType)) {
            //有请求体
            if (inputMessage.getBody() != null) {
                //读取之前前置操作
                inputMessage = getAdvice().beforeBodyRead(inputMessage, parameter, targetType, converterType);
                //读取
                body = genericConverter.read(targetType, contextClass, inputMessage);
                //读取之后后置操作 
                body = getAdvice().afterBodyRead(body, inputMessage, parameter, targetType, converterType);
            } else {
                //处理空请求体
                body = getAdvice().handleEmptyBody(null, inputMessage, parameter, targetType, converterType);
            }
            break;
        }
    } else if (targetClass != null) {
        //不是 GenericHttpMessageConverter 类型
        if (converter.canRead(targetClass, contentType)) {
            if (inputMessage.getBody() != null) {
                inputMessage = getAdvice().beforeBodyRead(inputMessage, parameter, targetType, converterType);
                body = ((HttpMessageConverter<T>) converter).read(targetClass, inputMessage);
                body = getAdvice().afterBodyRead(body, inputMessage, parameter, targetType, converterType);
            }
            else {
                body = getAdvice().handleEmptyBody(null, inputMessage, parameter, targetType, converterType);
            }
            break;
        }
    }
}
```

最终判断是否canRead，能读就read，走到下面代码处将输入的内容反序列化出来：

```java
protected Object _readMapAndClose(JsonParser p0, JavaType valueType)
        throws IOException
    {
        try (JsonParser p = p0) {
            Object result;
            JsonToken t = _initForReading(p);
            if (t == JsonToken.VALUE_NULL) {
                // Ask JsonDeserializer what 'null value' to use:
                DeserializationContext ctxt = createDeserializationContext(p,
                        getDeserializationConfig());
                result = _findRootDeserializer(ctxt, valueType).getNullValue(ctxt);
            } else if (t == JsonToken.END_ARRAY || t == JsonToken.END_OBJECT) {
                result = null;
            } else {
                DeserializationConfig cfg = getDeserializationConfig();
                DeserializationContext ctxt = createDeserializationContext(p, cfg);
                JsonDeserializer<Object> deser = _findRootDeserializer(ctxt, valueType);
                if (cfg.useRootWrapping()) {
                    result = _unwrapAndDeserialize(p, ctxt, cfg, valueType, deser);
                } else {
                    result = deser.deserialize(p, ctxt);
                }
                ctxt.checkUnresolvedObjectId();
            }
            // Need to consume the token too
            p.clearCurrentToken();
            return result;
        }
    }

```



**（2）响应过程解析**

还是在上面 `ServletInvocableHandlerMethod` 的 invokeAndHandle 方法中，这次它调用了 `HandlerMethodReturnValueHandlerComposite` 的 handleReturnValue 方法并最终委托给了 `AbstractMessageConverterMethodProcessor` 来处理返回值 

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8B%E5%8D%881.28.27.png)

因此，我们还是重点关注调用栈顶层`writeWithMessageConverters`内容，是不是很熟悉的样子，完全一样的逻辑, 判断是否能写canWrite，能写则write：

```java
for (HttpMessageConverter<?> messageConverter : this.messageConverters) {
    if (messageConverter instanceof GenericHttpMessageConverter) {
        if (((GenericHttpMessageConverter) messageConverter).canWrite(
                declaredType, valueType, selectedMediaType)) {
            //write前置操作
            outputValue = (T) getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType, (Class<? extends HttpMessageConverter<?>>) messageConverter.getClass(),
                    inputMessage, outputMessage);
            if (outputValue != null) {
                addContentDispositionHeader(inputMessage, outputMessage);
                ((GenericHttpMessageConverter) messageConverter).write(
                        outputValue, declaredType, selectedMediaType, outputMessage);
            }
            return;
        }
    } else if (messageConverter.canWrite(valueType, selectedMediaType)) {
        outputValue = (T) getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType, (Class<? extends HttpMessageConverter<?>>) messageConverter.getClass(), inputMessage, outputMessage);
        if (outputValue != null) {
            addContentDispositionHeader(inputMessage, outputMessage);
            ((HttpMessageConverter) messageConverter).write(outputValue, selectedMediaType, outputMessage);
        }
        return;
    }
}
```

在代码中我们观察到有这么一处逻辑：

```java
body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
							(Class<? extends HttpMessageConverter<?>>) converter.getClass(),
							inputMessage, outputMessage);
```

追踪代码，可以看到在这个类中有个 `processBody` 方法：

```java
private <T> Object processBody(@Nullable Object body, MethodParameter returnType, MediaType contentType,
		Class<? extends HttpMessageConverter<?>> converterType,                                         
		ServerHttpRequest request, ServerHttpResponse response) {                                       
   
  //遍历ResponseBodyAdvice集合
	for (ResponseBodyAdvice<?> advice : getMatchingAdvice(returnType, ResponseBodyAdvice.class)) {  
    //如果supports方法返回true就使用此ResponseBodyAdvice处理一次返回值
		if (advice.supports(returnType, converterType)) {  
      //使用ResponseBodyAdvice的beforeBodyWrite处理返回逻辑
			body = ((ResponseBodyAdvice<T>) advice).beforeBodyWrite((T) body, returnType,               
					contentType, converterType, request, response);                                     
		}                                                                                               
	}                                                                                                   
	return body;                                                                                        
}                                                                                                       
```

**`ResponseBodyAdvice`**  它是个接口，根据源码给的注释可以了解到它可以支持让我们在执行了目标控制器handler方法之后，同时在MessageConvertor返回之前自定义响应体。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8B%E5%8D%881.43.27.png)

这个有什么用呢？我们在设计 RESTful API 接口的时候通常会将返回的数据封装成统一格式，一般常见的实现方式如下：

```java
@RestController("/demo")
public DemoController {
  
  @GetMapping("/{id}")
  public Result<DemoBean> demo(@PathVariable Long id){
       DemoBean data = ...... 
       ......
       return  Result.success(data);
  }
  
}


//RESTful统一响应体
public Result<T> {
  
  private Integer code;
  private String message;
  private boolean success;
  private T data;
  
  public Result(Integer code,String message,boolean success,T data){
    this.code=code;
    this.message=message;
    this,success=success;
    this.data=data;
  }
  
  
  public <T> static Result success(T data){
    return new Result(200,"OK",true,data);
  }
  
  public  static Result failed(){
    return new Result(500,"falied",false,null);
  }
}
```

这个方式看上去很完美，但我要说他不够优雅，因为用起来有点繁琐，因为每次都要写 `Result.success(data)` 类似的代码，其实我们可以吧这句代码再次封装，实现上面所说的 `ResponseBodyAdvice`，然后在`beforeBodyWrite` 方法中返回统一响应体，改造后如下：

```java
@RestController("/demo")
public DemoController {
  
  @GetMapping("/{id}")
  public DemoBean demo(@PathVariable Long id){
       DemoBean data = ...... 
       ......
       //这里方法直接就返回本身处理获得到的对象即可
       return  data;
  }
  
}
```

然后我们可以实现一下 `ResponseBodyAdvice` 接口，在接口中统一封装返回响应体：

```java
@ControllerAdvice
public class ResponseBodyAdviceWrapper implements ResponseBodyAdvice<Object> {

  //supports方法可以用于判断是否需要此ResponseBodyAdvice来处理
  //这里直接返回true
	@Override
	public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
		return true;
	}

  //body 就是handler方法处理后获得的返回值
	@Override
	public Object beforeBodyWrite(Object body, 
                                MethodParameter returnType, 
                                MediaType selectedContentType,
			Class<? extends HttpMessageConverter<?>> selectedConverterType, 
                                ServerHttpRequest request,
			ServerHttpResponse response) {
		if (body instanceof Result) {
			return body;
		}
    //响应成功，响应失败由全局异常处理器来处理
		return Result.success(data);
	}

}
```

这样封装之后就可以避免在controller中出现大面积 `Result.success(data)` 这样重复的代码。

整个处理流程就是这样，文章开头我们说过添加自己的 MessageConverter 能更好的满足我们的定制化，除了上面这个可以利用ResponseBodyAdvice自定义返回响应体的例子，MessageConvertor这块还有哪些可以定制的呢？

**（1）空值处理**

请求和返回的数据有很多空值，这些值有时候并没有实际意义，我们可以过滤掉和不返回，或设置成默认值。比如通过重写 `getObjectMapper` 方法，将返回结果的空值不进行序列化：

```java
converters.add(0, new MappingJackson2HttpMessageConverter(){
    @Override
    public ObjectMapper getObjectMapper() {
        super.getObjectMapper().setSerializationInclusion(JsonInclude.Include.NON_NULL);
                return super.getObjectMapper();
    }
}
```

**（2）预防XSS 脚本攻击**

为了保证输入的数据更安全，防止 XSS 脚本攻击，我们可以添加自定义反序列化器：

```java
//对应无法直接返回String类型
converters.add(0, new MappingJackson2HttpMessageConverter(){
    @Override
    public ObjectMapper getObjectMapper() {
        super.getObjectMapper().setSerializationInclusion(JsonInclude.Include.NON_NULL);

        // XSS 脚本过滤
        SimpleModule simpleModule = new SimpleModule();
        // 这里的 StringXssDeserializer 需要我们自己动手实现
        simpleModule.addDeserializer(String.class, new StringXssDeserializer());
        super.getObjectMapper().registerModule(simpleModule);

        return super.getObjectMapper();
    }
}
```

**（3）RequestBodyAdvice 实现入参统一处理**

举一反三，既然返回的时候有 ResponseBodyAdvice ，那么在请求读取的时候应该会有 RequestBodyAdvice。

是的，Spring MVC 的确提供了这个接口：

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2022-03-01%20%E4%B8%8B%E5%8D%882.28.22.png)

`beforeBodyRead` 和 `afterBodyReady` 在`read` 方法调用前后会被调用，`handleEmptyBody` 会在请求体是空时被调用。

在实际项目中 , 往往需要对请求参数做一些统一的操作 , 例如参数的过滤 , 字符的编码 , 第三方的解密等等 , Spring提供了RequestBodyAdvice一个全局的解决方案 , 免去了我们在Controller处理的繁琐 .。比如如下这个处理入参解密的操作：

```java
//对入参统一解密
@Slf4j
@RestControllerAdvice
public class ParamDecryptRequestBodyAdvice implements RequestBodyAdvice { 

    @Value("${private.clientKey}")
    private String clientKey;

    @Override
    public boolean supports(MethodParameter methodParameter, 
   							Type type, Class<? extends HttpMessageConverter<?>> aClass) {
   		  // 执行的前提是方法上有 RequestBody 和 Encrypt 两个注解			
        return methodParameter.hasParameterAnnotation(
        						RequestBody.class) && 
        						methodParameter.hasMethodAnnotation(Encrypt.class);
    }

    @Override
    public HttpInputMessage beforeBodyRead(HttpInputMessage httpInputMessage,
    										MethodParameter methodParameter,
    										Type type,
    										Class<? extends HttpMessageConverter<?>> aClass) throws IOException {
        
        return new HttpInputMessage() {
            @Override
            public InputStream getBody() throws IOException {
                try {
                    String inputString = IOUtils.inputStream2String(httpInputMessage.getBody());
                    log.info("接收到解密请求：{}",inputString);
                    try{
                        JSONObject jsonObject = JSON.parseObject(inputString);
                        if(jsonObject.containsKey("requsetData")){
                            String encryptStr = jsonObject.getString("requsetData");
                            String decryptStr = RSAUtils.decrypt(encryptStr,clientKey); 
                        }
                    }catch (JSONException e){
                        log.error("解密出错,JSON解析异常：{}",e); 
                        e.printStackTrace();
                    }catch (ClassCastException e){
                        log.error("解密出错,JSON解析异常：{}",e); 
                        e.printStackTrace();
                    }
                }catch (IOException e){
                    log.error("解密出错：{}",e);
                    e.printStackTrace();
                }
                return httpInputMessage.getBody();
            }

            @Override
            public HttpHeaders getHeaders() {
                return httpInputMessage.getHeaders();
            }
        };
    }

    @Override
    public Object afterBodyRead(Object o, 
    							HttpInputMessage httpInputMessage,
    							MethodParameter methodParameter,
    							Type type, 
    							Class<? extends HttpMessageConverter<?>> aClass) {
        return o;
    }

    @Override
    public Object handleEmptyBody(Object o, 
    							  HttpInputMessage httpInputMessage, 
    							  MethodParameter methodParameter, 
    							  Type type, 
    							  Class<? extends HttpMessageConverter<?>> aClass) {
        return o;
    } 
}
```









### 参考

* [HttpMessageConvertor是这样转换数据的](https://blog.csdn.net/yusimiao/article/details/90600316)
* [Spring HttpMessageConverter的作用及替换解析](https://www.jb51.net/article/134588.htm)

