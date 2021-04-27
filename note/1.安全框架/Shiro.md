# Shiro

#安全框架/Shiro#

## 权限

权限管理在系统中一般分为：

* 访问权限

``` properties
一般表示你能做什么样的操作，或者能够访问那些资源。例如：给张三赋予“店铺主管”角色，“店铺主管”具有“查询员工”、“添加员工”、“修改员工”和“删除员工”权限。此时张三能够进入系统，则可以进行这些操作
```

* 数据权限

``` properties
一般表示某些数据你是否属于你，或者属于你可以操作范围。例如：张三是"店铺主管"角色，他可以看他手下客服人员所有的服务的买家订单信息，他的手下只能看自己负责的订单信息
```



## 认证

* 认证流程

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210417203411.png" alt="image-20210417203411294" style="zoom:50%;" />

### 认证的主要对象

* **Subject** : 主体 :  主体可以是访问的用户,或者是一个应用系统
* **Principal** : 身份 : 是主体Subject进行认证的标识,标识必须具有唯一性. 如用户名,手机号,邮箱等
  * 一个主体可以有多个身份,但是必须有一个主身份
* **credential** : 凭证 : 只有主体自己知道的信息,如密码,证书.

## 授权

​	即访问控制，控制谁能访问哪些资源。主体进行身份认证后，系统会为其分配对应的权限，当访问资

源时，会校验其是否有访问此资源的权限。

* 授权流程

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210417204515.png" alt="image-20210417204515762" style="zoom:50%;" />

​		用户对象user：当前操作的用户、程序。

​		资源对象resource：当前被访问的对象

​		角色对象role ：一组 "权限操作许可权" 的集合。

​		权限对象permission：权限操作许可权

### 授权的主要对象

**授权可简单理解为who对what进行How操作**

* **Who：**主体（Subject），可以是一个用户、也可以是一个程序
* **What：**资源（Resource），如系统菜单、页面、按钮、方法、系统商品信息等。
  * 访问类型：商品菜单，订单菜单、分销商菜单
  * 数据类型：我的商品，我的订单，我的评价
* **How：**权限/许可（Permission）
  * 我的商品（资源）访问我的商品(权限许可)
  * 分销商菜单（资源）访问分销商列表（权限许可）

## 核心组件

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210417204800.png" alt="image-20210417204759946" style="zoom: 67%;" />

- Subject

```properties
Subject主体，外部应用与subject进行交互，subject将用户作为当前操作的主体，这个主体：可以是一个通过浏览器请求的用户，也可能是一个运行的程序。Subject在shiro中是一个接口，接口中定义了很多认证授相关的方法，外部程序通过subject进行认证授，而subject是通过SecurityManager安全管理器进行认证授权
```

- SecurityManager

```properties
SecurityManager权限管理器，它是shiro的核心，负责对所有的subject进行安全管理。通过SecurityManager可以完成subject的认证、授权等，SecurityManager是通过Authenticator进行认证，通过Authorizer进行授权，通过SessionManager进行会话管理等。SecurityManager是一个接口，继承了Authenticator, Authorizer, SessionManager这三个接口
```

- Authenticator

```properties
Authenticator即认证器，对用户登录时进行身份认证
```

- Authorizer

```properties
Authorizer授权器，用户通过认证器认证通过，在访问功能时需要通过授权器判断用户是否有此功能的操作权限。
```

- Realm（数据库读取+认证功能+授权功能实现）

```properties
Realm领域，相当于datasource数据源，securityManager进行安全认证需要通过Realm获取用户权限数据
比如：
	如果用户身份数据在数据库那么realm就需要从数据库获取用户身份信息。
注意：
	不要把realm理解成只是从数据源取数据，在realm中还有认证授权校验的相关的代码。　
```

- SessionManager

```properties
SessionManager会话管理，shiro框架定义了一套会话管理，它不依赖web容器的session，所以shiro可以使用在非web应用上，也可以将分布式应用的会话集中在一点管理，此特性可使它实现单点登录。
```

- SessionDAO

```properties
SessionDAO即会话dao，是对session会话操作的一套接口
比如:
	可以通过jdbc将会话存储到数据库
	也可以把session存储到缓存服务器
```

- CacheManager 

```properties
CacheManager缓存管理，将用户权限数据存储在缓存，这样可以提高性能
```

- Cryptography

```
Cryptography密码管理，shiro提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能
```

## 概念

### 认证流程

1. Shiro把用户的数据封装成标识token，token一般封装着用户名，密码等信息
2. 使用Subject门面获取到封装着用户的数据的标识token
3. Subject把标识token交给SecurityManager，在SecurityManager安全中心中，SecurityManager把标识token委托给认证器Authenticator进行身份验证。认证器的作用一般是用来指定如何验证，它规定本次认证用到哪些Realm
4. 认证器Authenticator将传入的标识token，与数据源Realm对比，验证token是否合法

### 案例

```ini
#声明用户账号
[users]
jay=123
```

```java

/**
 * @Description：shiro的第一个例子
 */
public class HelloShiro {

    @Test
    public void shiroLogin() {
        //导入权限ini文件构建权限工厂
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        //工厂构建安全管理器
        SecurityManager securityManager = factory.getInstance();
        //使用SecurityUtils工具生效安全管理器
        SecurityUtils.setSecurityManager(securityManager);
        //使用SecurityUtils工具获得主体
        Subject subject = SecurityUtils.getSubject();
        //构建账号token
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("jay", "123");
        //登录操作
        subject.login(usernamePasswordToken);
        System.out.println("是否登录成功：" + subject.isAuthenticated());
    }
}
```

### AuthenticationToken接口

当Subject 调用login()方法时,会传递实现的Token,最终传递到doGetAuthenticationInfo(AuthenticationToken auth) 

该接口是Shiro提供的Token接口,默认有几个实现,也可以自己继承接口实现Token逻辑.

### Realm接口

* Shiro默认提供三个Realm接口
  * **CachingRealm** : 带缓存的Realm
  * **AuthenticatingRealm** : 带认证的Realm
  * **AuthorizingRealm** : 带授权的Realm
* 默认继承**AuthorizingRealm** 即可,该接口继承以上两个接口.

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210417211804.png" alt="image-20210417211804268" style="zoom:67%;" />

**直接继承AuthorizingRealm，能够继承到认证与授权功能。它需要强制重写两个方法**

```java
public class DefinitionRealm extends AuthorizingRealm {
 
    /**
	 * @Description 认证
	 * @param authcToken token对象
	 * @return 
	 */
	public abstract AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) {
        return null;
    }

	/**
	 * @Description 鉴权
	 * @param principals 令牌,身份
	 * @return
	 */
	public abstract AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals){
        return null;
    }
}
```

### 自定义Realm

Realm是Shiro的与我们数据交互的逻辑,我们实现可以进行相关逻辑的编写

* 新建一个Service 用于与数据库获取数据
* 新建一个Realm继承AuthorizingRealm接口,实现相关方法
  * doGetAuthenticationInfo(AuthenticationToken token) : 认证方法
  * doGetAuthorizationInfo(PrincipalCollection principals) : 授权方法

### 编码/散列算法

提供了Basse64的编码与解码

Shiro提供了许多算法,包括,MD5,sha等

### Realm使用散列算法

在做认证的时候,使用散列算法将铭文密码加密为密文密码,然后和数据库中的密文密码对比.

当使用了散列算法加密密码后,需要修改SecurityManage默认的密码匹配方式

* 在实现的Realm中可以通过空参构造方法来进行初始化
* 创建一个密码比较器对象,使用了那种密码比较器则new 对应的密码比较器对象,在比较器构造方法中指定所使用的算法,通过DigestsUtil.SHA1 ...等其他算法, 的方式指定在密码比较器的构造方法中
  * CredentialsMatcher接口
    * HashedCredentialsMatcher 哈希密码比较器
    * PasswordMaatcher 
    * SimpledCredentialsMatcher
    * AllowAlldCredentialsMatcher
* 密码比较器对象调用setHashInterations(DigestsUtil.ITERTIONS);的方式指定加密的次数
* 调用父类方法setCredentialsMatcher(密码比较器对象);使密码比较器生效;

```java
public class MyRealm{
    public MyRealm(){
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher(DegistsUtil.SHA1);
        matcher.setHashInterations(DegestsUtil.ITERTIONS);
        setCredentialsMatcher(matcher);
    }
    
    doAuthentication()....
    doAuthorizing()...
}
```

### 设置散列算法源码

1. 通过 setCredentialsMatcher(matcher);方法
2. 方法将我们new的密码匹配器对象设置给成员变量CredentialsMatcher,然后在认证时就会调用该实现类的方法进行密码匹配
3. 底层方法调用到AuthenticatingRealm类方法getAuthenticationInfo(token)
   1. 此方法核心逻辑一 getCachedAuthenticationInfo(token);获取用户info
   2. 核心逻辑二 cacheAuthenticationInfoIfPossible(token, info);缓存操作
   3. 核心逻辑三 assertCredentialsMatch(token, info);通过指定密码匹配器进行匹配
4. 在核心逻辑三中 assertCredentialsMatch(token, info);进行了密码匹配
   1. 方法内通过getCredentialsMatcher();获取密码匹配器对象
   2. 通过对应密码匹配器调用方法.doCredentialsMatch(token, info)进行匹配
   3. 匹配失败则会抛出new AuthenticationException异常
   4. 匹配成功抛出new IncorrectCredentialsException(msg);

## 登录流程

1. 当调用Sbuject.login(token)后,会交给SecurityManage进行登录
   1. 会找到DefaultSecurityManager的login调用父类authenticate(token)方法
   2. authenticate(token)方法会找到Authenticator的authenticate(token)方法
   3. authenticate是抽象方法,找到实现类AbstractAuthenticator#ModularRealmAuthenticator
   4. AbstractAuthenticator调用doAuthenticate(token);此方法有子类ModularRealmAuthenticator实现
   5. 子类方法判断当前Realm数量
      1. 如果只有一个,则会调用该doSingleRealmAuthentication方法调用Realm的getAuthenticationInfo实现方法获取info返回
      2. 如过有多个Realm,则会执行doMultiRealmAuthentication方法,从多个Realm中查询info信息
         1. 该返回的info是聚合info,包含从多个Realm中查询到的所有info信息
   6. 如果没有查询到info信息则会在authenticate方法中抛出AuthenticationException异常并被自身捕获
      1. 捕获后会对异常进行包装再次抛出,并且会调用notifyFailure方法
   7. 查询到info信息,则会调用notifySuccess方法,并返回info



## 身份授权流程

1. 当Subject调用 后交给SercurityManage

   * hasRole("角色代码")  返回值boolean

   * checkRole("角色代码")   不存在则抛出异常
   * isPermitted("权限代码") 返回值boolean
   * checkPermitted("权限代码") 不存在则抛出异常

2. SercurityManage调用Authorizer的hasRole(subjectPrincipal,roleIdentifier)

3. 最终会通过缓存获取,如果缓存获取不到,则会通过我们实现类的doGetAuthorizationInfo()获取;



# SpringBoot 集成Shiro

## Realm

![image-20210418215616874](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210418215624.png)

#### 原理分析

- ShiroDbRealmImpl继承ShiroDbRealm向上继承AuthorizingRealm，ShiroDbRealmImpl实例化时会创建密码匹配器HashedCredentialsMatcher实例，HashedCredentialsMatcher指定hash次数与方式，交于AuthenticatingRealm

- 调用login方法后，最终调用doGetAuthenticationInfo(AuthenticationToken authcToken)方法，拿到自定义Token的对象，调用自定义Service的查找用户方法，把ShiroUser对象、密码和salt交于SimpleAuthenticationInfo去认证

- 访问需要鉴权时，调用doGetAuthorizationInfo(PrincipalCollection principals)方法，然后调用自定义Service的授权验证 

## ShiroConfig

![image-20210418215929789](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210418215929.png)

* #### 原理分析

  - 创建SimpleCookie对象，访问项目时，会在客户端中cookie中存放ShiroSession的对象

  * 创建DefaultWebSessionManager会话管理器定义cookie机制、定时刷新、全局会话超时时间然后交于DefaultWebSecurityManager权限管理器管理
  * 创建自定义ShiroDbRealm实现，用于权限认证、授权、加密方式的管理，同时从数据库中取得相关的角色、资源、用户的信息，然后交于DefaultWebSecurityManager权限管理器管理
  * 创建DefaultWebSecurityManager权限管理器用于管理DefaultWebSessionManager会话管理器、ShiroDbRealm
  * 创建lifecycleBeanPostProcessor和DefaultAdvisorAutoProxyCreator相互配合事项注解的权限鉴权
  * 创建ShiroFilterFactoryBean的shiro过滤器指定权限管理器、同时启动连接链及登录URL、未登录的URL

### DefaultSecurityMannage

继承关系,集成了缓存,认证,授权等SecurityMannage,默认实现DefaultSecurityMannage即可

![image-20210418221020656](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210418221020.png)

```java
/**
 * @Description：权限配置类
 */
@Configuration
@ComponentScan(basePackages = "com.jianjian.shiro")
@Log4j2
public class ShiroConfig {

    /**
     * @Description 创建cookie对象
     */
    @Bean(name="sessionIdCookie")
    public SimpleCookie simpleCookie(){
        //生产cookie的对象,shiro提供
        SimpleCookie simpleCookie = new SimpleCookie();
        simpleCookie.setName("ShiroSession");
        return simpleCookie;
    }

    /**
     * @Description 权限管理器
     */
    @Bean(name="securityManager")
    public DefaultWebSecurityManager defaultWebSecurityManager(){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //设置领域
        securityManager.setRealm(shiroDbRealm());
        //设置session管理器
        securityManager.setSessionManager(shiroSessionManager());
        return securityManager;
    }

    /**
     * @Description 自定义RealmImpl,并交给IOC管理
     */
    @Bean(name="shiroDbRealm")
    public ShiroDbRealm shiroDbRealm(){
        //Realm可以指定缓存管理器
        //shiroDbRealm realm = new shiroDbRealm();
        //只有本地缓存EhCache才需要开启,整合Redisson不需要这样
        //Cache实现类方法,是在代码中已经存在的过程,所以不需要开启
        return new ShiroDbRealmImpl();
    }


    /**
     * @Description 会话管理器
     */
    @Bean(name="sessionManager")
    public DefaultWebSessionManager shiroSessionManager(){
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        //关闭会话更新
        sessionManager.setSessionValidationSchedulerEnabled(false);
        //开启cookie
        sessionManager.setSessionIdCookieEnabled(true);
        //指定生产cookie的策略
        sessionManager.setSessionIdCookie(simpleCookie());
        //会话超时时间
        sessionManager.setGlobalSessionTimeout(3600000);
        return sessionManager;
    }

    /**
     * @Description 保证实现了Shiro内部lifecycle函数的bean执行
     * static 保证LifecycleBeanPostProcessor优先被实例化能够读取配置文件中的配置
     */
    @Bean(name = "lifecycleBeanPostProcessor")
    public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    /**
     * @Description AOP式方法级权限检查
     *  依赖于LifecycleBeanPostProcessor生命周期对象;
     */
    @Bean
    @DependsOn("defaultAdvisorAutoProxyCreator")
    public DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

    /**
     * @Description 配合DefaultAdvisorAutoProxyCreator开启事项注解权限校验
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor() {
        AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
        aasa.setSecurityManager(defaultWebSecurityManager());
        return new AuthorizationAttributeSourceAdvisor();
    }


    /**
     * @Description Shiro过滤器
     */
    @Bean("shiroFilter")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(){
        //过滤器
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(defaultWebSecurityManager());
        //设置配置文件中的过滤器链  此方法的接收 k,v 都是String类型的过滤器链
        shiroFilter.setFilterChainDefinitionMap(filterChainDefinition());
        
        /*自定义过滤器
         *实现AuthrizatoionFilter接口,重写isAccsessAllowed()方法
         *定义map:
         *key - String 为过滤器的名称
         *value - Filter: 传入自定义Filter对象
         *ShiroFilterFactoryBean类方法setFilters(定义的map); 即可
         */
        //登录路径
        shiroFilter.setLoginUrl("/login");
        //未授权时跳转的路径
        shiroFilter.setUnauthorizedUrl("/login");
        return shiroFilter;
    }
    
    /**
     * @Description 过滤器链  (此方法为工具方法)
     */
    private Map<String, String> filterChainDefinition(){
        List<Object> list  = PropertiesUtil.propertiesShiro.getKeyList();
        Map<String, String> map = new LinkedHashMap<>();
        for (Object object : list) {
            String key = object.toString();
            String value = PropertiesUtil.getShiroValue(key);
            log.info("读取防止盗链控制：---key{},---value:{}",key,value);
            map.put(key, value);
        }
        return map;
    }
}

```



## 过滤器



* Shiro在枚举中提供了很多的默认过滤器

```java
public enum DefaultFilter {

    anon(AnonymousFilter.class),
    authc(FormAuthenticationFilter.class),
    authcBasic(BasicHttpAuthenticationFilter.class),
    logout(LogoutFilter.class),
    noSessionCreation(NoSessionCreationFilter.class),
    perms(PermissionsAuthorizationFilter.class),
    port(PortFilter.class),
    rest(HttpMethodPermissionFilter.class),
    roles(RolesAuthorizationFilter.class),
    ssl(SslFilter.class),
    user(UserFilter.class);
}
```

#### 【1】认证相关

| 过滤器 | 过滤器类                 | 说明                                                         | 默认 |
| ------ | ------------------------ | ------------------------------------------------------------ | ---- |
| authc  | FormAuthenticationFilter | 基于表单的过滤器；如“/**=authc”，如果没有登录会跳到相应的登录页面登录 | 无   |
| logout | LogoutFilter             | 退出过滤器，主要属性：redirectUrl：退出成功后重定向的地址，如“/logout=logout” | /    |
| anon   | AnonymousFilter          | 匿名过滤器，即不需要登录即可访问；一般用于静态资源过滤；示例“/static/**=anon” | 无   |

#### 【2】授权相关

| 过滤器 | 过滤器类                       | 说明                                                         | 默认 |
| ------ | ------------------------------ | ------------------------------------------------------------ | ---- |
| roles  | RolesAuthorizationFilter       | 角色授权拦截器，验证用户是否拥有所有角色；主要属性： loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例“/admin/**=roles[admin]” | 无   |
| perms  | PermissionsAuthorizationFilter | 权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例“/user/**=perms["user:create"]” | 无   |
| port   | PortFilter                     | 端口拦截器，主要属性：port（80）：可以通过的端口；示例“/test= port[80]”，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样 | 无   |
| rest   | HttpMethodPermissionFilter     | rest风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例“/users=rest[user]”，会自动拼出“user:read,user:create,user:update,user:delete”权限字符串进行权限匹配（所有都得匹配，isPermittedAll） | 无   |
| ssl    | SslFilter                      | SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口（443）；其他和port拦截器一样； |      |

* Shiro的过滤器配置是自上而下顺序的,如果匹配了第一个过滤器则不会再向下匹配

<img src="https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210419224439.png" alt="image-20210419224439634" style="zoom: 67%;" />

* 在创建ShiroFilterFactoryBean时,构造方法初始化了一个过滤器,和过滤器链 的 LinkedHashMap集合
* ShiroFilterFactoryBean类createFilterChainManager方法其内部方法自动加载默认的过滤器
  * 加载完成默认过滤器后,会加载一个全局配置
  * 加载完全局配置后,会加载过滤器链

### 自定义过滤器

自定义过滤器可以继承AccessControlFilter,实现即可,此过滤器下有很多实现类,也可以继承实现类

```java
//过滤器方法分析
public abstract class AccessControlFilter extends PathMatchingFilter {

    /**
     */
    public static final String DEFAULT_LOGIN_URL = "/login.jsp";

    /**
     */
    public static final String GET_METHOD = "GET";

    /**
     */
    public static final String POST_METHOD = "POST";

    /**
     */
    private String loginUrl = DEFAULT_LOGIN_URL;

    /**
     */
    public String getLoginUrl() {
        return loginUrl;
    }

    /**
     */
    public void setLoginUrl(String loginUrl) {
        this.loginUrl = loginUrl;
    }

    /**
	 * 获取Subject对象
     */
    protected Subject getSubject(ServletRequest request, ServletResponse response) {
        return SecurityUtils.getSubject();
    }

    /**
     * 是否拒绝访问
     */
    protected abstract boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception;

    /**
     * 拒绝后处理的业务逻辑
     */
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return onAccessDenied(request, response);
    }

    /**
     * 拒绝后处理的业务逻辑
     */
    protected abstract boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception;

    /**
     * 最初调用的方法,过滤器流程开始的方法
     * 只要两个方法有一个返回true,则会继续执行后续的逻辑
     */
    public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);
    }

    /**
     *
     */
    protected boolean isLoginRequest(ServletRequest request, ServletResponse response) {
        return pathsMatch(getLoginUrl(), request);
    }

    /**
     * 宝UN请
     */
    protected void saveRequestAndRedirectToLogin(ServletRequest request, ServletResponse response) throws IOException {
        saveRequest(request);
        redirectToLogin(request, response);
    }

    /**
     * 保存请求
     */
    protected void saveRequest(ServletRequest request) {
        WebUtils.saveRequest(request);
    }

    /**
     * 跳转登录的路径
     */
    protected void redirectToLogin(ServletRequest request, ServletResponse response) throws IOException {
        String loginUrl = getLoginUrl();
        WebUtils.issueRedirect(request, response, loginUrl);
    }

}
```



#### 定义

* 实现AuthrizatoionFilter接口,重写isAccsessAllowed()方法
* 定义map:
  * key - String 为过滤器的名称
  * value - Filter: 传入自定义Filter对象
* ShiroFilterFactoryBean类方法setFilters(定义的map); 
* 返回ShiroFilterFactoryBean对象,交给IOC管理;即可

```java
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        try {
            if (this.isWhiteList(request)) {
                return true;
            } else if (this.isLoginAttempt(request, response)) {
                try {
                    this.executeLogin(request, response);
                    return true;
                } catch (Exception var7) {
                    log.error("execute login error.", var7);
                    String msg = var7.getMessage();
                    Throwable throwable = var7.getCause();
                    if (throwable instanceof SignatureVerificationException) {
                        msg = String.format("Token或者密钥不正确(%s)", throwable.getMessage());
                    } else if (throwable instanceof TokenExpiredException) {
                        msg = String.format("Token已过期(%s)", throwable.getMessage());
                    } else if (throwable != null) {
                        msg = "un know error";
                    }

                    throw new ApplicationException(HttpStatus.UNAUTHORIZED.value(), msg);
                }
            } else {
                return false;
            }
        } catch (Throwable var8) {
            throw var8;
        }
    }

```

#### 使用

```java
/**
 * @Description Shiro过滤器
 */
@Bean("shiroFilter")
public ShiroFilterFactoryBean shiroFilterFactoryBean(){
    //过滤器
    ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
    shiroFilter.setSecurityManager(defaultWebSecurityManager());
    //设置配置文件中的过滤器链  此方法的接收 k,v 都是String类型的过滤器链
    shiroFilter.setFilterChainDefinitionMap(filterChainDefinition());
    
    /*自定义过滤器
     *实现AuthrizatoionFilter接口,重写isAccsessAllowed()方法
     *定义map:
     *key - String 为过滤器的名称
     *value - Filter: 传入自定义Filter对象
     *ShiroFilterFactoryBean类方法setFilters(定义的map); 即可
     */
    
    //登录路径
    shiroFilter.setLoginUrl("/login");
    //未授权时跳转的路径
    shiroFilter.setUnauthorizedUrl("/login");
    return shiroFilter;
}
```



## 注解方式鉴权

### 注解

**类型:**

* **@RequiresAuthentication** : 当前方法是已经认证过的用户才可以访问的
* **@RequiresGuest** : 表明当前需要时Guest(匿名)用户
* **@RequiresPermission** : 表名需要特定权限才可以访问
* **@RequiresRoles** : 用户需要拥有指定角色才可以访问
* **@RequiresUser** : 当前已经认证或者记住的角色才可以访问

**作用位置:**

* 在Dao,Service,Controller层都是可以加的,如果需要很高的细粒度可以加在Dao,Service
* 不需要很高的细粒度则加在Controller层即可

### 源码原理

基于Spring AOP的思想实现的,但是并非使用@Aspect注解

```java
入口: 在Config配置中
1. DefaultAdvisorAutoProxyCreator(AOP方法级别权限检查)
2. AuthorizationAttributeSourceAdvisor(配合DefaultAdvisorAutoProxyCreator实现注解权限校验)
依靠AuthorizationAttributeSourceAdvisor类配置切面和切入点来实现
3. AuthorizationAttributeSourceAdvisor继承StaticMethodMatcherPointcutAdvisor类,此类是Spring提供

```

* `DefaultAdvisorAutoProxyCreator`这个类实现了`BeanProcessor`接口,当`ApplicationContext`读取所有的Bean配置信息后，这个类将扫描上下文，寻找所有的`Advistor`(一个`Advisor`是一个切入点和一个通知的组成)，将这些`Advisor`应用到所有符合切入点的Bean中。

```java
@Configuration
public class ShiroAnnotationProcessorConfiguration extends AbstractShiroAnnotationProcessorConfiguration{
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    protected DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        return super.defaultAdvisorAutoProxyCreator();
    }

    @Bean
    protected AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        return super.authorizationAttributeSourceAdvisor(securityManager);
    }

}

```

* `AuthorizationAttributeSourceAdvisor`继承了`StaticMethodMatcherPointcutAdvisor`，如下代码所示，只匹配五个注解，也就是说只对这五个注解标注的类或者方法增强。`StaticMethodMatcherPointcutAdvisor`是静态方法切点的抽象基类，默认情况下它匹配所有的类。`StaticMethodMatcherPointcut`包括两个主要的子类分别是`NameMatchMethodPointcut`和`AbstractRegexpMethodPointcut`，前者提供简单字符串匹配方法前面，而后者使用正则表达式匹配方法前面。动态方法切点：`DynamicMethodMatcerPointcut`是动态方法切点的抽象基类，默认情况下它匹配所有的类，而且也已经过时，建议使用`DefaultPointcutAdvisor`和`DynamicMethodMatcherPointcut`动态方法代替。另外还需关注构造器中的传入的`AopAllianceAnnotationsAuthorizingMethodInterceptor`。

```java
public class AuthorizationAttributeSourceAdvisor extends StaticMethodMatcherPointcutAdvisor {

    private static final Logger log = LoggerFactory.getLogger(AuthorizationAttributeSourceAdvisor.class);

    //初始化需要增强的类型
    private static final Class<? extends Annotation>[] AUTHZ_ANNOTATION_CLASSES =
            new Class[] {
                    RequiresPermissions.class, RequiresRoles.class,
                    RequiresUser.class, RequiresGuest.class, RequiresAuthentication.class
            };

    protected SecurityManager securityManager = null;

    public AuthorizationAttributeSourceAdvisor() {
        setAdvice(new AopAllianceAnnotationsAuthorizingMethodInterceptor());
    }

    public SecurityManager getSecurityManager() {
        return securityManager;
    }

    public void setSecurityManager(org.apache.shiro.mgt.SecurityManager securityManager) {
        this.securityManager = securityManager;
    }
	//匹配规则,true则增强
    public boolean matches(Method method, Class targetClass) {
        Method m = method;

        if ( isAuthzAnnotationPresent(m) ) {
            return true;
        }
        
        if ( targetClass != null) {
            try {
                m = targetClass.getMethod(m.getName(), m.getParameterTypes());
                if ( isAuthzAnnotationPresent(m) ) {
                    return true;
                }
            } catch (NoSuchMethodException ignored) {
                
            }
        }

        return false;
    }

    private boolean isAuthzAnnotationPresent(Method method) {
        for( Class<? extends Annotation> annClass : AUTHZ_ANNOTATION_CLASSES ) {
            Annotation a = AnnotationUtils.findAnnotation(method, annClass);
            if ( a != null ) {
                return true;
            }
        }
        return false;
    }

}
```

* `AopAllianceAnnotationsAuthorizingMethodInterceptor`在初始化时，`interceptors`添加了5个方法拦截器(都继承自`AuthorizingAnnotationMethodInterceptor`)，这5个拦截器分别对5种权限验证的方法进行拦截，执行invoke方法。

```java
public class AopAllianceAnnotationsAuthorizingMethodInterceptor
        extends AnnotationsAuthorizingMethodInterceptor implements MethodInterceptor {

    public AopAllianceAnnotationsAuthorizingMethodInterceptor() {
        List<AuthorizingAnnotationMethodInterceptor> interceptors =
                new ArrayList<AuthorizingAnnotationMethodInterceptor>(5);
        AnnotationResolver resolver = new SpringAnnotationResolver();
        
        interceptors.add(new RoleAnnotationMethodInterceptor(resolver));
        interceptors.add(new PermissionAnnotationMethodInterceptor(resolver));
        interceptors.add(new AuthenticatedAnnotationMethodInterceptor(resolver));
        interceptors.add(new UserAnnotationMethodInterceptor(resolver));
        interceptors.add(new GuestAnnotationMethodInterceptor(resolver));
        setMethodInterceptors(interceptors);
    }
    //invoke方法核心方法
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        org.apache.shiro.aop.MethodInvocation mi = createMethodInvocation(methodInvocation);
        return super.invoke(mi);
    }
    ...
}

```

* `AopAllianceAnnotationsAuthorizingMethodInterceptor`的invoke方法，又会调用超类`AuthorizingMethodInterceptor`的invoke方法，在该方法中先执行assertAuthorized方法，进行权限校验，校验不通过，抛出`AuthorizationException`异常，中断方法；校验通过，则执行`methodInvocation.proceed()`，该方法也就是被拦截并且需要权限校验的方法。

```java
public abstract class AuthorizingMethodInterceptor extends MethodInterceptorSupport {

    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        assertAuthorized(methodInvocation);
        return methodInvocation.proceed();
    }
	//断言方法
    protected abstract void assertAuthorized(MethodInvocation methodInvocation) throws AuthorizationException;
}

```

* assertAuthorized方法最终执行的还是`AuthorizingAnnotationMethodInterceptor.assertAuthorized`，而`AuthorizingAnnotationMethodInterceptor`有5个的具体的实现类(`RoleAnnotationMethodInterceptor`, `PermissionAnnotationMethodInterceptor`, `AuthenticatedAnnotationMethodInterceptor`, `UserAnnotationMethodInterceptor`, `GuestAnnotationMethodInterceptor`)。

```java
public abstract class AnnotationsAuthorizingMethodInterceptor extends 	AuthorizingMethodInterceptor {
  
    protected void assertAuthorized(MethodInvocation methodInvocation) throws AuthorizationException {
        //default implementation just ensures no deny votes are cast:
        Collection<AuthorizingAnnotationMethodInterceptor> aamis = getMethodInterceptors();
        if (aamis != null && !aamis.isEmpty()) {
            for (AuthorizingAnnotationMethodInterceptor aami : aamis) {
                if (aami.supports(methodInvocation)) {
                    aami.assertAuthorized(methodInvocation);
                }
            }
        }
    }
    ...
}

```

* 5个的具体的实现类都有对应的Handler处理方法
* `AuthorizingAnnotationMethodInterceptor`的assertAuthorized，首先从子类获取`AuthorizingAnnotationHandler`，再调用该实现类的`assertAuthorized`方法。

```java
public abstract class AuthorizingAnnotationMethodInterceptor extends AnnotationMethodInterceptor
{

    public AuthorizingAnnotationMethodInterceptor( AuthorizingAnnotationHandler handler ) {
        super(handler);
    }

    public AuthorizingAnnotationMethodInterceptor( AuthorizingAnnotationHandler handler,
                                                   AnnotationResolver resolver) {
        super(handler, resolver);
    }

    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        assertAuthorized(methodInvocation);
        return methodInvocation.proceed();
    }

    public void assertAuthorized(MethodInvocation mi) throws AuthorizationException {
        try {
            ((AuthorizingAnnotationHandler)getHandler()).assertAuthorized(getAnnotation(mi));
        }
        catch(AuthorizationException ae) {
            if (ae.getCause() == null) ae.initCause(new AuthorizationException("Not authorized to invoke method: " + mi.getMethod()));
            throw ae;
        }         
    }
}


```

* 现在分析其中一种实现类`PermissionAnnotationMethodInterceptor`，也是用的最多的，但是这个类的实际代码很少，很明显上述分析的getHandler在`PermissionAnnotationMethodInterceptor`中返回值为`PermissionAnnotationHandler`。

```java
public class PermissionAnnotationMethodInterceptor extends AuthorizingAnnotationMethodInterceptor {

    public PermissionAnnotationMethodInterceptor() {
        super( new PermissionAnnotationHandler() );
    }

 	//调用对应Handler处理方法
    public PermissionAnnotationMethodInterceptor(AnnotationResolver resolver) {
        super( new PermissionAnnotationHandler(), resolver);
    }
}

```

* 在`PermissionAnnotationHandler`类中，终于发现实际的检验逻辑，还是调用的`Subject.checkPermission()`进行校验。

```java
public class PermissionAnnotationHandler extends AuthorizingAnnotationHandler {

    public PermissionAnnotationHandler() {
        super(RequiresPermissions.class);
    }

    protected String[] getAnnotationValue(Annotation a) {
        RequiresPermissions rpAnnotation = (RequiresPermissions) a;
        return rpAnnotation.value();
    }
	//断言方法, 最终的处理逻辑
    public void assertAuthorized(Annotation a) throws AuthorizationException {
        if (!(a instanceof RequiresPermissions)) return;

        RequiresPermissions rpAnnotation = (RequiresPermissions) a;
        String[] perms = getAnnotationValue(a);
        Subject subject = getSubject();

        if (perms.length == 1) {
            subject.checkPermission(perms[0]);
            return;
        }
        if (Logical.AND.equals(rpAnnotation.logical())) {
            getSubject().checkPermissions(perms);
            return;
        }
        if (Logical.OR.equals(rpAnnotation.logical())) {
            boolean hasAtLeastOnePermission = false;
            for (String permission : perms) if (getSubject().isPermitted(permission)) hasAtLeastOnePermission = true;
            if (!hasAtLeastOnePermission) getSubject().checkPermission(perms[0]);
            
        }
    }
}


```

### 自定义实现注解的理解

使用模板模式核心的是配置`DefaultAdvisorAutoProxyCreator`和继承`StaticMethodMatcherPointcutAdvisor`。其中的5中权限注解，使用了统一一套代码架构，用到了的模板模式，方便扩展。

* `DefaultAdvisorAutoProxyCreator`类会自动找到IOC中的所有Advisor
  * 我们配置了StaticMethodMatcherPointcutAdvisor便是一个Advisor
  * 然后会将Advisor(一个`Advisor`是一个切入点和一个通知的组成)，将这些`Advisor`应用到所有符合切入点的Bean中。

- StaticMethodMatcherPointcutAdvisor这是一个Advisor,
  - 方法setAdvice定义了通知类(Adivce)
  - 重写了matche,返回true的bena或method都会被增强,自动创建代理对象

- 实现MethodInterceptor接口,重写invoke ,spring会自动调用器invoke

- 需要配置`DefaultAdvisorAutoProxyCreator`才能够自动创建代理



#### 自定义

* 定义一个注解

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
	String value() default "";
}
```

* 核心 继承`StaticMethodMatcherPointcutAdvisor`类，并实现相关的方法。

```java
@SuppressWarnings("serial")
@Component
public class HelloAdvisor extends StaticMethodMatcherPointcutAdvisor{
	//设置Advice,调用器invoke方法
    public HelloAdvisor() {
        setAdvice(new LogMethodInterceptor());
    }
	//匹配规则返回true 则需要增强
    public boolean matches(Method method, Class targetClass) {
        Method m = method;
        if ( isAuthzAnnotationPresent(m) ) {
            return true;
        }

        if ( targetClass != null) {
            try {
                m = targetClass.getMethod(m.getName(), m.getParameterTypes());
                return isAuthzAnnotationPresent(m);
            } catch (NoSuchMethodException ignored) {
               
            }
        }
        return false;
    }
	
    private boolean isAuthzAnnotationPresent(Method method) {
        Annotation a = AnnotationUtils.findAnnotation(method, Log.class);
        return a!= null;
    }
}


```

* 实现`MethodInterceptor`接口，定义切面处理的逻辑

```java
public class LogMethodInterceptor implements MethodInterceptor{

	public Object invoke(MethodInvocation invocation) throws Throwable {
		Log log = invocation.getMethod().getAnnotation(Log.class);
		System.out.println("log: "+log.value());
		return invocation.proceed();	
	}
}

```

* 定义一个测试类，并添加Log注解

```java
@Component
public class TestHello {

	@Log("test log")
	public String say() {
		return "ss";
	}
}

```

* 编写启动类，并且配置`DefaultAdvisorAutoProxyCreator`

```java
@Configuration
public class TestBoot {

	public static void main(String[] args) {
		ApplicationContext ctx = new AnnotationConfigApplicationContext("com.fzsyw.test");	
		TestHello th = ctx.getBean(TestHello.class);
		System.out.println(th.say());
	}
	
	@Bean
	public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator(){
		DefaultAdvisorAutoProxyCreator da = new DefaultAdvisorAutoProxyCreator();
		da.setProxyTargetClass(true);
		return da;
	}
}

```



# Shiro整合缓存

整合Redis存储用户信息,查询用户信息时,优先从缓存查询,不存在则查询数据库,提高效率,降低数据库压力;

### 继承缓存的思路

![image-20210422205826029](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210422205826.png)

### Redisson的集成

引入Redisson的依赖,在配置文件中指定属性,通过Redisson中的Config类构建对应的缓存对象,其中有多种缓存模式;

* 指定Redisson的属性在配置文件中
* Redisson有单节点与集群模式两种,对应两个实现类
* 通过Config类,来构造对象;

因为认证,鉴权的使用频率较高,可以使用与业务两套Redis来保证业务Redis的吞吐量;

此处使用的是与业务不同的一套Redis配置

#### 配置Redisson

```java
/**
 * @Description  redis配置文件
 */
@Data
@ConfigurationProperties(prefix = "itheima.framework.shiro.redis")
public class ShiroRedisProperties implements Serializable {

	/**
	 * redis连接地址
	 */
	private String nodes ;

	/**
	 * 获取连接超时时间
	 */
	private int connectTimeout ;

	/**
	 * 连接池大小
	 */
	private int connectPoolSize;

	/**
	 * 初始化连接数
	 */
	private int connectionMinimumidleSize ;

	/**
	 * 等待数据返回超时时间
	 */
	private int timeout ;

	/**
	 *  全局超时时间
	 */
	private long globalSessionTimeout;

}
```

```java
    //在shiroConfig中配置缓存

	@Autowired
    private ShiroRedisProperties shiroRedisProperties;

    /**
     * @Description redission客户端
     */
    @Bean("redissonClientForShiro")
    public RedissonClient redissonClient() {
        log.info("=====初始化redissonClientForShiro开始======");
        String[] nodeList = shiroRedisProperties.getNodes().split(",");
        Config config = new Config();
        if (nodeList.length == 1) {
            //单节点配置
            config.useSingleServer().setAddress(nodeList[0])
                    .setConnectTimeout(shiroRedisProperties.getConnectTimeout())
.setConnectionMinimumIdleSize(shiroRedisProperties.getConnectionMinimumidleSize())
.setConnectionPoolSize(shiroRedisProperties.getConnectPoolSize()).setTimeout(shiroRedisProperties.getTimeout());
        } else {
            //多节点 集群配置
            config.useClusterServers().addNodeAddress(nodeList)
                    .setConnectTimeout(shiroRedisProperties.getConnectTimeout())
.setMasterConnectionMinimumIdleSize(shiroRedisProperties.getConnectionMinimumidleSize())
.setMasterConnectionPoolSize(shiroRedisProperties.getConnectPoolSize())
.setTimeout(shiroRedisProperties.getTimeout());
        }
        //创建Redisson缓存对象
        RedissonClient redissonClient =  Redisson.create(config);
        log.info("=====初始化redissonClientForShiro完成======");
        return redissonClient;
    }

```

### 实现Cache

Shiro提供了Cache接口,Shiro默认提供了一个其实现类MapCache,但是并未实现序列化

* 我们需要实现Cache<Object,Object>接口,并实现序列化接口;并重写方法
* 也可以继承MapCache<K,V>类,并实现序列化接口.可以省去重写部分方法

### 缓存管理工具

工具类,主要用来对缓存中的数据操作,管理

* 此工具类可以作用在UserService查询数据库之前从缓存中查询. 登陆成功后可以通过此方法将用户数据放到缓存中

```java
//具体实现可根据个人需要制定
/**
 * @Description 简单的缓存管理接口
 */
public interface SimpleCacheService {

    /**
     * <b>功能说明：</b>：新增缓存堆到管理器<br>
     */
     void createCache(String cacheName, Cache<Object, Object> cache) throws CacheException;

    /**
     * <b>方法名：</b>：getCache<br>
     * <b>功能说明：</b>：获取缓存堆<br>
     */
     Cache<Object, Object> getCache(String cacheName) throws CacheException;

    /**
     * <b>方法名：</b>：removeCache<br>
     * <b>功能说明：</b>：移除缓存堆<br>
     */
     void removeCache(String cacheName) throws CacheException;

    /**
     * <b>方法名：</b>：updateCahce<br>
     * <b>功能说明：</b>：更新缓存堆<br>
     */
     void updateCahce(String cacheName, Cache<Object, Object> cache) throws CacheException;
}

```

### 缓存清理

用户退出听该清理缓存,否则会产生大量的缓存垃圾.

* 在重写的Realm中覆盖重写父类方法doClearCache(), 在退出时,Shiro会调用此方法

```java
    /**
     * @Description 清理缓存
     */
    @Override
    public void doClearCache(PrincipalCollection principalcollection) {
        //自定义的缓存管理工具,具体可以自行根据需求实现
        String sessionId = ShiroUtil.getShiroSessionId();
        simpleCacheManager.removeCache(CacheConstant.ROLE_KEY+sessionId);
        simpleCacheManager.removeCache(CacheConstant.RESOURCES_KEY+sessionId);
        simpleCacheManager.removeCache(CacheConstant.TOKEN+sessionId);
    }
```



# 分布式会话

## Session共享

* 所有服务器的session信息都存储到了同一个Redis集群中，即所有的服务都将 Session 的信息存储到 Redis 集群中，无论是对 Session 的注销、更新都会同步到集群中，达到了 Session 共享的目的。

![image-20210422225614064](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210422225614.png)

​	Cookie 保存在客户端浏览器中，而 Session 保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是 Session。客户端浏览器再次访问时只需要从该 Session 中查找该客户的状态就可以了。

​		在实际工作中我们建议使用外部的缓存设备(包括Redis)来共享 Session，避免单个服务器节点挂掉而影响服务，共享数据都会放到外部缓存容器中

![image-20210422225636252](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210422225636.png)

#### 原理

主要依赖DefaultWebSessionManager来完成Session共享机制,

通过继承AbstractSessionDao类重写方法,并配置到DefaultWebSessionManager来完成

自定义SessionDao类需要配置缓存客户端(Redis)信息

- 在ShiroConfig中配置DefaultWebSessionManager对象
- 此对象默认使用内存实现类,需要替换成自定义的实现类
- 自定义的SessionDao需要继承AbstractSessionDao,然后重写方法

#### ShiroCofig

```java
    
    /**
     * @Description 创建cookie对象
     */
    @Bean(name="sessionIdCookie")
    public SimpleCookie simpleCookie(){
        //生产cookie的对象,shiro提供
        SimpleCookie simpleCookie = new SimpleCookie();
        simpleCookie.setName("ShiroSession");
        return simpleCookie;
    }
	/**
     * @Description 自定义session会话存储的实现类 ，使用Redis来存储共享session，达到分布式部署目的
     */
    @Bean("redisSessionDao")
    public SessionDAO redisSessionDao(){
        //自定义的sessionDao
        RedisSessionDao sessionDAO =   new RedisSessionDao();
        //超时时间
        sessionDAO.setGlobalSessionTimeout(shiroRedisProperties.getGlobalSessionTimeout());
        return sessionDAO;
    }

    /**
     * @Description 会话管理器
     */
    @Bean(name="sessionManager")
    public DefaultWebSessionManager shiroSessionManager(){
        //配置
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        //设置自定义的SessionDao
        sessionManager.setSessionDAO(redisSessionDao());
        sessionManager.setSessionValidationSchedulerEnabled(false);
        sessionManager.setSessionIdCookieEnabled(true);
        sessionManager.setSessionIdCookie(simpleCookie());
        sessionManager.setGlobalSessionTimeout(shiroRedisProperties.getGlobalSessionTimeout());
        return sessionManager;
    }
```

#### 自定义SessionDao

```java
package com.itheima.shiro.core.impl;

import com.itheima.shiro.constant.CacheConstant;
import com.itheima.shiro.utils.ShiroRedissionSerialize;
import lombok.extern.log4j.Log4j2;
import org.apache.shiro.session.Session;
import org.apache.shiro.session.mgt.eis.AbstractSessionDAO;
import org.redisson.api.RBucket;
import org.redisson.api.RedissonClient;

import javax.annotation.Resource;
import java.io.Serializable;
import java.util.Collection;
import java.util.Collections;
import java.util.concurrent.TimeUnit;

/**
 * @Description 实现shiro session的memcached集中式管理~
 */
@Log4j2
public class RedisSessionDao extends AbstractSessionDAO {
	/**
	 * Redis客户端 
	 */
	@Resource(name = "redissonClientForShiro")
	RedissonClient redissonClient;

	private Long globalSessionTimeout;
	/**
	 * 创建一个唯一SesisonId放入缓存, 并返回
	 */
	@Override
	protected Serializable doCreate(Session session) {
       //此方法来自AbstractSessionDAO,创建唯一标识的SessionId
		Serializable sessionId = generateSessionId(session);
        //为会话指定唯一Sesisonid
		assignSessionId(session, sessionId);
        //从缓存中获取  CacheConstant.GROUP_CAS 为自定义key
        RBucket<String> sessionIdRBucket = redissonClient.getBucket(CacheConstant.GROUP_CAS+sessionId.toString());
        sessionIdRBucket.trySet(ShiroRedissionSerialize.serialize(session), globalSessionTimeout, TimeUnit.SECONDS);
		return sessionId;
	}
	/**
	 * 从缓存中读取Sessio
	 */
	@Override
	protected Session doReadSession(Serializable sessionId) {
		RBucket<String> sessionIdRBucket = redissonClient.getBucket(CacheConstant.GROUP_CAS+sessionId.toString());
		Session session = (Session) ShiroRedissionSerialize.deserialize(sessionIdRBucket.get());
		return session;
	}
	/**
	 * 删除Session
	 */
	@Override
	public void delete(Session session) {
		RBucket<String> sessionIdRBucket = redissonClient.getBucket(CacheConstant.GROUP_CAS+session.getId().toString());
		sessionIdRBucket.delete();
	}
	/**
	 * 获取活跃的
	 */
	@Override
	public Collection<Session> getActiveSessions() {
        //暂未实现
		return Collections.emptySet();  
	}
	/**
	 * 更新Session
	 */
	@Override
	public void update(Session session) {
		RBucket<String> sessionIdRBucket = redissonClient.getBucket(CacheConstant.GROUP_CAS+session.getId().toString());
		sessionIdRBucket.set(ShiroRedissionSerialize.serialize(session), globalSessionTimeout, TimeUnit.SECONDS);
	}

	public void setGlobalSessionTimeout(Long globalSessionTimeout) {
		this.globalSessionTimeout = globalSessionTimeout;
	}
}


```

# 管理限制

## 限制密码重试次数

使用自定义实现密码比较器,在密码比较器中整合Redission客户端,通过在缓存中设置值标记密码输入错误次数进行限制;

修改Realm配置,初始化修改密码比较器为自定义的密码比较器

### 实现原理

保证原子性：

​		单系统：AtomicLong计数

​		集群系统：RedissionClient提供的RAtomicLong计数

```java
1、获取系统中是否已有登录次数缓存,缓存对象结构预期为："用户名--登录次数"。

2、如果之前没有登录缓存，则创建一个登录次数缓存。

3、如果缓存次数已经超过限制，则驳回本次登录请求。

4、将缓存记录的登录次数加1,设置指定时间内有效

5、验证用户本次输入的帐号密码，如果登录登录成功，则清除掉登录次数的缓存
```

AuthenticatingRealm里有比较密码的入口doCredentialsMatch方法

![image-20210424215821384](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210424215821.png)

![image-20210424215830838](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210424215830.png)

### 自定义密码比较器

RAtomicLong是Redisson提供的原子类

```java
/**
 * @Description：密码重试比较器
 */
public class RetryLimitCredentialsMatcher extends HashedCredentialsMatcher {

    private RedissonClient redissonClient; //整合Redisson

    private static Long RETRY_LIMIT_NUM = 4L; //密码限制次数

    /**
     * @Description 构造函数
     * @param hashAlgorithmName 匹配次数
     */
    public RetryLimitCredentialsMatcher(String hashAlgorithmName,RedissonClient redissonClient) {
        super(hashAlgorithmName);
        this.redissonClient = redissonClient;
    }

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        //获得登录名
        String loginName = (String) token.getPrincipal();
        //获得缓存数据
        RAtomicLong atomicLong = redissonClient.getAtomicLong(loginName);
        long retryFlag = atomicLong.get();
        //判断次数
        if (retryFlag>RETRY_LIMIT_NUM){
            //超过次数设计10分钟后重试
            atomicLong.expire(10, TimeUnit.MICROSECONDS);
            throw new ExcessiveAttemptsException("密码错误5次，请10分钟以后再试");
        }
        //累加次数
        atomicLong.incrementAndGet();
        atomicLong.expire(10, TimeUnit.MICROSECONDS);
        //密码校验
        boolean flag =  super.doCredentialsMatch(token, info);
        if (flag){
            //校验成功删除限制
            atomicLong.delete();
        }
        return flag;
    }
}

```

#### 重写ShiroDbRealmImpl

修改initCredentialsMatcher方法，使用RetryLimitCredentialsMatcher

```java
/**
 * @Description：自定义shiro的实现
 */
public class ShiroDbRealmImpl extends ShiroDbRealm {

    @Autowired
    private UserBridgeService userBridgeService;

    @Autowired
    private SimpleCacheManager simpleCacheManager;

    @Resource(name = "redissonClientForShiro")
    private RedissonClient redissonClient;

    /**
     * @Description 认证方法
     * @param authcToken 校验传入令牌
     * @return AuthenticationInfo
     */
    @Override
    public AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) {
        SimpleToken token = (SimpleToken)authcToken;
        User user  = userBridgeService.findUserByLoginName(token.getUsername());
        if(EmptyUtil.isNullOrEmpty(user)){
            throw new UnknownAccountException("账号不存在");
        }
        ShiroUser shiroUser = BeanConv.toBean(user, ShiroUser.class);
        String sessionId = ShiroUserUtil.getShiroSessionId();
        String cacheKeyResourcesIds = CacheConstant.RESOURCES_KEY_IDS+sessionId;
        shiroUser.setResourceIds(userBridgeService.findResourcesIdsList(cacheKeyResourcesIds,user.getId()));
        String salt = user.getSalt();
        String password = user.getPassWord();
        return new SimpleAuthenticationInfo(shiroUser, password, ByteSource.Util.bytes(salt), getName());
    }

    /**
     * @Description 授权方法
     * @param principals SimpleAuthenticationInfo对象第一个参数
     * @return
     */
    @Override
    public AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        ShiroUser shiroUser = (ShiroUser) principals.getPrimaryPrincipal();
        return userBridgeService.getAuthorizationInfo(shiroUser);
    }

    /**
     * @Description 清理缓存
     */
    @Override
    public void doClearCache(PrincipalCollection principalcollection) {
        String sessionId = ShiroUtil.getShiroSessionId();
        simpleCacheManager.removeCache(CacheConstant.ROLE_KEY+sessionId);
        simpleCacheManager.removeCache(CacheConstant.RESOURCES_KEY+sessionId);
        simpleCacheManager.removeCache(CacheConstant.TOKEN+sessionId);
    }

    /**
     * @Description 加密方式,密码匹配器
     */
    @Override
    public void initCredentialsMatcher() {
        RetryLimitCredentialsMatcher matcher = new RetryLimitCredentialsMatcher(SuperConstant.HASH_ALGORITHM,redissonClient);
        matcher.setHashIterations(SuperConstant.HASH_INTERATIONS);
        setCredentialsMatcher(matcher);

    }
}

```

## 限制登录人数

一个账号只允许同时一个在线，当账号在其他地方登陆的时候，会踢出前面登陆的账号实现

- 自定义过滤器:继承AccessControlFilter

- 使用redis队列控制账号在线数目

  #### 原理

```properties
1、只针对登录用户处理，首先判断是否登录
2、使用RedissionClien创建队列
3、判断当前sessionId是否存在于此用户的队列=key:登录名 value：多个sessionId
4、不存在则放入队列尾端==>存入sessionId
5、判断当前队列大小是否超过限定此账号的可在线人数
6、超过：
	*从队列头部拿到用户sessionId
	*从sessionManger根据sessionId拿到session
	*从sessionDao中移除session会话 // 删除session时,如果session已过期或失效,则应该对应处理
7、未超过：放过操作
```

#### 代码实现

```java
/**
 * @Description：
 */
@Log4j2
public class KickedOutAuthorizationFilter extends AccessControlFilter {

    private RedissonClient redissonClient;

    private SessionDAO redisSessionDao;

    private DefaultWebSessionManager sessionManager;

    public KickedOutAuthorizationFilter(RedissonClient redissonClient, SessionDAO redisSessionDao, DefaultWebSessionManager sessionManager) {
        this.redissonClient = redissonClient;
        this.redisSessionDao = redisSessionDao;
        this.sessionManager = sessionManager;
    }

    @Override
    protected boolean isAccessAllowed(ServletRequest servletRequest, ServletResponse servletResponse, Object o) throws Exception {
        return false;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        Subject subject = getSubject(servletRequest, servletResponse);
        if (!subject.isAuthenticated()) {
            //如果没有登录，直接进行之后的流程
            return true;
        }
        //存放session对象进入队列
        String sessionId = ShiroUserUtil.getShiroSessionId();
        String LoginName = ShiroUserUtil.getShiroUser().getLoginName();
        RDeque<String> queue = redissonClient.getDeque("KickedOutAuthorizationFilter:"+LoginName);
        //判断sessionId是否存在于此用户的队列中
        boolean flag = queue.contains(sessionId);
        if (!flag) {
            queue.addLast(sessionId);
        }
        //如果此时队列大于1，则开始踢人
        if (queue.size() > 1) {
            sessionId = queue.getFirst();
            queue.removeFirst();
            Session session = null;
            try {
                session = sessionManager.getSession(new DefaultSessionKey(sessionId));
            }catch (UnknownSessionException ex){
                log.info("session已经失效");
            }catch (ExpiredSessionException expiredSessionException){
                log.info("session已经过期");
            }
            if (!EmptyUtil.isNullOrEmpty(session)){
                redisSessionDao.delete(session);
            }
        }
        return true;
    }
}
```

配置ShiroConfig

```java
/**
  * @Description 自定义过滤器定义
  */
private Map<String, Filter> filters() {
    Map<String, Filter> map = new HashMap<String, Filter>();
    map.put("roleOr", new RolesOrAuthorizationFilter());
    map.put("kickedOut", new KickedOutAuthorizationFilter(redissonClient(), redisSessionDao(), shiroSessionManager()));
    return map;
}
```

# SpringBoot前后分离鉴权

基于JWT的token解决方案

```java
1、用户登陆之后，获得此时会话的sessionId,使用JWT根据sessionId颁发签名并设置过期时间(与session过期时间相同)返回token

2、将token保存到客户端本地，并且每次发送请求时都在header上携带JwtToken

3、ShiroSessionManager继承DefaultWebSessionManager，重写getSessionId方法，从header上检测是否携带JwtToken，如果携带，则进行解码JwtToken，使用JwtToken中的jti作为SessionId。

4、重写shiro的默认过滤器，使其支持jwtToken有效期校验、及对JSON的返回支持
	JwtAuthcFilter:实现是否需要登录的过滤，拒绝时如果header上携带JwtToken,则返回对应json
	JwtPermsFilter:实现是否有对应资源的过滤，拒绝时如果header上携带JwtToken,则返回对应json
	JwtRolesFilter:实现是否有对应角色的过滤，拒绝时如果header上携带JwtToken,则返回对应json
```

## JWT

JWT（JSON WEB TOKEN）：JSON网络令牌，JWT是一个轻便的安全跨平台传输格式，定义了一个紧凑的自包含的方式在不同实体之间安全传输信息（JSON格式）。它是在Web环境下两个实体之间传输数据的一项标准。实际上传输的就是一个字符串。

- 广义上：JWT是一个标准的名称；

- 狭义上：JWT指的就是用来传递的那个token字符串


JWT由三部分构成：header（头部）、payload（载荷）和signature（签名）。

1. Header

   存储两个变量

   1. 秘钥（可以用来比对）
   2. 算法（也就是下面将Header和payload加密成Signature）

2. payload

   存储很多东西，基础信息有如下几个

   1. 签发人，也就是这个“令牌”归属于哪个用户。一般是`userId` 
   2. 创建时间，也就是这个令牌是什么时候创建的
   3. 失效时间，也就是这个令牌什么时候失效(session的失效时间)
   4. 唯一标识(jti)，一般可以使用算法生成一个唯一标识（jti==>sessionId）

3. Signature

   这个是上面两个经过Header中的算法加密生成的，用于比对信息，防止篡改Header和payload

然后将这三个部分的信息经过加密生成一个`JwtToken`的字符串，发送给客户端，客户端保存在本地。当客户端发起请求的时候携带这个到服务端(可以是在`cookie`，可以是在`header`)，在服务端进行验证，我们需要解密对于的payload的内容

## 集成JWT

- 通过配置文件指定签名

```java
package com.itheima.shiro.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

import java.io.Serializable;

/**
 * @Description：jw配置文件
 */
@Data
@ConfigurationProperties(prefix = "itheima.framework.jwt")
public class JwtProperties implements Serializable {

    /**
     * @Description 签名密码
     */
    private String hexEncodedSecretKey;
}
```

- JwtUtil

自定义封装Jwt包提供的工具的工具类,负责令牌的颁发、解析、校验

```java
package com.itheima.shiro.core.impl;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.itheima.shiro.config.JwtProperties;
import com.itheima.shiro.utils.EncodesUtil;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@Service("jwtTokenManager")
@EnableConfigurationProperties({JwtProperties.class})
public class JwtTokenManager {

    @Autowired
    JwtProperties jwtProperties;


    /**
     * @Description 签发令牌
     *      jwt字符串包括三个部分
     *        1. header
     *            -当前字符串的类型，一般都是“JWT”
     *            -哪种算法加密，“HS256”或者其他的加密算法
     *            所以一般都是固定的，没有什么变化
     *        2. payload
     *            一般有四个最常见的标准字段（下面有）
     *            iat：签发时间，也就是这个jwt什么时候生成的
     *            jti：JWT的唯一标识
     *            iss：签发人，一般都是username或者userId
     *            exp：过期时间
     * @param iss 签发人
     * @param ttlMillis 有效时间
     * @param claims jwt中存储的一些非隐私信息
     * @return
     */
    public String IssuedToken(String iss, long ttlMillis,String sessionId, Map<String, Object> claims) {
        if (claims == null) {
            claims = new HashMap<>();
        }
        long nowMillis = System.currentTimeMillis();

        String base64EncodedSecretKey = EncodesUtil.encodeHex(jwtProperties.getBase64EncodedSecretKey().getBytes());

        JwtBuilder builder = Jwts.builder()
                .setClaims(claims)
                .setId(sessionId)//2. 这个是JWT的唯一标识，一般设置成唯一的，这个方法可以生成唯一标识,此时存储的为sessionId,登录成功后回写
                .setIssuedAt(new Date(nowMillis))//1. 这个地方就是以毫秒为单位，换算当前系统时间生成的iat
                .setSubject(iss)//3. 签发人，也就是JWT是给谁的（逻辑上一般都是username或者userId）
                .signWith(SignatureAlgorithm.HS256, base64EncodedSecretKey);//这个地方是生成jwt使用的算法和秘钥
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date exp = new Date(expMillis);//4. 过期时间，这个也是使用毫秒生成的，使用当前时间+前面传入的持续时间生成
            builder.setExpiration(exp);
        }
        return builder.compact();
    }

    /**
     * @Description 解析令牌
     * @param jwtToken 令牌
     * @return
     */
    public Claims decodeToken(String jwtToken) {

        String base64EncodedSecretKey = EncodesUtil.encodeHex(jwtProperties.getBase64EncodedSecretKey().getBytes());

        // 得到 DefaultJwtParser
        return Jwts.parser()
                // 设置签名的秘钥
                .setSigningKey(base64EncodedSecretKey)
                // 设置需要解析的 jwt
                .parseClaimsJws(jwtToken)
                .getBody();
    }

    /**
     * @Description 判断令牌是否合法
     * @param jwtToken 令牌
     * @return
     */
    public boolean isVerifyToken(String jwtToken) {

        String base64EncodedSecretKey = EncodesUtil.encodeHex(jwtProperties.getBase64EncodedSecretKey().getBytes());

        //这个是官方的校验规则，这里只写了一个”校验算法“，可以自己加
        Algorithm algorithm = Algorithm.HMAC256(EncodesUtil.decodeBase64(base64EncodedSecretKey));
        JWTVerifier verifier = JWT.require(algorithm).build();
        verifier.verify(jwtToken);  // 校验不通过会抛出异常
        //判断合法的标准：1. 头部和荷载部分没有篡改过。2. 没有过期
        return true;
    }

}
```







# 分布式统一权限系统



