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

## 自定义Realm

![image-20210418215616874](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210418215624.png)

#### * 原理分析

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

### 过滤器

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

* 实现AuthrizatoionFilter接口,重写isAccsessAllowed()方法
* 定义map:
  * key - String 为过滤器的名称
  * value - Filter: 传入自定义Filter对象
* ShiroFilterFactoryBean类方法setFilters(定义的map); 
* 返回ShiroFilterFactoryBean对象,交给IOC管理;即可

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

### Shiro配置代码

``` java
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
    @DependsOn("lifecycleBeanPostProcessor")
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

