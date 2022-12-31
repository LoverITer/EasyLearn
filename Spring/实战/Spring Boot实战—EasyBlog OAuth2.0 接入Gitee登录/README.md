### Spring Boot实战—EasyBlog OAuth2.0 接入Gitee登录



### 一、开发前准备

**（1）在 [修改资料](https://gitee.com/profile) -> [第三方应用](https://gitee.com/oauth/applications)，创建要接入码云的应用**。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220306003728.png)



**（2）填写应用相关信息，勾选应用所需要的权限。其中: 回调地址是用户授权后，码云回调到应用，并且回传授权码的地址**。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220306004149.png)



**（3）创建成功后，会生成 Cliend ID 和 Client Secret。他们将会在上述OAuth2 认证基本流程用到**。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220306004443.png)



### 二、实现第三方登录/认证接口

在[Spring Boot实战—EasyBlog OAuth2.0 接入GitHub登录](./Spring Boot实战—EasyBlog OAuth2.0 接入GitHub登录.md)一文中我们提出的架构，我们可以通过实现`IAuthService` 接口实现`GiteeAuthServiceImpl` 定制 Gitee 相关的第三方登录认证策略，并且在枚举类中添加此策略实现。

**（1）应用通过 浏览器 或 Webview 将用户引导到码云三方认证页面上（ GET请求 ）**

在前端页面放置 Gitee 授权链接，这我们同样在后端服务中统一维护这些三方信息，然后当用户跳转到我们系统的的登录界面时可以让前端动态请求系统后端服务生成链接：

```java
@Slf4j
@Service
public class GiteeAuthServiceImpl implements IAuthService<GiteeAuthBean>, IOauthService {

    @Autowired
    private GiteeClient giteeClient;

    @Autowired
    private GiteeAuthProperties giteeAuthProperties;

    @Autowired
    private ILoginService loginService;

    //获取授权链接
    @Override
    public AuthorizationBean authorize(OauthRequest request) {
        String authorizationUr = String.format("%s?client_id=%s&redirect_uri=%s&response_type=code", giteeAuthProperties.getAuthorizeUrl(),
                giteeAuthProperties.getClientId(), giteeAuthProperties.getRedirectUrl());
        return AuthorizationBean.builder().authorizationUrl(authorizationUr).build();
    }
}
```

**（2）获取AccessToken**

应用服务器 或 Webview 使用 access_token API 向 码云认证服务器发送post请求传入 用户授权码 以及 回调地址，**注：请求过程建议将 client_secret 放在 Body 中传值，以保证数据安全。**同时，这里注意一个小坑：码云第三方验证需要加上如下header信息，否则会报403错误：

```javascript
"User-Agent","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36)"
```

使用HttpClient等工具的的话可以参考如下操作：

```java
public Stirng getAccessToken(){
    String url = "https://gitee.com/oauth/token?grant_type=authorization_code&code="+code+"&client_id="+CLIENT_ID+"&redirect_uri="+REDIRECTURI+"&client_secret="+CLIENT_SECRET;
    Map<String,String> map = new HashMap<>();
    map.put("User-Agent","Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36)");
    JSONObject s = HttpUtils.post(url,map);
    
     //....
}
```

使用Feign的话，可以直接使用`@Header` 注解：

```java
@FeignClient(name = "gitee", url = "${urls.gitee}", configuration = CommonFeignConfig.class)
public interface GiteeClient {

    /**
     * 获取Gitee access_token
     *
     * @param request
     * @return
     */
    @PostMapping(value = "/oauth/token")
    @Headers({"User-Agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36)"})
    GiteeAuthTokenBean getAccessToken(@RequestBody QueryGiteeAuthTokenRequest request);
}
```

本系统集成了Feign，获取 access_token 的实现如下：

```java
@Slf4j
@Service
public class GiteeAuthServiceImpl implements IAuthService<GiteeAuthBean>, IOauthService {

    @Autowired
    private GiteeClient giteeClient;

    @Autowired
    private GiteeAuthProperties giteeAuthProperties;

    @Autowired
    private ILoginService loginService;

    //获取access_token
    @Override
    public String getAccessToken(String code) {
        GiteeAuthTokenBean accessToken = giteeClient.getAccessToken(QueryGiteeAuthTokenRequest.builder()
                .clientId(giteeAuthProperties.getClientId())
                .clientSecret(giteeAuthProperties.getClientSecret())
                .code(code)
                .grantType(LoginConstants.COMMON_GRANT_TYPE)
                .redirectUri(giteeAuthProperties.getRedirectUrl())
                .build());
        return Optional.ofNullable(accessToken).map(item -> {
            String token = item.getAccessToken();
            log.info("Get Gitee access_token: {}", JsonUtils.toJSONString(accessToken));
            return token;
        }).orElseThrow(() -> new BusinessException(ResultCode.REQUEST_GITEE_ACCESS_TOKEN_FAILED));
    }
}
```

**（3）获取码云用户信息**

通过授权码获取到的json数据，其中access_token参数，可以访问码云的用户数据，这里依然使用Feign来作为HttpClient：

```java
@Slf4j
@Service
public class GiteeAuthServiceImpl implements IAuthService<GiteeAuthBean>, IOauthService {

    @Autowired
    private GiteeClient giteeClient;

    @Autowired
    private GiteeAuthProperties giteeAuthProperties;

    @Autowired
    private ILoginService loginService;

     @Override
    public GiteeAuthBean getUserInfo(String accessToken) {
        GiteeAuthBean giteeUserInfo = giteeClient.getGiteeUserInfo(accessToken);
        return Optional.of(giteeUserInfo).map(item -> {
            log.info("Get gitee user info: {}", JsonUtils.toJSONString(item));
            return item;
        }).orElseThrow(() -> new BusinessException(ResultCode.REQUEST_GITEE_USER_INFO_FAILED));
    }
}
```

**（4）回调处理**

```java
@Slf4j
@Service
public class GiteeAuthServiceImpl implements IAuthService<GiteeAuthBean>, IOauthService {

    @Autowired
    private GiteeClient giteeClient;

    @Autowired
    private GiteeAuthProperties giteeAuthProperties;

    @Autowired
    private ILoginService loginService;

    @Override
    public AuthorizationBean callback(AuthCallbackRequest callback) {
        String accessToken = getAccessToken(callback.getCode());
        return Optional.ofNullable(accessToken).map(token -> {
            GiteeAuthBean userInfo = getUserInfo(token);
            AuthenticationDetailsBean authenticationDetailsBean = loginService.register(RegisterUserRequest.builder()
                    .identifierType(IdentifierType.GITEE.getSubCode())
                    .identifier(userInfo.getId())
                    .build());
            return AuthorizationBean.builder().user(authenticationDetailsBean.getUser()).build();
        }).orElseThrow(() -> new BusinessException(ResultCode.REQUEST_GITEE_ACCESS_TOKEN_FAILED));
    }
}
```



ok，今天的分享到这里介绍了，感谢大家点赞评论，如有不足之处，还望各位大大批评指正。另外，本文所有代码可以在[GitHub仓库](https://github.com/easyblog-org/User)获得，感兴趣的小伙伴可以pull下来研究研究~~

### 参考

* [原生java代码实现码云第三方验证登录](https://blog.csdn.net/qq_40565265/article/details/105103701?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3.pc_relevant_default&utm_relevant_index=6)
* [码云的第三方应用在获取Token时服务端响应状态403是什么情况？](https://gitee.com/oschina/git-osc/issues/IDBSA)
* [码云OpenApi OAuth 说明文档](https://gitee.com/api/v5/oauth_doc#/)

