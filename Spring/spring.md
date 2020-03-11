# **Spring**

## 一、ioc总结

ioc是一个容器，帮我们管理所有的组件；

1）依赖注入：`@Autowired`；自动赋值；

2）某个组件要使用spring提供的更多（IOC、AOP）必须加入到容器中；



体会：

1）容器启动。创建所有的单例bean；

2）autowired自动装配的时候，是从容器中找这些符合要求的bean；

3）ioc.getBean("bookService")；也是从容器中找到这个bean；

4）容器中包括了所有的bean；

5）调试spring源码，容器到底是什么？其实就是一个map

6）这个map中保存所有创建好的bean，并提供外界获取功能；



SpringIOC源码：

1）ioc容器的启动过程？启动期间做了什么（什么时候创建所有单实例Bean）

2）ioc是如何创建这些单实例Bean，并如何管理的；到底保存在了哪里？



1、ClassPathXmlApplicationContext.class

```java
ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");

this(new String[] {configLocation}, true, null);

public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        //所有单实例bean创建完成
        refresh();
    }
}
```

2、refresh()

BeanFactory：Bean工厂

![](images/BeanFactory.png)



```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      // Spring解析XML文件将要创建的所有bean的配置信息保存起来
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         // 支持国际化功能
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         // 留给子类的方法
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         // 初始化所有单实例bean对象
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

3、finishBeanFactoryInitialization(beanFactory);

`AbstractApplicationContext.class`：

```java
/**
	 * Finish the initialization of this context's bean factory,
	 * initializing all remaining singleton beans.
	 */
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration();

    // Instantiate all remaining (non-lazy-init) singletons.
    // 初始化所有单实例bean对象
    beanFactory.preInstantiateSingletons();
}
```

4、preInstantiateSingletons();

`DefaultListableBeanFactory.class`：

```java
public void preInstantiateSingletons() throws BeansException {
   if (logger.isTraceEnabled()) {
      logger.trace("Pre-instantiating singletons in " + this);
   }

   // Iterate over a copy to allow for init methods which in turn register new bean definitions.
   // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
   List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

   // Trigger initialization of all non-lazy singleton beans...
   //按顺序创建bean
   for (String beanName : beanNames) {
      //根据bean id获取到bean的定义信息
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         //是否是一个实现了FactoryBean接口的bean
         if (isFactoryBean(beanName)) {
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
               final FactoryBean<?> factory = (FactoryBean<?>) bean;
               boolean isEagerInit;
               if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                  isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                              ((SmartFactoryBean<?>) factory)::isEagerInit,
                        getAccessControlContext());
               }
               else {
                  isEagerInit = (factory instanceof SmartFactoryBean &&
                        ((SmartFactoryBean<?>) factory).isEagerInit());
               }
               if (isEagerInit) {
                  getBean(beanName);
               }
            }
         }
         else {
            getBean(beanName);
         }
      }
   }

   // Trigger post-initialization callback for all applicable beans...
   for (String beanName : beanNames) {
      Object singletonInstance = getSingleton(beanName);
      if (singletonInstance instanceof SmartInitializingSingleton) {
         final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
         if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
               smartSingleton.afterSingletonsInstantiated();
               return null;
            }, getAccessControlContext());
         }
         else {
            smartSingleton.afterSingletonsInstantiated();
         }
      }
   }
}
```

5、getBean(beanName);	创建bean的细节

`AbstractBeanFactory` : `doGetBean(name,requiredType,args,typeCheckOnly);`

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
      @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

   final String beanName = transformedBeanName(name);
   Object bean;

   // Eagerly check singleton cache for manually registered singletons.
   Object sharedInstance = getSingleton(beanName);
   if (sharedInstance != null && args == null) {
      if (logger.isTraceEnabled()) {
         if (isSingletonCurrentlyInCreation(beanName)) {
            logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                  "' that is not fully initialized yet - a consequence of a circular reference");
         }
         else {
            logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
         }
      }
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
   }

   else {
      // Fail if we're already creating this bean instance:
      // We're assumably within a circular reference.
      if (isPrototypeCurrentlyInCreation(beanName)) {
         throw new BeanCurrentlyInCreationException(beanName);
      }

      // Check if bean definition exists in this factory.
      BeanFactory parentBeanFactory = getParentBeanFactory();
      if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
         // Not found -> check parent.
         String nameToLookup = originalBeanName(name);
         if (parentBeanFactory instanceof AbstractBeanFactory) {
            return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                  nameToLookup, requiredType, args, typeCheckOnly);
         }
         else if (args != null) {
            // Delegation to parent with explicit args.
            return (T) parentBeanFactory.getBean(nameToLookup, args);
         }
         else if (requiredType != null) {
            // No args -> delegate to standard getBean method.
            return parentBeanFactory.getBean(nameToLookup, requiredType);
         }
         else {
            return (T) parentBeanFactory.getBean(nameToLookup);
         }
      }

      if (!typeCheckOnly) {
         markBeanAsCreated(beanName);
      }

      try {
         final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
         checkMergedBeanDefinition(mbd, beanName, args);

         // Guarantee initialization of beans that the current bean depends on.
         // 拿到创建当前bean之前需要提前创建的bean。depends-on属性；如果有就循环创建
         String[] dependsOn = mbd.getDependsOn();
         if (dependsOn != null) {
            for (String dep : dependsOn) {
               if (isDependent(beanName, dep)) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
               }
               registerDependentBean(dep, beanName);
               try {
                  getBean(dep);
               }
               catch (NoSuchBeanDefinitionException ex) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
               }
            }
         }

         // Create bean instance.
         if (mbd.isSingleton()) {
            sharedInstance = getSingleton(beanName, () -> {
               try {
                  return createBean(beanName, mbd, args);
               }
               catch (BeansException ex) {
                  // Explicitly remove instance from singleton cache: It might have been put there
                  // eagerly by the creation process, to allow for circular reference resolution.
                  // Also remove any beans that received a temporary reference to the bean.
                  destroySingleton(beanName);
                  throw ex;
               }
            });
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
         }

         else if (mbd.isPrototype()) {
            // It's a prototype -> create a new instance.
            Object prototypeInstance = null;
            try {
               beforePrototypeCreation(beanName);
               prototypeInstance = createBean(beanName, mbd, args);
            }
            finally {
               afterPrototypeCreation(beanName);
            }
            bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
         }

         else {
            String scopeName = mbd.getScope();
            final Scope scope = this.scopes.get(scopeName);
            if (scope == null) {
               throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
            }
            try {
               Object scopedInstance = scope.get(beanName, () -> {
                  beforePrototypeCreation(beanName);
                  try {
                     return createBean(beanName, mbd, args);
                  }
                  finally {
                     afterPrototypeCreation(beanName);
                  }
               });
               bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
            }
            catch (IllegalStateException ex) {
               throw new BeanCreationException(beanName,
                     "Scope '" + scopeName + "' is not active for the current thread; consider " +
                     "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                     ex);
            }
         }
      }
      catch (BeansException ex) {
         cleanupAfterBeanCreationFailure(beanName);
         throw ex;
      }
   }

   // Check if required type matches the type of the actual bean instance.
   if (requiredType != null && !requiredType.isInstance(bean)) {
      try {
         T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
         if (convertedBean == null) {
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
         }
         return convertedBean;
      }
      catch (TypeMismatchException ex) {
         if (logger.isTraceEnabled()) {
            logger.trace("Failed to convert bean '" + name + "' to required type '" +
                  ClassUtils.getQualifiedName(requiredType) + "'", ex);
         }
         throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
      }
   }
   return (T) bean;
}
```



6、getSingleton(String beanName, ObjectFactory<?> singletonFactory)：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    /** Cache of singleton objects: bean name to bean instance. */
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
}
```

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
   Assert.notNull(beanName, "Bean name must not be null");
   synchronized (this.singletonObjects) {
      Object singletonObject = this.singletonObjects.get(beanName);
      if (singletonObject == null) {
         if (this.singletonsCurrentlyInDestruction) {
            throw new BeanCreationNotAllowedException(beanName,
                  "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                  "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
         }
         if (logger.isDebugEnabled()) {
            logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
         }
         beforeSingletonCreation(beanName);
         boolean newSingleton = false;
         boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
         if (recordSuppressedExceptions) {
            this.suppressedExceptions = new LinkedHashSet<>();
         }
         try {
            singletonObject = singletonFactory.getObject();
            newSingleton = true;
         }
         catch (IllegalStateException ex) {
            // Has the singleton object implicitly appeared in the meantime ->
            // if yes, proceed with it since the exception indicates that state.
            singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
               throw ex;
            }
         }
         catch (BeanCreationException ex) {
            if (recordSuppressedExceptions) {
               for (Exception suppressedException : this.suppressedExceptions) {
                  ex.addRelatedCause(suppressedException);
               }
            }
            throw ex;
         }
         finally {
            if (recordSuppressedExceptions) {
               this.suppressedExceptions = null;
            }
            afterSingletonCreation(beanName);
         }
         if (newSingleton) {
            //添加创建的bean
            addSingleton(beanName, singletonObject);
         }
      }
      return singletonObject;
   }
}
```

创建好的对象最终会保存在一个map中；

ioc容器之一：保存单实例bean的地方，保存在一个map中；

`DefaultSingletonBeanRegistry`类中的`singletonObjects`



BeanFactory和ApplicationContext的区别：

1）ApplicationContext是BeanFactory的一个子接口；

2）BeanFactory，bean工厂接口；负责创建bean实例；容器里面保存的所有单例bean其实是一个map；

​		Spring最底层的接口；

​       ApplicationContext：是容器接口，更多的负责容器功能的实现；（可以基于beanFactory创建好的对象之上完成强大的容器）

​		容器可以从map获取这个bean，并且aop，di，在ApplicationContext接口下的这些类里面；





## 二、AOP

### 1、动态代理对象

利用jdk中的Proxy类进行反射创建目标对象：

```java
public class CalculatorProxy {

    public static Calculator getProxy(Calculator calculator) {

        //方法处理器
        InvocationHandler handler = new InvocationHandler() {
            /**
             *
             * @param proxy		交由jdk使用，我们不动
             * @param method	当前对象执行的方法对象
             * @param args	    当前执行方法中外界传入的参数列表
             * @return		    执行方法后返回的返回值
             * @throws Throwable
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object result = null;

                try {
                    LogUtils.logStart(method,args);
                    result = method.invoke(calculator, args);
                    LogUtils.logReturn(method,result);
                } catch (Exception e) {
                    LogUtils.logException(method,e);
                } finally {
                    LogUtils.logEnd(method);
                }
                //返回值必须返回出去外界才能拿到真正执行后的返回值
                return result;
            }
        };
        ClassLoader loader = calculator.getClass().getClassLoader();
        Class<?>[] classes = calculator.getClass().getInterfaces();
        //利用Proxy创建目标对象
        Object proxyInstance = Proxy.newProxyInstance(loader, classes, handler);

        return (Calculator) proxyInstance;
    }
}
```

```java
public class LogUtils {
    public static void logStart(Method method, Object[] args) {
        System.out.println("【" + method.getName() + "】方法开始执行了，传入的参数是：" + Arrays.asList(args));
    }

    public static void logReturn(Method method, Object result) {
        System.out.println("【" + method.getName() + "】方法正常执行完成，执行结果是：【" + result + "】");
    }

    public static void logException(Method method, Exception e) {
        System.out.println("【" + method.getName() + "】方法执行出现异常了！异常信息是：【原因：" + e.getCause() +"】；请联系技术人员！！");
    }

    public static void logEnd(Method method) {
        System.out.println("【" + method.getName() + "】方法最终执行结束！");
    }
}
```

注意：

Proxy.newProxyInstance(loader, classes, handler)方法中可以看出目标类必须要**实现接口**才可以创建代理对象；



### 2、写配置

1）将目标类和切面类（封装了通知方法（在目标方法执行前后执行的方法））加入到ioc容器中；

2）告诉Spring哪个是切面类`@Aspect`

3）告诉Spring，切面类的每一个方法，都是何时何地运行的

**切入点表达式**：

> 固定格式：**execution(访问权限符 返回值 方法全类名(参数列表))**
>
> 通配符：
>
> ​	*：
>
> ​		1）匹配一个或者多个字符：execution(public int com.huawei.impl.MyMath\*.*(int,int))
>
> ​		2）匹配任意一个参数：第一个参数是int类型，第二个参数是任意类型（只匹配两个参数）
>
> ​			execution(public int com.huawei.impl.MyMath\*r.*(int,\*))
>
> ​		3）只能匹配一层路径
>
> ​		4）权限位置*不能；权限位置不写就行；public【可选】
>
> ​	..：
>
> ​		1）匹配任意多个参数、任意参数类型
>
> ​		2）匹配任意多层路径
>
> ​			execution(public int com.huawei..MyMath\*.*(..))
>
> 记住两种：
>
> **最精确的**：execution(public int com.huawei.impl.MyMathCalculator.add(int,int))
>
> **最模糊的**：execution(* *(..))
>
> 
>
> 
>
> &&、||、!
>
> 
>
> **&&**：我们要切入的位置要满足这两个表达式；
>
> MyMathCalculator.add(int,double)满足第一个，不满足第二个
>
> execution(public int com.huawei.impl.MyMath\*.*(int,int))&&execution(\* \*.\*(int,int))
>
> 
>
> **||**：满足其中一个表达式即可；
>
> execution(public int com.huawei.impl.MyMath\*.*(int,int))||execution(\* \*.\*(int,int))
>
> 
>
> **!**：只要不是这个位置的都切；
>
> !execution(public int com.huawei.impl.MyMath\*.*(int,int))



**通知注解**

> 分别有5个通知注解：
>
> `@Before`：在目标方法执行之前执行；				**前置通知**
>
> `@After`：在目标方法结束后执行；					**后置通知**
>
> `@AfterReturning`：在目标方法正常返回之后执行；	**返回通知**
>
> `@AfterThrowing`：在目标方法抛出异常以后执行；		**异常通知**
>
> `@Around`：环绕；									**环绕通知**
>
> 通知方法的执行顺序：
>
> 1）正常执行：@Before（前置通知）-->@After（后置通知）-->@AfterReturning（方法正常返回）
>
> 2）异常执行：@Before（前置通知）-->@After（后置通知）-->@AfterThrowing（方法异常）
>
> ```java
> try{
>     @Before
> 	method.invoke(obj,args);
>     @AfterReturning
> }catch(Exception e){
>     @AfterThrowing
> }finally{
>     @After
> }
> ```



切面类：

```java
/**
 * 切面类
 */
@Component
@Aspect
public class LogUtils {

    /**
     * @param joinPoint     封装了当前目标方法的详细信息
     */
    @Before(value = "execution(public int com.huawei.impl.MyMath*.*(..))")
    public static void logStart(JoinPoint joinPoint) {
        //获取参数
        Object[] args = joinPoint.getArgs();
        //获取方法签名
        Signature signature = joinPoint.getSignature();
        String name = signature.getName();
        System.out.println("【"+name+"】方法开始执行了，传入的参数是："+Arrays.asList(args));
    }
	/**
	 * @@AfterReturning中的returning告诉spring我们用哪个参数接收返回值
	 */
    @AfterReturning(value = "execution(public int com.huawei.impl.MyMathCalculator.*(int,int))",returning = "result")
    public static void logReturn(JoinPoint joinPoint,Object result) {
        System.out.println("【"+joinPoint.getSignature().getName()+"】方法正常执行完成，执行结果是：【"+result+"】");
    }
	/**
	 * @AfterThrowing中的throwing告诉spring我们用哪个参数接收异常对象
	 */
    @AfterThrowing(value = "execution(public int com.huawei.impl.MyMathCalculator.*(int,int))",throwing = "exception")
    public static void logException(JoinPoint joinPoint,Exception exception) {
        System.out.println("【"+joinPoint.getSignature().getName()+"】方法执行出现异常了！异常信息是：【原因："+exception.getMessage()+"】；请联系技术人员！！");
    }

    @After(value = "execution(public int com.huawei.impl.MyMathCalculator.*(int,int))")
    public static void logEnd(JoinPoint joinPoint) {
        System.out.println("【"+joinPoint.getSignature().getName()+"】方法最终执行结束！");
    }
}
```

获取目标方法的详细信息

1）只需要在通知方法的参数列表中写上一个参数

​	JoinPoint joinPoint：封装了当前目标方法的详细信息；

2）Spring对通知方法不严格，但是参数列表不能乱写；





目标类：

```java
@Component
public class MyMathCalculator implements Calculator {

    @Override
    public int add(int x, int y) {
        int result = x + y;
        return result;
    }

    @Override
    public int sub(int x, int y) {
        int result = x - y;
        return result;
    }

    @Override
    public int mul(int x, int y) {
        int result = x * y;
        return result;
    }

    @Override
    public int div(int x, int y) {
        int result = x / y;
        return result;
    }
}
```



配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.huawei" />

    <!--开启基于注解的AOP功能；导入AOP名称空间-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>

</beans>
```



总结：

1）Spring配置AOP实现切面编程时，不需要实现接口也能创建代理对象；ioc.getBean(MyMathCalculator.class);cglib帮我们创建好了代理对象（class com.huawei.impl.MyMathCalculator$$EnhancerBySpring**CGLIB**$$a9947700）

2）从ioc获取对象时，获取的是代理对象而不是本类对象，因此，在获取对象getBean时，类型写的应该是接口的对象类型而不是本类；ioc.getBean(Calculator.class);

我们也可以通过id来获取对象：ioc.getBean("myMathCalculator");



### 3、环绕通知

关于切入点表达式的重用：

1）定义一个空方法；

2）在方法上添加`@Pointcut`注解

3）写入切入点表达式；

4）其他通知注解应用时只需输入方法的名字即可；

```java
@Pointcut(value = "execution(public int com.huawei.impl.MyMath*.*(..))")
public void pointCut(){}

@Before("pointCut()")
public static void logStart(JoinPoint joinPoint) {
```



------



@Around：环绕，动态代理，是Spring强大的通知；

四合一通知就是环绕通知

环绕通知中有一个参数： ProceedingJointPoint pjp

环绕通知是优于普通通知执行，执行顺序：

```java
[普通前置]

{

	try{

		环绕前置

		环绕执行：目标方法执行

		环绕返回

	}catch(){

		环绕异常

	}finally{

		环绕后置

	}

}

[普通后置]

[普通方法返回/方法异常]



新的顺序：

环绕前置---普通前置---目标方法执行---环绕返回/异常---环绕后置---普通后置---普通返回/异常

```





```java
@Around(value = "pointCut()")
    public static Object myAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Around开始执行。。。");
        //获取参数
        Object[] args = pjp.getArgs();

        String name = pjp.getSignature().getName();

        Object result = null;
        try {
            //@Before
            System.out.println("【环绕前置通知】【"+name+"方法开始】");
            //method.invoke(obj,args);
            result = pjp.proceed(args);
            //@AfterReturning
            System.out.println("【环绕返回通知】【"+name+"方法返回，返回值"+result+"】");
        } catch (Exception e) {
            //@AfterThrowing
            System.out.println("【环绕异常通知】【"+name+"】方法出现异常，异常信息："+e);
        } finally {
            //@After
            System.out.println("【环绕后置通知】【"+name+"】方法结束");
        }

        //反射调用后的返回值一定要返回出去
        return result;
    }
```



## 三、事务

### 1、步骤

1）、配置事务管理器（切面）让其进行事务控制

```xml
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

2）、开启基于注解的事务控制模式；依赖tx名称空间

```xml
<tx:annotation-driven transaction-manager="dataSourceTransactionManager" />
```

3）、给事务方法添加注解`@Transactional`

```java
/**
     * 结账
     * @param username
     * @param isbn
     */
    @Transactional
    public void checkout(String username,String isbn){
        int price = bookDao.getPrice(isbn);
        bookDao.updateStock(isbn,1);
        bookDao.updateBalance(username,price);
        System.out.println("结账完成");
    }
```



### 2、事务细节

**isolation**-Isolation：事务的隔离级别

**propagation**-Propagation：事务的传播行为



**noRollbackFor**-Class[]：哪些异常 事务不回滚（可以让原来默认回滚的异常让它不回滚）

**noRollbackForClassName**-String[]（String全类名）

**rollbackFor**-Class[]：哪些异常 事务需要回滚

**rollbackForClassName**-String[]（String全类名）

> 异常分类：
>
> ​	运行时异常（非检查异常）：可以不用处理；默认都回滚
>
> ​	编译时异常（检查异常）：要么try-catch，要么在方法上声明throws；默认不回滚
>
> 事务的回滚：默认发生运行时异常都回滚，发生编译时异常不回滚



**timeout**-int（秒为单位）：超时；事务超出指定执行时长后自动终止并回滚

**readOnly**-boolean：设置事务为只读事务；可以进行事务优化：加快查询速度，不用管事务的那一堆操作了；

```java
org.springframework.dao.TransientDataAccessResourceException: PreparedStatementCallback; SQL [UPDATE book_stock SET stock=stock-1 WHERE isbn=?]; Connection is read-only. Queries leading to data modification are not allowed;
```



### 3、事务的传播行为

| 传播属性      | 描述                                                         |
| ------------- | :----------------------------------------------------------- |
| REQUIRED      | 如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行 |
| REQUIRES_NEW  | 当前的方法必须启动新事务，并在自己的事务内运行，如果有事务正在运行，应该将它挂起 |
| SUPPORTS      | 如果有事务在运行，当前的方法就在这个事务内运行。否则它可以不运行在事务中 |
| NOT_SUPPORTED | 当前的方法不应该运行在事务中。如果有运行的事务，将它挂起     |
| MANDATORY     | 当前的方法必须运行在事务内部，如果没有正在运行的事务，就抛出异常 |
| NEVER         | 当前的方法不应该运行在事务中。如果有运行的事务，就抛出异常   |
| NESTED        | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行。否则，就启动一个新的事务，并在自己的事务内运行。 |



## 4、事务的特性（ACID）

​         **1、原子性（Atomicity）：事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。**

　　 **2、一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。**

　　 **3、隔离性（Isolation）：同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。**

　　 **4、持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。**

 