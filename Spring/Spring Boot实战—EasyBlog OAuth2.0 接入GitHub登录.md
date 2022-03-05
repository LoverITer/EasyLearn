### Spring Boot实战—EasyBlog OAuth2.0 接入GitHub登录



### 一、OAuth2.0 简介

OAuth： OAuth（开放授权）是一个开放标准，允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方网站或分享他们数据的所有内容。

OAuth2.0 ：对于用户相关的 OpenAPI（例如获取用户信息，动态同步，照片，日志，分享等），为了保护用户数据的安全和隐私，第三方网站访问用户数据前都需要显式的向用户征求授权。



### 二、开发前准备

开发前我们需要在GitHub注册一个App-ID

**（1）创建应用**

登录 github 后台（地址：https://github.com/settings/developers ），创建应用你需要添加GitHub登录的应用。或者在GitHub的`settings` 中点击`Deveploper settings`

![img](http://image.easyblog.top/15909311367618577f19b-7b06-49fe-9a5e-525044c0b8c0.png)

点击`New GitHub App` 创建一个OAuth App

![img](http://image.easyblog.top/159093129295504956a25-bd8c-4a59-b06f-892a9a93bc2b.png)

**（2）填写你的App有关的信息**

![img](http://image.easyblog.top/159093158972141a2e657-8060-472e-97ec-86911a0ab230.png)

在注册成功之后，GitHub 会返回客户端 ID（client ID）和客户端密钥（client secret），这就是应用的身份识别码，在授权的时候会用到。如下是一个填写示例：

![img](http://image.easyblog.top/1590931756749f11a071c-ee88-42a6-89b9-29cf15e85ce8.png)



### 三、设计实现

这里就直接贴出Github第三方登录文档地址：[Github第三方登录文档](https://developer.github.com/apps/building-github-apps/identifying-and-authorizing-users-for-github-apps/) 英文好的小伙伴可以直接观看原生文档，就是一个单点登录，有学习和了解过基础的原理的看起来应该不算是困难。

#### 1、数据表设计

第三方授权用户的信息最终是需要存储在我们的系统中的，不然我们没有办法知道用户这个来自第三方系统的用户是否已经在我们自己的系统注册过，同时我们的系统在运营过程中肯定不会只接入GitHub一家第三方，因此综合上述考虑，我们可以在系统中维护一张Account表用于专门维护用户的账户信息，具体表结构如下：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220305225320.png)

其中，比较重要的是 `user_id`、`identity_type`、`identitifier` 三个字段 ：

* `user_id`：用于标识（绑定）第三方账户到系统用户ID
* `identity_type`：账号类型，比如：1 标识系统账户名昵称注册用户、2 表示手机注册用户、3 表示GitHub用户......
* `identifier`：对应账号，如果是第三方账号就存第三方账号的open_id即可，如果是系统注册的用户名密码用户，那么可以配合将`credential` 作为用户密码使用。
* `status`：用户账号状态是否正常，如果不正常的账户可以限制不予登录。
* `verified`：可以在用户使用邮箱号注册、手机号注册等场景下对用户的账号进行验证，验证通过了才可以允许用户使用此账号。

#### 2、抽象系统登录接口

随着后期系统接的越来越多的时候，即在数据库层面我们有着相对比较完善的数据表设计，如果我们仅仅是在接入一个系统后基于 `identity_type` 判断一下登录类型然后加一个 `if` 分支，这样面对数量众多登录方式，系统也是难以维护的，因此我们用软件工程的办法优化设计思路：

**基于策略模式抽象出一个第三方认证/登录接口，然后每种登录方式实现此接口**。但是这样一种设计还是比较难以使用使用，因为调用层必须要知道有哪些策略，并且还需要自己实例化此策略，比较麻烦，因此我们还**需要让配合工厂模式，即让工厂帮助上层调用者生产策略**，而调用者只需要传入 `identity_type` 接口。同时为了避免我们把 if-else 这样的结构引入到 工厂类中，我们可以定义一个枚举，枚举有两个重要的域，一个是 `identity_type`  ，另一个对应的登录策略的全类名 calssName，这样我们就可以**借助 Java 的反射机制**在代码运行中根据 `identity_type`  动态实例化策略类并提供给调用方使用了，**现在这个工厂类也“解放”出来了，并且策略、调用者也相互隔离了，他们都可以自由扩展，相互不受影响**。具体实现如下：

**（1）抽象第三方认证/登录接口**

```java
public interface IAuthService<T> {
    /**
     * 根据code获得Token
     *
     * @param code code
     * @return token
     */
    String getAccessToken(String code);

    /**
     * 根据Token获得OpenId
     *
     * @param accessToken Token
     * @return openId
     */
    String getOpenId(String accessToken);

    /**
     * 刷新Token
     *
     * @param code code
     * @return 新的token
     */
    String refreshToken(String code);


    /**
     * 根据Token和OpenId获得用户信息
     *
     * @param accessToken Token
     * @return 第三方应用给的用户信息
     */
    T getUserInfo(String accessToken);
    
    /**
     * 第三方登录获取 AccessToken
     *
     * @param request
     * @return
     */
    AuthorizationBean authorize(OauthRequest request);

    /**
     * 第三方授权回调
     *
     * @param callback
     * @return
     */
    AuthorizationBean callback(AuthCallbackRequest callback);

}
```

**（2）生产第三方登录策略工厂**

```java
@Slf4j
public class OauthStrategyFactory {
    public static IOauthService getOauthPolicy(Integer identifierType) {
        try {
            //基于identifierType通过枚举获取基于策略类的类型
            IdentifierType type = IdentifierType.subCodeOf(identifierType);
            if (Objects.isNull(type) || Objects.equals(IdentifierType.UNKNOWN, type)) {
                throw new UnsupportedOperationException(ResultCode.ERROR_OAUTH_POLICY.getCode());
            }
            String authServiceName = type.getAuthServiceName();
            //既然是Spring项目，那么就可以从IOC容器中根据策略类名获取对应bean
            return ApplicationContextBeanHelper.getBeanByClassName(authServiceName, IOauthService.class);
        } catch (Exception e) {
            log.error("Get oauth policy failed,error:{},cause:{}", e.getMessage(), e.getCause());
        }
        return null;
    }
}
```

**（3）顶层调用者通过工厂类和策略类交互**

```java
@Slf4j
@Service
public class OauthService {
    
    public AuthorizationBean authorize(OauthRequest request) {
        //通过策略工厂获取策略
        IOauthService oauthPolicy = OauthStrategyFactory.getOauthPolicy(request.getIdentifierType());
        if (Objects.isNull(oauthPolicy)) {
            throw new BusinessException(ResultCode.INTERNAL_ERROR);
        }
        //调用策略方法处理对应逻辑
        return oauthPolicy.authorize(request);
    }

    public AuthorizationBean callback(AuthCallbackRequest authCallback) {
         //通过策略工厂获取策略
        IOauthService oauthPolicy = OauthStrategyFactory.getOauthPolicy(authCallback.getPlatform());
        if (Objects.isNull(oauthPolicy)) {
            throw new BusinessException(ResultCode.INTERNAL_ERROR);
        }
        //调用策略方法处理对应逻辑
        return oauthPolicy.callback(authCallback);
    }
}
```

**（4）controller层基于OauthService对外交互**

```java
@RestController
@RequestMapping("/v1/open")
public class OauthController {

    @Autowired
    private OauthService oauthService;

    @ResponseWrapper
    @GetMapping(value = "/authorize")
    public AuthorizationBean authorize(@RequestParamAlias @Valid OauthRequest request) {
        return oauthService.authorize(request);
    }

    @ResponseWrapper
    @GetMapping(value = "/callback/{platform}")
    public AuthorizationBean callback(@PathVariable("platform") Integer platform, @RequestParamAlias @Valid AuthCallbackRequest callback) {
        if (IdentifierType.SYSTEM_IDENTITY_TYPE.contains(IdentifierType.codeOf(callback.getPlatform()))) {
            throw new BusinessException(ResultCode.INVALID_IDENTITY_TYPE);
        }
        callback.setPlatform(platform);
        return oauthService.callback(callback);
    }

}
```



#### 3、实现GitHub登录策略接口

上面主要实现了第三方刚认证登录的大致框架，实际干活的部分还未实现，这里借用网上的一幅时序图（如下图所示），大致向我们说明使用GitHub做第三方登录时需要App和GitHub做的交互操作，当然我们作为App\网站开发者肯定只用关注App部分的交互设计即可，主要来说需要做以下3件事情：

<img src="http://image.easyblog.top/15909305101017fb45bfc-cf28-4265-b86c-609a2d74ca98.png" style="width:70%;" />

**（1）在前端登录界面给用户一个可以点击并跳转到GitHub授权页面的链接**

随着接入的第三方越来越多，将这个链接直接维护在HTML页面不是一个很好的策略，我们需要在后端服务中统一维护这些三方信息，然后当用户跳转到我们系统的的登录界面时可以让前端动态请求系统后端服务生成链接：

```java
@Slf4j
@Service
public class GitHubAuthServiceImpl implements IAuthService<GitHubAuthBean>, IOauthService {

    @Autowired
    private GitHubAuthProperties gitHubAuthProperties;

    @Autowired
    private ILoginService loginService;


    @Autowired
    private GitHubOpenApiClient gitHubOpenApiClient;

    //获取GitHub登录授权链接
    @Override
    public AuthorizationBean authorize(OauthRequest request) {
        //client_id 是Github授权的app_id， redirect_uri是用户授权之后GitHub回调的地址，调试时可以写成本地地址
        String authorizationUr = String.format("%s?client_id=%s&state=STATE&redirect_uri=%s", gitHubAuthProperties.getAuthorizeUrl(),
                gitHubAuthProperties.getClientId(), gitHubAuthProperties.getRedirectUrl());
        return AuthorizationBean.builder().authorizationUrl(authorizationUr).build();
    }
}
```

这样做之后前端只需要在请求到链接之后直接将链接设置到登录按钮的href中等待用户点击接口。

**（2）获取AccessToken**

当用户点击授权之后，GitHub（或其他第三方平台）会回调我们在 redirect_uri 配置的回调地址，在回调方法中我们首先做的事情时带着gitHub 回调时的入参code再次请求github获取accessToken：

```java
@Slf4j
@Service
public class GitHubAuthServiceImpl implements IAuthService<GitHubAuthBean>, IOauthService {

    @Autowired
    private GitHubAuthProperties gitHubAuthProperties;

    @Autowired
    private ILoginService loginService;

    @Autowired
    private GitHubClient gitHubClient;

    @Autowired
    private GitHubOpenApiClient gitHubOpenApiClient;

    @Override
    public String getAccessToken(String code) {
        //这里使用Feign将GitHub请求的accessToken的地址包装了一下，也可以直接使用HttpClient请求
        MultiValueMap<String, String> accessToken = gitHubClient.getAccessToken(QueryGitHubAuthTokenRequest.builder()
                .clientId(gitHubAuthProperties.getClientId())
                .clientSecret(gitHubAuthProperties.getClientSecret())
                .code(code)
                .grantType(LoginConstants.GITHUB_GRANT_TYPE)
                .build());
        if (Objects.isNull(accessToken)) {
            throw new BusinessException(ResultCode.REQUEST_GITHUB_ACCESS_TOKEN_FAILED);
        }
        return Iterables.getFirst(accessToken.get("accessToken"), null);
    }
}
```



**（3）带着accessToken再次请求GitHub获取用户信息**

```java
@Slf4j
@Service
public class GitHubAuthServiceImpl implements IAuthService<GitHubAuthBean>, IOauthService {

    @Autowired
    private GitHubAuthProperties gitHubAuthProperties;

    @Autowired
    private ILoginService loginService;

    @Autowired
    private GitHubClient gitHubClient;

    @Autowired
    private GitHubOpenApiClient gitHubOpenApiClient;

    @Override
    public GitHubAuthBean getUserInfo(String accessToken) {
        GitHubAuthBean gitHubAuthBean = gitHubOpenApiClient.getGithubUserInfo(String.format("token %s", accessToken));
        if (Objects.isNull(gitHubAuthBean)) {
            throw new BusinessException(ResultCode.REQUEST_GITHUB_USER_INFO_FAILED);
        }
        log.info("Get github user information: {}", JsonUtils.toJSONString(gitHubAuthBean));
        return gitHubAuthBean;
    }
}
```

**（4）用户获取成功，将用户保存到系统数据库并让用户登录成功**

```java
@Slf4j
@Service
public class GitHubAuthServiceImpl implements IAuthService<GitHubAuthBean>, IOauthService {

    @Autowired
    private GitHubAuthProperties gitHubAuthProperties;

    @Autowired
    private ILoginService loginService;

    @Autowired
    private GitHubClient gitHubClient;

    @Autowired
    private GitHubOpenApiClient gitHubOpenApiClient;

    @Override
    public String getAccessToken(String code) {
        MultiValueMap<String, String> accessToken = gitHubClient.getAccessToken(QueryGitHubAuthTokenRequest.builder()
                .clientId(gitHubAuthProperties.getClientId())
                .clientSecret(gitHubAuthProperties.getClientSecret())
                .code(code)
                .grantType(LoginConstants.GITHUB_GRANT_TYPE)
                .build());
        if (Objects.isNull(accessToken)) {
            throw new BusinessException(ResultCode.REQUEST_GITHUB_ACCESS_TOKEN_FAILED);
        }
        return Iterables.getFirst(accessToken.get("accessToken"), null);
    }

    @Override
    public GitHubAuthBean getUserInfo(String accessToken) {
        GitHubAuthBean gitHubAuthBean = gitHubOpenApiClient.getGithubUserInfo(String.format("token %s", accessToken));
        if (Objects.isNull(gitHubAuthBean)) {
            throw new BusinessException(ResultCode.REQUEST_GITHUB_USER_INFO_FAILED);
        }
        log.info("Get github user information: {}", JsonUtils.toJSONString(gitHubAuthBean));
        return gitHubAuthBean;
    }


    @Override
    public AuthorizationBean authorize(OauthRequest request) {
        String authorizationUr = String.format("%s?client_id=%s&state=STATE&redirect_uri=%s", gitHubAuthProperties.getAuthorizeUrl(),
                gitHubAuthProperties.getClientId(), gitHubAuthProperties.getRedirectUrl());
        return AuthorizationBean.builder().authorizationUrl(authorizationUr).build();
    }

    //回调函数
    @Override
    public AuthorizationBean callback(AuthCallbackRequest callback) {
        //1.获取accessToken
        String accessToken = getAccessToken(callback.getCode());
        return Optional.ofNullable(accessToken).map(token -> {
            //2.获取用户信息
            GitHubAuthBean userInfo = getUserInfo(token);
            //3.保存三方用户信息，这里也需要用到策略+工厂+枚举类的方式来处理
            AuthenticationDetailsBean authenticationDetailsBean = loginService.register(RegisterUserRequest.builder()
                    .identifierType(IdentifierType.GITHUB.getSubCode())
                    .identifier(userInfo.getId())
                    .build());
            return AuthorizationBean.builder().user(authenticationDetailsBean.getUser()).build();
        }).orElseThrow(() -> new BusinessException(ResultCode.REQUIRED_REQUEST_PARAM_NOT_EXISTS));
    }
}
```



本文所有代码可以在[GitHub仓库](https://github.com/easyblog-org/User)获得，感兴趣的小伙伴可以pull下来研究研究~~

### 参考

* [第三方登录之Github登录篇](https://blog.csdn.net/weixin_44015043/article/details/106059924)
* [谷粒商城OAuth2.0接入Gitee登录](https://blog.csdn.net/Alvin199765/article/details/118604099?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-3-118604099.pc_agg_new_rank&utm_term=gitee+%E6%80%8E%E4%B9%88%E7%99%BB%E5%BD%95&spm=1000.2123.3001.4430)
* [SpringBoot网站基于OAuth2添加第三方登录之GitHub登录](https://easyblog.top/article/details/236)

