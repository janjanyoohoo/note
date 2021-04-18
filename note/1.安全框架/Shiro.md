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

## Shiro入门与概念

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

## 登录源码流程

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



