# API接口签名验证



系统从外部获取数据时，通常采用API接口调用的方式来实现。请求方和接口提供方之间的通信过程，有这几个问题需要考虑：

1. **请求参数是否被篡改；**
2. **请求来源是否合法；**
3. **请求是否具有唯一性；**

今天跟大家探讨一下主流的通信安全解决方案。



## 参数签名方式

这种方式是主流。它要求调用方按照约定好的算法生成签名字符串，作为请求的一部分，接口提供方验算签名即可知是否合法。步骤通常如下：

1. 接口提供方给出app_id和app_secret
2. 调用方根据app_id和app_secret以及请求参数，按照一定算法生成签名sign
3. 接口提供方验证签名

生成签名的步骤如下：

1. 将所有业务请求参数按字母先后顺序排序
2. 参数名称和参数值链接成一个字符串A
3. 在字符串A的首尾加上app_secret组成一个新字符串B
4. 对字符串进行md5得到签名sign

假设请求的参数为：f=1,b=23,k=33，排序后为b=23,f=1,k=33，参数名和参数值链接后为b23f1k33，首尾加上app_secret后md5：
md5(secretkey1value1key2value2...secret)。

以上签名方法安全有效地解决了参数被篡改和身份验证的问题，如果参数被篡改，没事，因为别人无法知道app_secret，也就无法重新生成新的sign。
这里使用了md5的算法进行签名，也可以自行选择其他签名方式，例如RSA，SHA等。



## 请求唯一性保证

md5签名方法可以保证来源及请求参数的合法性，但是请求链接一旦泄露，可以反复请求，对于某些拉取数据的接口来说并不是一件好事，相当于是泄露了数据。
在请求中带上时间戳，并且把时间戳也作为签名的一部分，在接口提供方对时间戳进行验证，只允许一定时间范围内的请求，例如1分钟。因为请求方和接口提供方的服务器可能存在一定的时间误差，建议时间戳误差在5分钟内比较合适。允许的时间误差越大，链接的有效期就越长，请求唯一性的保证就越弱。所以需要在两者之间衡量。



## 秘钥的保存　

在签名的过程中，起到决定性作用之一的是app_secret，因此如何保存成为关键。我们分类讨论。
接口调用方的代码跑在服务器的情况比较好办，除非服务器被攻陷，否则外接无法知道app_secret，当然，要注意不能往日志里写入app_secret的值，其他敏感值也禁止写入日志，如账号密码等信息。
假如是客户端请求接口，就需要多想一步了。假如把app_secret硬编码到客户端，会有反编译的风险，特别是android。可以在客户端登陆验证成功后，返回给客户端的信息中带上app_secret(当然，返回的数据也可能被拦截，真是防不胜防啊。。。)。特别说明一下，在android开发中，假如硬要把app_secret硬编码，建议把app_secret放到NDK中编译成so文件，app启动后去读取。





## 最总方案签名规则

　　1、线下分配**app_id**和app_secret，针对不同的调用方分配不同的app_id和app_secret

　　2、加入**timestamp**（时间戳），10分钟内数据有效

　　3、加入流水号**nonce**（防止重复提交），至少为10位。针对查询接口，流水号只用于日志落地，便于后期日志核查。 针对办理类接口需校验流水号在有效期内的唯一性，以避免重复请求。

　　4、加入**signature**，所有数据的签名信息。

　　以上字段放在请求头中。**signature** 字段生成规则如下：

### **数据部分**

* **Path：**按照path中的顺序将所有value进行拼接
* **Query**：按照key字典序排序，将所有key=value进行拼接
* **Form**：按照key字典序排序，将所有key=value进行拼接
* **Body**：
  * Json: 按照key字典序排序，将所有key=value进行拼接（例如{"a":"a","c":"c","b":{"e":"e"}} => a=ab=e=ec=c）
  * String: 整个字符串作为一个拼接

如果存在多种数据形式，则按照path、query、form、body的顺序进行再拼接，得到所有数据的拼接值。上述拼接的值记作 Y。

### **请求头部分**

X=”appid=xxxnonce=xxxtimestamp=xxx”

### **生成签名**

最终拼接值=XY

最后将最终拼接值按照如下方法进行加密得到签名。

**signature=**org.apache.commons.codec.digest.HmacUtils.hmacSha256Hex(app_secret, 拼接的值);



## 签名算法实现

### **指定哪些接口或者哪些实体需要进行签名**

```java
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target({TYPE, METHOD})
@Retention(RUNTIME)
@Documented
public @interface Signature {
    /**
     * 按照order值排序
     */
    String ORDER_SORT = "ORDER_SORT";
    /**
     * 字典序排序
     */
    String ALPHA_SORT = "ALPHA_SORT";

    /**
     * 是否允许重复请求
     */
    boolean resubmit() default true;

    String sort() default Signature.ALPHA_SORT;
}
```

### 指定哪些字段需要进行签名

```java
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target({FIELD})
@Retention(RUNTIME)
@Documented
public @interface SignatureField {
    //签名顺序
    int order() default 0;

    //字段name自定义值
    String customName() default "";

    //字段value自定义值
    String customValue() default "";
}
```

### 核心算法

