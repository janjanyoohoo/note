# 启动过程

## 自动装配

### 模式注解

Stereotype Annotation俗称为模式注解，Spring中常见的模式注解有`@Service`，`@Repository`，`@Controller`等，它们都“派生”自`@Component`注解。我们都知道，凡是被`@Component`标注的类都会被Spring扫描并纳入到IOC容器中，那么由`@Component`派生的注解所标注的类也会被扫描到IOC容器中。下面我们主要通过自定义模式注解来了解`@Component`的“派生性”和“层次性”。

#### @Component “派生性”







# 启动过程扩展

## FactoryBean

FactoryBean 和 BeanFactory虽然名字很像，但是这两者是完全不同的两个概念，用途上也是天差地别。BeanFactory是一个Bean工厂，在一定程度上我们可以简单理解为它就是我们平常所说的Spring容器(注意这里说的是简单理解为容器)，它完成了Bean的创建、自动装配等过程，存储了创建完成的单例Bean。而FactoryBean通过名字看，我们可以猜出它是Bean，但它是一个特殊的Bean，

[来源:掘金]: https://juejin.cn/post/6844903954615107597	"作者：天堂同志"

## 用法

```
FactoryBean的特殊之处在于它可以向容器中注册两个Bean
一个是它本身
一个是FactoryBean.getObject()方法返回值所代表的Bean
```

自定义一个类CustomerFactoryBean，让它实现了FactoryBean接口，重写了接口中的两个方法，在getObejct()方法中，返回了一个UserService的实例对象；在getObjectType()方法中返回了UserService.class。然后在CustomerFactoryBean添加了注解@Component注解交给Spring管理。

### 解析

FactoryBean是一个特殊的Bean。

- 我们自定义的CustomerFactoryBean实现了FactoryBean接口，所以当CustomerFactoryBean被扫描进Spring容器时，实际上它向容器中注册了两个bean，一个是CustomerFactoryBean类的单例对象；另外一个就是getObject()方法返回的对象，
- 我们重写的getObject()方法中，我们通过new UserService()返回了一个UserService的实例对象，所以我们从容器中能获取到UserService的实例对象。
- 如果我们想通过beanName去获取CustomerFactoryBean的单例对象，需要在beanName前面添加一个`&`符号，如下代码，这样就能根据beanName获取到原生对象了。

### 代码示例

```java
@Component
public class CustomerFactoryBean implements FactoryBean<UserService> {
    @Override
    public UserService getObject() throws Exception {
        return new UserService();
    }
    @Override
    public Class<?> getObjectType() {
        return UserService.class;
    }
}
```

定义了一个UserService类

```java
public class UserService {

    public UserService(){
        System.out.println("userService construct");
    }
}
```

定义了一个配置类AppConfig，在类中指明了Spring需要扫描包

```java
@Configuration
@ComponentScan("com.tiantang.study.component")
public class AppConfig {
}
```

启动类

```java
public class MainApplication {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        System.out.println("容器启动完成");
        //从容其中获取getBean的类
        UserService userService = applicationContext.getBean(UserService.class);
        System.out.println(userService);
        //此名称获取不到customerFactoryBean的bean, 需要添加$符,详情参考源码
        Object customerFactoryBean = applicationContext.getBean("customerFactoryBean");
        System.out.println(customerFactoryBean);
    }
}
```

### 源码

1. 在Spring容器启动阶段，会调用到`refresh()`方法，

2. 在`refresh()`中有调用了`finishBeanFactoryInitialization()`方法

3. 最终会调用到`beanFactory.preInstantiateSingletons()`方法

```java
/*
 * 单例对象实例化预处理方法
 */
public void preInstantiateSingletons() throws BeansException {
	// 从容器中获取到所有的beanName
	List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
	for (String beanName : beanNames) {
        //获取当前beanName的RootBeanDefinition对象
		RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        //不是抽象,是单例,不是懒加载
		if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
			// 在此处会根据beanName判断bean是不是一个FactoryBean，实现了FactoryBean接口的bean，会返回true
			// 此时当beanName为customerFactoryBean时，会返回true，会进入到if语句中
			if (isFactoryBean(beanName)) {
				// 然后通过getBean()方法去获取或者创建单例对象
				// 注意：在此处为beanName拼接了一个前缀：FACTORY_BEAN_PREFIX
				// FACTORY_BEAN_PREFIX是一个常量字符串，即：&
				// 所以在此时容器启动阶段，对于customerFactoryBean，应该是：getBean("&customerFactoryBean")
				Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
				// 下面这一段逻辑，是判断是否需要在容器启动阶段，就去实例化getObject()返回的对象，即是否调用FactoryBean的getObject()方法
				if (bean instanceof FactoryBean) {
					final FactoryBean<?> factory = (FactoryBean<?>) bean;
					boolean isEagerInit;
                    //如果bean实现了SmartFactoryBean接口,且isEagerInit()方法返回的是true,会在容器初始化时就调用getObject()方法
					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
						isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
										((SmartFactoryBean<?>) factory)::isEagerInit,
								getAccessControlContext());
					}
					else {
						isEagerInit = (factory instanceof SmartFactoryBean &&
								((SmartFactoryBean<?>) factory).isEagerInit());
					}
                    //isEagerInit()方法返回的是true, 调用getBean() -> 进入此方法看源码
					if (isEagerInit) {
						getBean(beanName);
					}
				}
			}
		}
	}
}
```

- 在容器启动阶段，会先通过getBean()方法来创建CustomerFactoryBean的实例对象。
  - 如果实现了SmartFactoryBean接口，且isEagerInit()方法返回的是true，那么在容器启动阶段，就会调用getObject()方法，向容器中注册getObject()方法返回值的对象。
  - 否则，只有当第一次获取getObject()返回值的对象时，才会去回调getObject()方法。

- 在getBean()中会调用到doGetBean()方法，下面为doGetBean()精简后的源码。从源码中我们发现，最终都会调用getObjectForBeanInstance()方法。

```java
/*
 * getBean()方法调用doGetBean()方法,最终调用getObjectForBeanInstance()方法
 */
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	final String beanName = transformedBeanName(name);
	Object bean;

	Object sharedInstance = getSingleton(beanName);
    // 最终都会调用getObjectForBeanInstance()方法
	if (sharedInstance != null && args == null) {
		bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
	}
	else {
		if (mbd.isSingleton()) {
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
		}
		else if (mbd.isPrototype()) {
			bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
		}
		else {
			bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
		}
		
	}
	return (T) bean;
}
```

- 在`getObjectForBeanInstance()`方法中会先判断bean是不是FactoryBean，如果不是，就直接返回Bean。
- **如果是FactoryBean，且name是以&符号开头，那么表示的是获取FactoryBean的原生对象**，也会直接返回。
- **如果name不是以&符号开头，那么表示要获取FactoryBean中getObject()方法返回的对象。**
  - 会先尝试从`FactoryBeanRegistrySupport`类的`factoryBeanObjectCache`这个缓存map中获取，如果缓存中存在，则返回
  - 如果不存在，则去调用`getObjectFromFactoryBean()`方法。getObjectForBeanInstance()方法的部分源码如下：

```java
protected Object getObjectForBeanInstance(
		Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {
	// 如果请求的bean是beanFactory的引用对象 &开头的名称 直接返回
	if (BeanFactoryUtils.isFactoryDereference(name)) {
		if (beanInstance instanceof NullBean) {
			return beanInstance;
		}
		if (!(beanInstance instanceof FactoryBean)) {
			throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
		}
	}
	// 如果bean不是factoryBean，那么会直接返回Bean
	// 或者bean是factoryBean但name是以&特殊符号开头的,此时表示要获取FactoryBean的原生对象。
	// 例如：如果name = &customerFactoryBean，那么此时会返回CustomerFactoryBean类型的bean
	if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
		return beanInstance;
	}
	// 如果是FactoryBean，那么先从cache中获取，如果缓存不存在，则会去调用FactoryBean的getObject()方法。
	Object object = null;
	if (mbd == null) {
		// 从缓存中获取。什么时候放入缓存的呢？在第一次调用getObject()方法时，会将返回值放入到缓存。
		object = getCachedObjectForFactoryBean(beanName);
	}
	if (object == null) {
		FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
		if (mbd == null && containsBeanDefinition(beanName)) {
			mbd = getMergedLocalBeanDefinition(beanName);
		}
		boolean synthetic = (mbd != null && mbd.isSynthetic());
		// 在getObjectFromFactoryBean()方法中最终会调用到getObject()方法
		object = getObjectFromFactoryBean(factory, beanName, !synthetic);
	}
	return object;
}
```

- `getObjectFromFactoryBean()`方法中，主要是通过调用`doGetObjectFromFactoryBean()`方法得到bean，然后对bean进行处理，最后放入缓存。
- 而且还会针对单例bean和非单例bean做区分处理，对于单例bean，会在创建完后，将其放入到缓存中，非单例bean则不会放入缓存，而是每次都会重新创建。

```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
	// 如果BeanFactory的isSingleton()方法返回值是true,表示getObject()返回值对象是单例的
	if (factory.isSingleton() && containsSingleton(beanName)) {
		synchronized (getSingletonMutex()) {
			// 再一次判断缓存中是否存在。(双重检测机制，和平时写线程安全的代码类似)
			Object object = this.factoryBeanObjectCache.get(beanName);
			if (object == null) {
				// 在doGetObjectFromFactoryBean()中才是真正调用getObject()方法
				object = doGetObjectFromFactoryBean(factory, beanName);
				Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
				if (alreadyThere != null) {
					object = alreadyThere;
				}
				else {
					// 下面是进行后置处理，和普通的bean的后置处理没有任何区别
					if (shouldPostProcess) {
						if (isSingletonCurrentlyInCreation(beanName)) {
							return object;
						}
						beforeSingletonCreation(beanName);
						try {
							object = postProcessObjectFromFactoryBean(object, beanName);
						}
						catch (Throwable ex) {
							throw new BeanCreationException(beanName,
									"Post-processing of FactoryBean's singleton object failed", ex);
						}
						finally {
							afterSingletonCreation(beanName);
						}
					}
					// 放入到缓存中
					if (containsSingleton(beanName)) {
						this.factoryBeanObjectCache.put(beanName, object);
					}
				}
			}
			return object;
		}
	}
	// 非单例
	else {
		Object object = doGetObjectFromFactoryBean(factory, beanName);
		if (shouldPostProcess) {
			try {
				object = postProcessObjectFromFactoryBean(object, beanName);
			}
			catch (Throwable ex) {
				throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
			}
		}
		return object;
	}
}
```

- `doGetObjectFromFactoryBean()`方法的逻辑比较简单，直接调用了`FactoryBean的getObject()`方法。部分源码如下

```java
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
			throws BeanCreationException {

	Object object;
	if (System.getSecurityManager() != null) {
		AccessControlContext acc = getAccessControlContext();
		try {
			object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
		}
		catch (PrivilegedActionException pae) {
			throw pae.getException();
		}
	}
	else {
		// 调用getObject()方法
		object = factory.getObject();
	}
	return object;
}
```

- Spring的代码实在是写的太好了，每个方法几乎都复用性比较高，这就导致了总是方法中套方法，层级比较深，所以最后以一张流程图总结下FactoryBean的创建流程。

![FactoryBean的创建流程](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210531202301.webp)

### 应用场景

- Spring-Mybatis插件原理
  - 单独使用Mybatis时。添加依赖，配置数据源，创建`SqlSessionFactory`，这样环境就搭建完成了。参考官方文档：[mybatis.org/mybatis-3/g…](https://mybatis.org/mybatis-3/getting-started.html)

  - 当我们在将Mybatis整合到Spring中时，也是添加mybatis的依赖，但还需要额外添加一个jar包：`mybatis-spring`，然后是配置数据源。最后还需要一个配置

如果你是通过XML配置的话，还需要如下配置：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

通过JavaConfig配置，那么需要进行如下配置：

```java
@Bean
public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    return sqlSessionFactoryBean;
}
```

我们发现，无论是XML还是JavaConfig，都是向容器中注册了一个`SqlSessionFactoryBean`。

从类名我们就能知道这是一个`FactoryBean`。当我们单独使用Mybatis时，需要创建一个`SqlSessionFactory`，然而当MyBatis和Spring整合时，却需要一个`SqlSessionFactoryBean`.

查看`SqlSessionFactoryBean`的源代码发现，实现了`FactoryBean`接口，并且重写了`getObejct`方法，通过`getObject()`方法向容器中注册了一个`SqlSessionFactory`。

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
	// ...省略其他代码
	
	public SqlSessionFactory getObject() throws Exception {
	if (this.sqlSessionFactory == null) {
	  afterPropertiesSet();
	}
	//返回一个sqlSessionFactory对象
	return this.sqlSessionFactory;
	}
}
```

`sqlSessionFactory`是`SqlSessionFactoryBean`的一个属性，它的赋值是在通过回调`afterPropertiesSet()`方法进行的。

（因为`SqlSessionFactoryBean`实现了`InitializingBean`接口，所以在Spring初始化Bean的时候，能回调`afterPropertiesSet()`方法）

```java
public void afterPropertiesSet() throws Exception {
    // buildSqlSessionFactory()方法会根据mybatis的配置进行初始化。
	this.sqlSessionFactory = buildSqlSessionFactory();
}
```

在Spring和MyBatis整合时，还有另外一个地方也利用到了FactoryBean。我们在开发时，通常会通过`MapperScan`注解来扫描我们Mapper文件。`@MapperScan`注解中，添加了`@Import(MapperScannerRegistrar.class)`，(关于Import注解的作用，可以看下笔者的另一篇文章：[参考文章](https://mp.weixin.qq.com/s/y_2Z9m0gevp-cMkEIflrwA)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(MapperScannerRegistrar.class) 
public @interface MapperScan {
}
```

在`MapperScannerRegistrar`的`registerBeanDefinitions()`方法中，会将我们定义的`Mapper`扫描出来，解析成`BeanDefinition`，注意，解析成`BeanDefinition`后，`beanClass`属性不再是我们定义的Mapper类的class了，而是被设置成了`MapperFactoryBean.class`。

这说明了我们定义的每一个Mapper接口，被加载进Spring后，最后都会对应一个`MapperFactoryBean`。

我们再看看`MapperFactoryBean`这个类干了哪些事。下面是`MapperFactoryBean`类的部分源码。从源码中，我们发现，它实现了`FactoryBean`接口，重写了接口中的三个方法。

在`getObject()`方法中，通过调用`getSqlSession().getMapper(this.mapperInterface)`返回了一个对象。这一行代码最终会调用到`MapperProxyFactory`的`newInstance()`方法，为每一个Mapper创建一个代理对象。

```java
public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {
	@Override
	public T getObject() throws Exception {
		return getSqlSession().getMapper(this.mapperInterface);
	}

	@Override
	public Class<T> getObjectType() {
		return this.mapperInterface;
	}

	@Override
	public boolean isSingleton() {
        // 返回true是为了让Mapper接口是一个单例的
		return true;
	}
}
```

`MapperProxyFactory`类的源码。最终是**调用JDK的动态代理来为我们定义的Mapper创建动态代理**。(MyBatis框架就是通过动态代理实现的dao层)

```java
public class MapperProxyFactory<T> {

  protected T newInstance(MapperProxy<T> mapperProxy) {
  	// JDK动态代理
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }

}
```

这样我们写的每一个Mapper接口都会对应一个`MapperFactoryBean`，每一个`MapperFactoryBean`的`getObject()`方法**最终会采用JDK动态代理创建一个对象**，**所以每一个Mapper接口最后都对应一个代理对象**，这样就实现了Spring和MyBatis的整合。



`mybatis-spring-boot-starter`中，会引入`mybatis-spring-boot-autoconfigure`这个jar包，MyBatis的自动配置就是通过这个jar包中的`MybatisAutoConfiguration`类实现的。从`MybatisAutoConfiguration`的源码中我们可以看到同样是通过`SqlSessionFactoryBean`的`getObject()`方法向容器中注册了一个`SqlSessionBean`。

```java
@Configuration
@ConditionalOnClass({SqlSessionFactory.class, SqlSessionFactoryBean.class})
@ConditionalOnBean({DataSource.class})
@EnableConfigurationProperties({MybatisProperties.class})
@AutoConfigureAfter({DataSourceAutoConfiguration.class})
public class MybatisAutoConfiguration {
	@Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        // 省略部分代码
        return factory.getObject();
    }
}
```

