API接口签名验证



系统从外部获取数据时，通常采用API接口调用的方式来实现。请求方和接口提供方之间的通信过程，有这几个问题需要考虑：

1. **请求参数是否被篡改；**
2. **请求来源是否合法；**
3. **请求是否具有唯一性；**

今天跟大家探讨一下主流的通信安全解决方案。



## 一、参数签名方式

这种方式是主流。它要求调用方按照约定好的算法生成签名字符串，作为请求的一部分，接口提供方验算签名即可知是否合法。步骤通常如下：

1. 接口提供方给出`app_id`和`app_secret`
2. 调用方根据`app_id`和`app_secret`以及请求参数，按照一定算法生成签名`sign`
3. 接口提供方验证签名

**生成签名的步骤如下**：

1. 将所有业务请求参数按字母先后顺序排序
2. 参数名称和参数值链接成一个字符串A
3. 在字符串A的首尾加上`app_secret`组成一个新字符串B
4. 对字符串使用摘要算法得到签名`sign`

假设请求的参数为：f=1,b=23,k=33，排序后为b=23,f=1,k=33，参数名和参数值链接后为b=23&f=1&k=33，首尾加上`app_secret`后SHA256：`SHA256(secretkey1value1key2value2...secret)`。

以上签名方法安全有效地解决了参数被篡改和身份验证的问题，如果参数被篡改，没事，因为别人无法知道`app_secret`，也就无法重新生成新的sign。
这里不建议md5的算法进行签名，这个算法现在已经被我国王小云教授破解了，可以自行选择其他签名方式，例如RSA，SHA等。



## 二、请求唯一性保证

md5签名方法可以保证来源及请求参数的合法性，但是请求链接一旦泄露，可以反复请求，对于某些拉取数据的接口来说并不是一件好事，相当于是泄露了数据。
在请求中带上时间戳，并且把时间戳也作为签名的一部分，在接口提供方对时间戳进行验证，只允许一定时间范围内的请求，例如1分钟。因为请求方和接口提供方的服务器可能存在一定的时间误差，建议时间戳误差在5分钟内比较合适。允许的时间误差越大，链接的有效期就越长，请求唯一性的保证就越弱。所以需要在两者之间衡量。



## 三、秘钥的保存　

在签名的过程中，起到决定性作用之一的是app_secret，因此如何保存成为关键。我们分类讨论。
接口调用方的代码跑在服务器的情况比较好办，除非服务器被攻陷，否则外接无法知道app_secret，当然，要注意不能往日志里写入app_secret的值，其他敏感值也禁止写入日志，如账号密码等信息。
假如是客户端请求接口，就需要多想一步了。假如把app_secret硬编码到客户端，会有反编译的风险，特别是android。可以在客户端登陆验证成功后，返回给客户端的信息中带上app_secret(当然，返回的数据也可能被拦截，真是防不胜防啊。。。)。特别说明一下，在android开发中，假如硬要把app_secret硬编码，建议把app_secret放到NDK中编译成so文件，app启动后去读取。





## 四、签名规则

1、线下分配`app_id`和`app_secret`，针对不同的调用方分配不同的`app_id`和`app_secret`

2、加入`timestamp`（时间戳），10分钟内数据有效

3、加入流水号**nonce**（防止重复提交），至少为10位。针对查询接口，流水号只用于日志落地，便于后期日志核查。 针对办理类接口需校验流水号在有效期内的唯一性，以避免重复请求。

4、加入**signature**，所有数据的签名信息。

以上字段放在请求头中。**signature** 字段生成规则如下：

### 4.1 数据部分

* **Path：**按照path中的顺序将所有value进行拼接
* **Query**：按照key字典序排序，将所有key=value进行拼接
* **Form**：按照key字典序排序，将所有key=value进行拼接
* **Body**：
  * Json: 按照key字典序排序，将所有key=value进行拼接（例如{"a":"a","c":"c","b":{"e":"e"}} => a=ab=e=ec=c）
  * String: 整个字符串作为一个拼接

如果存在多种数据形式，则按照path、query、form、body的顺序进行再拼接，得到所有数据的拼接值。上述拼接的值记作 Y。

### 4.2 请求头部分

X=”app_id=xxx&nonce=xxx&timestamp=xxx”

### 4.3 生成签名

最终拼接值=XY

最后将最终拼接值按照如下方法进行加密得到签名。

**signature=**org.apache.commons.codec.digest.HmacUtils.hmacSha256Hex(app_secret, 拼接的值);



## 五、签名算法实现

### 5.1 自定义拦截器`SignInterceptor`

实现spring提供的拦截器接口`HandlerInterceptor` ，在自定义拦截器中拦截请求并解析请求的有关参数封装成`SignEntity`，获取到`SignEntity`之后将验签实体对象交给`SignHandler`再次按照定义的规则进行验签计算，之后对比两次验签值，结果一致放行；否则不一致抛出验签不符合法异常。

```java
@Slf4j
@Component
public abstract class SignInterceptor implements HandlerInterceptor {

    //由子类实现将SignHandler实现类注入
    public abstract SignHandler handler();

    @Override
    public boolean preHandle(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response, @NonNull Object handler) throws Exception {
        try {
            return handler().checkSign(getRequestEntity(request));
        } catch (Throwable e) {
            log.error("SignInterceptor>>>>>>>>{}", e.getMessage());
            throw e;
        }
    }


    /**
     * 解析出请求信息并封装成SignEntity {@link top.easyboot.titan.sign.SignEntity}
     *
     * @param request
     * @return
     */
    public SignEntity getRequestEntity(HttpServletRequest request) {
        SignEntity signEntity = SignEntity.builder().build();
        signEntity.setMethod(request.getMethod());
        signEntity.setPath(request.getRequestURL().toString());
        //获取parameters（对应@RequestParam）
        Map<String, String[]> parameterMap = request.getParameterMap();
        if(CollectionUtils.isEmpty(parameterMap)){
            parameterMap=Maps.newHashMap();
        }
        signEntity.setPathParams(parameterMap);
        return signEntity;
    }

}
```

抽象类无法被实例化，我们需要提供一个子类`CommonSignInterceptor` ,在子类中从Spring容器中获取到SignHanlder并提供给父类，这里将SignHandler提供给子类实现也是为了可以实现对验签算法（SignHandler）的快速替换，当需要不同的验签规则时，你可以直接继承`SignInterceptor`并提供不同的SignHandler即可：

```java
@Component
public class CommonSignInterceptor extends SignInterceptor{

    @Autowired
    private SignHandler signHandler;

    @Override
    public SignHandler handler() {
        return signHandler;
    }
}
```



### 5.2 定义`SignHandler`—验签规则的落地

SignHandler我们将它定义成接口的形式，提供了4个接口：

* `String getSecret()`：用于获取本系统的secret，用于在本系统远程调用其他系统的使用进行验签计算
* `String getAppId()`：用户获取本系统的app_id，用于在本系统远程调用其他系统的使用进行验签计算
*  `boolean checkSign(SignEntity)`：用于对请求传递的验签相关的参数进行校验，比如关键验签参数判空校验，timestamp时间校验，以及最重要的验签值比对
* `String generateSign(SignEntity)`：验签规则的最终实现场所，对请求中的各种参数按照规定的验签规则拼接并使用SHA256生成验签值

其中`checkSign(SignEntity)` 和 `generateSign(SignEntity)` 提供默认实现，核心代码实现如下：

```java
public interface SignHandler {

    default String getSecret() {
        throw new IllegalArgumentException("secret can not be empty");
    }

    default String getAppId() {
        throw new IllegalArgumentException("appId can not be empty");
    }

    /**
     * 根据请求参数重新计算验签值
     * 验签字符串拼接规则：${请求方法} ${请求路径}?appId=${appId}&secret=${secret}&timestamp=${timestamp}&param1=${value1}....
     *
     * @param signEntity
     * @return
     */
    default String generateSign(SignEntity signEntity) {
        StringBuilder appender = new StringBuilder(signEntity.getMethod() + " " + signEntity.getPath() + "?");
        Map<String, String[]> pathParams = signEntity.getPathParams();
        if (!CollectionUtils.isEmpty(pathParams)) {
            pathParams.entrySet().stream().filter(entry -> !Constants.SIGN.equalsIgnoreCase(entry.getKey()))
                    .sorted(Map.Entry.comparingByKey())
                    .forEach(entry -> {
                String paramValue = String.join(Constants.COMMA, Arrays.stream(entry.getValue()).sorted().toArray(String[]::new));
                appender.append(entry.getKey()).append(Constants.EQUAL_SIGN).append(paramValue).append(Constants.DELIMETER);
            });
        }
        return SignUtils.sign(appender.substring(0, appender.length() - 1));
    }

    default boolean checkSign(SignEntity signEntity) throws Exception {
        if (Objects.isNull(signEntity)) {
            throw new BusinessException(ResultCode.SIGN_FAIL);
        }
        Map<String, String[]> params = signEntity.getPathParams();
        String[] requestTimestamp = params.get(Constants.TIMESTAMP);
        if (null==requestTimestamp||requestTimestamp.length==0||StringUtils.isBlank(requestTimestamp[0])) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "invalid request");
        }
        long timestamp = Long.parseLong(requestTimestamp[0]);
        long now = new Date().getTime();
        if (now - timestamp > Constants.TEN_MINUS) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "sign expired");
        }
        String[] appId = params.get(Constants.APP_ID);
        if (null==appId||appId.length==0||StringUtils.isBlank(appId[0])) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "invalid request");
        }
        String[] secret = params.get(Constants.SECRET);
        if (null==secret||secret.length==0||StringUtils.isBlank(secret[0])) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "invalid request");
        }
        String[] oldSign = params.get(Constants.SIGN);
        if (null==oldSign||oldSign.length==0||StringUtils.isBlank(oldSign[0])) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "invalid request");
        }
        String newSign = generateSign(signEntity);
        if (!oldSign[0].equals(newSign)) {
            throw new BusinessException(ResultCode.SIGN_FAIL, "sign invalid");
        }
        return true;
    }

}
```



同样接口也是无法实例化的，我们需要提供一个实现类`CommonSignHandler` 并交给Spring管理：

```java
@Component
public class CommonSignHandler implements SignHandler{


    @Value("${application.easyblig-cli.app_id:app_id}")
    private String appId;
    @Value("${application.easyblig-cli.secret:secret}")
    private String secret;


    @Override
    public String getSecret() {
        return this.secret;
    }

    @Override
    public String getAppId() {
        return this.appId;
    }
}
```



完成上面事情之后，我们就来测试一把，这里以Poatman调用接口为例：

* **不输入任何验签参数，直接请求接口**

  <img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211121114901.png" style="width:80%;" />

* **输入验签值不正确**

  <img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211121115143.png" style="width:80%;" />

* **输入正确的验签值**

  <img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211121115337.png" style="width:80%;" />

  

在上面的测试中我们发现，手动填写验签参数以及计算验签值非常麻烦，于是追求效率的我模拟Java代码中的验签规则，实现了如下Postman脚手架签名脚本，脚本兼容Js语法，我相信看到本文的亲~的都可以看懂：

```js
var app_id="10086";
var secret="a5fbe495127e41da9c2b7f7f6609e39c";
var timestamp=new Date().getTime();


var generateSign=function(){
   var signEntity = getSignEntity();
   var url=signEntity.method+" http://"+signEntity.path+"?";
   var pathParams=signEntity.pathParams;
   if(undefined!=pathParams && null!=pathParams){
       Object.keys(pathParams).sort().map((key)=>{
        var val = pathParams[key];
        if(key=="app_id"){
          val=app_id;
        }else if(key=="secret"){
          val=secret;
        }else if(key=="timestamp"){
          val= timestamp;
        }
         url += key + '=' + val + '&';
       });
   }
   console.log(url.substring(0,url.length-1));
   return CryptoJS.SHA256(url.substring(0,url.length-1)).toString(); 
}

var getSignEntity=function(){
   var signEntity={
       method:request.method,
       path:"",
       bodyParams:{},
       pathParams:{},
       headers:{}
    }
   var url=request.url;
   if(undefined == url || url.length==0){
       throw "url is empty";
   }
   if(url.indexOf("?")!=-1){
     url=url.substring(0,url.indexOf("?"));
   }
   //get请求参数
   var queryParam = pm.request.url.query.members;
   var pathParams = {};
   for (let i in queryParam) {
       var key=queryParam[i].key;
      if(key!="sign"){
         pathParams[key]=queryParam[i].value;
      }
   }
   signEntity.path=url;
   signEntity.pathParams=pathParams;
   return signEntity;
}

pm.environment.set("app_id", app_id);
pm.environment.set("secret", secret);
pm.environment.set("timestamp", timestamp);
pm.environment.set("sign",generateSign());
```



在使用时我们只需将验签参数的值使用 类似`key={{key}}` 占位符表示，之后脚本会自动在发送请求之前填充对应的值：

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211121115819.png" style="width:80%;" />

测试一把，发现没有问题：

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211121115940.png" style="width:80%;" />



### 5.3 定义`FeignSignHandler`—调用其他系统接口自动生成验签值

上面只是实现了系统对第三方调用的验签值验证，在当前微服务大行其道的趋势下，系统间调用也是非常常见的，这就需要我们系统在调用其他系统时也可以自动计算验签值

