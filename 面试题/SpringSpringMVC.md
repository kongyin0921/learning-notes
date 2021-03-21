# Spring/Spring MVC

## 90.为什么要使用 spring？

spring 是一个开源的轻量级 JavaBean 容器框架。使用 JavaBean 代替 EJB ，并提供了丰富的企业应用功能，降低应用开发的复杂性。

- 轻量：非入侵性的、所依赖的东西少、资源占用少、部署简单，不同功能选择不同的 jar 组合
- 容器：工厂模式实现对 JavaBean 进行管理，通过控制反转（IOC）将应用程序的配置和依赖性与应用代码分开
- 松耦合：通过 xml 配置或注解即可完成 bean 的依赖注入
- AOP：通过 xml 配置 或注解即可加入面向切面编程的能力，完成切面功能，如：日志，事务...的统一处理
- 方便集成：通过配置和简单的对象注入即可集成其他框架，如 Mybatis、Hibernate、Shiro...
- 丰富的功能：JDBC 层抽象、事务管理、MVC、Java Mail、任务调度、JMX、JMS、JNDI、EJB、动态语言、远程访问、Web Service... 

## 91.解释一下什么是 aop？

- **AOP相关的概念**

1） *Aspect* ：切面，切入系统的一个切面。比如事务管理是一个切面，权限管理也是一个切面；

2） *Join point* ：连接点，也就是可以进行横向切入的位置；

3） *Advice* ：通知，切面在某个连接点执行的操作(分为: *Before advice* , *After returning advice* , *After throwing advice* , *After (finally) advice* , *Around advice* )；

4） *Pointcut* ：切点，符合切点表达式的连接点，也就是真正被切入的地方；

- **AOP 的实现原理**

AOP分为静态AOP和动态AOP。

静态AOP是指AspectJ实现的AOP，他是将切面代码直接编译到Java类文件中。

动态AOP是指将切面代码进行动态织入实现的AOP。

Spring的AOP为动态AOP，实现的技术为： JDK提供的动态代理技术 和 CGLIB(动态字节码增强技术) 。尽管实现技术不一样，但 都是基于代理模式 ， 都是生成一个代理对象 。

1) JDK动态代理

主要使用到 InvocationHandler 接口和 Proxy.newProxyInstance() 方法。

 JDK动态代理要求被代理实现一个接口，只有接口中的方法才能够被代理 。

其方法是将被代理对象注入到一个中间对象，而中间对象实现InvocationHandler接口，

在实现该接口时，可以在 被代理对象调用它的方法时，在调用的前后插入一些代码。

而 Proxy.newProxyInstance() 能够利用中间对象来生产代理对象。

插入的代码就是切面代码。所以使用JDK动态代理可以实现AOP。

我们看个例子：

被代理对象实现的接口，只有接口中的方法才能够被代理：

```java
public interface UserService {
    public void addUser(User user);
    public User getUser(int id);
}
```

```java
public class UserServiceImpl implements UserService {
    public void addUser(User user) {
        System.out.println("add user into database.");
    }
    public User getUser(int id) {
        User user = new User();
        user.setId(id);
        System.out.println("getUser from database.");
        return user;
    }
}
```

代理中间类：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
public class ProxyUtil implements InvocationHandler {
    private Object target;    // 被代理的对象
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("do sth before....");
        Object result =  method.invoke(target, args);
        System.out.println("do sth after....");
        return result;
    }
    ProxyUtil(Object target){
        this.target = target;
    }
    public Object getTarget() {
        return target;
    }
    public void setTarget(Object target) {
        this.target = target;
    }
}
```

测试：

```java
import java.lang.reflect.Proxy;
import net.aazj.pojo.User;
public class ProxyTest {
    public static void main(String[] args){
        Object proxyedObject = new UserServiceImpl();    // 被代理的对象
        ProxyUtil proxyUtils = new ProxyUtil(proxyedObject);
        // 生成代理对象，对被代理对象的这些接口进行代理：UserServiceImpl.class.getInterfaces()
        UserService proxyObject = (UserService) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), 
                    UserServiceImpl.class.getInterfaces(), proxyUtils);
        proxyObject.getUser(1);
        proxyObject.addUser(new User());
    }
}
```

执行结果：

```java
do sth before....
getUser from database.
do sth after....
do sth before....
add user into database.
do sth after....
我们看到在 UserService接口中的方法 addUser 和 getUser方法的前面插入了我们自己的代码。这就是JDK动态代理实现AOP的原理。

我们看到该方式有一个要求， 被代理的对象必须实现接口，而且只有接口中的方法才能被代理 。
```

2）CGLIB （code generate libary）

字节码生成技术实现AOP，其实就是继承被代理对象，然后Override需要被代理的方法，在覆盖该方法时，自然是可以插入我们自己的代码的。

因为需要Override被代理对象的方法，所以自然CGLIB技术实现AOP时，就 必须要求需要被代理的方法不能是final方法，因为final方法不能被子类覆盖 。

我们使用CGLIB实现上面的例子：

```java
package net.aazj.aop;
import java.lang.reflect.Method;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
public class CGProxy implements MethodInterceptor{
    private Object target;    // 被代理对象
    public CGProxy(Object target){
        this.target = target;
    }
    public Object intercept(Object arg0, Method arg1, Object[] arg2, MethodProxy proxy) throws Throwable {
        System.out.println("do sth before....");
        Object result = proxy.invokeSuper(arg0, arg2);
        System.out.println("do sth after....");
        return result;
    }
    public Object getProxyObject() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());    // 设置父类
        // 设置回调
        enhancer.setCallback(this);    // 在调用父类方法时，回调 this.intercept()
        // 创建代理对象
        return enhancer.create();
    }
}
```

```java
public class CGProxyTest {
    public static void main(String[] args){
        Object proxyedObject = new UserServiceImpl();    // 被代理的对象
        CGProxy cgProxy = new CGProxy(proxyedObject);
        UserService proxyObject = (UserService) cgProxy.getProxyObject();
        proxyObject.getUser(1);
        proxyObject.addUser(new User());
    }
}
```

输出结果：

```java
do sth before....
getUser from database.
do sth after....
do sth before....
add user into database.
do sth after....
我们看到达到了同样的效果。
它的原理是生成一个父类 enhancer.setSuperclass( this.target.getClass()) 的子类 enhancer.create() ，然后对父类的方法进行拦截enhancer.setCallback( this) . 
对父类的方法进行覆盖，所以父类方法不能是final的。
```

3） 接下来我们看下spring实现AOP的相关源码：

```java
@SuppressWarnings("serial")
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
    @Override
    public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: " +
                        "Either an interface or a target is required for proxy creation.");
            }
            if (targetClass.isInterface()) {
                return new JdkDynamicAopProxy(config);
            }
            return new ObjenesisCglibAopProxy(config);
        }
        else {
            return new JdkDynamicAopProxy(config);
        }
    }
```

a从上面的源码我们可以看到：

```java
if (targetClass.isInterface()) {
                return new JdkDynamicAopProxy(config);
            }
            return new ObjenesisCglibAopProxy(config);
```

如果被代理对象实现了接口，那么就使用JDK的动态代理技术，反之则使用CGLIB来实现AOP，所以 Spring默认是使用JDK的动态代理技术实现AOP的 。

JdkDynamicAopProxy的实现其实很简单：

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {    
@Override
public Object getProxy(ClassLoader classLoader) {
    if (logger.isDebugEnabled()) {
        logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
    }
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised);
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```

- **Spring AOP的配置**

Spring中AOP的配置一般有两种方法，一种是使用 <aop:config> 标签在xml中进行配置，一种是使用注解以及@Aspect风格的配置。

1） 基于<aop:config>的AOP配置

下面是一个典型的事务AOP的配置：

```xml
<tx:advice id="transactionAdvice" transaction-manager="transactionManager"?>
    <tx:attributes >
        <tx:method name="add*" propagation="REQUIRED" />
        <tx:method name="append*" propagation="REQUIRED" />
        <tx:method name="insert*" propagation="REQUIRED" />
        <tx:method name="save*" propagation="REQUIRED" />
        <tx:method name="update*" propagation="REQUIRED" />
        <tx:method name="get*" propagation="SUPPORTS" />
        <tx:method name="find*" propagation="SUPPORTS" />
        <tx:method name="load*" propagation="SUPPORTS" />
        <tx:method name="search*" propagation="SUPPORTS" />
        <tx:method name="*" propagation="SUPPORTS" />
    </tx:attributes>
</tx:advice>
<aop:config>
    <aop:pointcut id="transactionPointcut" expression="execution(* net.aazj.service..*Impl.*(..))" />
    <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" />
</aop:config>
```

再看一个例子：

```xml
<bean id="aspectBean" class="net.aazj.aop.DataSourceInterceptor"/>
<aop:config>
    <aop:aspect id="dataSourceAspect" ref="aspectBean">
        <aop:pointcut id="dataSourcePoint" expression="execution(public * net.aazj.service..*.getUser(..))" />
        <aop:pointcut expression="" id=""/>
        <aop:before method="before" pointcut-ref="dataSourcePoint"/>
        <aop:after method=""/>
        <aop:around method=""/>
    </aop:aspect>
    <aop:aspect></aop:aspect>
</aop:config>
```

<aop:aspect> 配置一个切面；

<aop:pointcut>配置一个切点，基于切点表达式；

<aop:before>,<aop:after>,<aop:around>是定义不同类型的advise. a

spectBean 是切面的处理bean：

```java
public class DataSourceInterceptor {
    public void before(JoinPoint jp) {
        DataSourceTypeManager.set(DataSources.SLAVE);
    }
}
```

2) 基于注解和@Aspect风格的AOP配置

我们以事务配置为例：首先我们启用基于注解的事务配置

```xml
<!-- 使用annotation定义事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
```

然后扫描Service包：

```xml
<context:component-scan base-package="net.aazj.service,net.aazj.aop" />
```

最后在service上进行注解：

```java
@Service("userService")
@Transactional
public class UserServiceImpl implements UserService{
    @Autowired
    private UserMapper userMapper;
    @Transactional (readOnly=true)
    public User getUser(int userId) {
        System.out.println("in UserServiceImpl getUser");
        System.out.println(DataSourceTypeManager.get());
        return userMapper.getUser(userId);
    }
    public void addUser(String username){
        userMapper.addUser(username);
//        int i = 1/0;    // 测试事物的回滚
    }
    public void deleteUser(int id){
        userMapper.deleteByPrimaryKey(id);
//        int i = 1/0;    // 测试事物的回滚
    }
    @Transactional (rollbackFor = BaseBusinessException.class)
    public void addAndDeleteUser(String username, int id) throws BaseBusinessException{
        userMapper.addUser(username);
        this.m1();
        userMapper.deleteByPrimaryKey(id);
    }
    private void m1() throws BaseBusinessException {
        throw new BaseBusinessException("xxx");
    }
    public int insertUser(User user) {
        return this.userMapper.insert(user);
    }
}
```

搞定。这种事务配置方式，不需要我们书写pointcut表达式，而是我们在需要事务的类上进行注解。但是如果我们自己来写切面的代码时，还是要写pointcut表达式。下面看一个例子(自己写切面逻辑)：

首先去扫描 @Aspect 注解定义的 切面：

```xml
<context:component-scan base-package="net.aazj.aop" />
```

切面代码：

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
@Aspect    // for aop
@Component // for auto scan
@Order(0)  // execute before @Transactional
public class DataSourceInterceptor {
    @Pointcut("execution(public * net.aazj.service..*.get*(..))")
    public void dataSourceSlave(){};
    @Before("dataSourceSlave()")
    public void before(JoinPoint jp) {
        DataSourceTypeManager.set(DataSources.SLAVE);
    }
}
```

我们使用到了 @Aspect 来定义一个切面；

@Component是配合<context:component-scan/>，不然扫描不到；

@Order定义了该切面切入的顺序 ，因为在同一个切点，可能同时存在多个切面，那么在这多个切面之间就存在一个执行顺序的问题。

该例子是一个切换数据源的切面，那么他应该在 事务处理 切面之前执行，所以我们使用 @Order(0) 来确保先切换数据源，然后加入事务处理。

@Order的参数越小，优先级越高，默认的优先级最低：

```java
/**
 * Annotation that defines ordering. The value is optional, and represents order value
 * as defined in the {@link Ordered} interface. Lower values have higher priority.
 * The default value is {@code Ordered.LOWEST_PRECEDENCE}, indicating
 * lowest priority (losing to any other specified order value).
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
public @interface Order {
    /**
     * The order value. Default is {@link Ordered#LOWEST_PRECEDENCE}.
     * @see Ordered#getOrder()
     */
    int value() default Ordered.LOWEST_PRECEDENCE;
}
```

```
关于数据源的切换可以参加专门的博文：http://www.cnblogs.com/digdeep/p/4512368.html
```

3） 切点表达式(pointcut)

上面我们看到，无论是 <aop:config> 风格的配置，还是 @Aspect 风格的配置，切点表达式都是重点。都是我们必须掌握的。

 1>pointcut语法形式(execution)：

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern)throws-pattern?)
```

带有 ? 号的部分是可选的，所以可以简化成： ret-type-pattern name-pattern(param_pattern) 返回类型，方法名称，参数三部分来匹配 。

配置起来其实也很简单： * 表示任意返回类型，任意方法名，任意一个参数类型； .. 连续两个点表示0个或多个包路径，还有0个或多个参数 。就是这么简单。看下例子：

```java
execution(* net.aazj.service..*.get*(..)) ：表示net.aazj.service包或者子包下的以get开头的方法，参数可以是0个或者多个（参数不限）；
execution(* net.aazj.service.AccountService.*(..)): 表示AccountService接口下的任何方法，参数不限；
注意这里，将类名和包路径是一起来处理的，并没有进行区分，因为类名也是包路径的一部分。
参数param- pattern 部分比较复杂： () 表示没有参数，(..)参数不限，(*,String) 第一个参数不限类型，第二参数为String .
```

2>within() 语法:

within()只能指定(限定)包路径(类名也可以看做是包路径)，表示某个包下或者子报下的所有方法：

```java
within(net.aazj.service.*)， within(net.aazj.service..*)，within(net.aazj.service.UserServiceImpl.*)
```

3>this() 与 target():

this是指代理对象，target是指被代理对象(目标对象)。所以 this() 和 target() 分别限定 代理对象的类型和被代理对象的类型：

```java
this(net.aazj.service.UserService): 实现了UserService的代理对象(中的所有方法)； 

target (net.aazj.service.UserService): 被代理对象 实现了UserService(中的所有方法)；
```

4> *args():*

限定方法的参数的类型：

```java
args(net.aazj.pojo.User): 参数为User类型的方法。
5>@target(), @within(), @annotation(), @args():

这些语法形式都是针对注解的 ，比如 带有某个注解的 类 ， 带有某个注解的 方法， 参数的类型 带有某个注解 ：
@within(org.springframework.transaction.annotation.Transactional) 
@target(org.springframework.transaction.annotation.Transactional)
```

两者都是指被代理对象 类 上有 @Transactional 注解的(类的所有方法)，（两者似乎没有区别？？？）

```java
@annotation(org.springframework.transaction.annotation.Transactional)：  方法 带有 @Transactional 注解的所有方法 

@args(org.springframework.transaction.annotation.Transactional)： 参数的类型 带有 @Transactional 注解 的所有方法 
```

6>bean(): 指定某个bean的名称

```java
bean(userService): bean的id为 "userService" 的所有方法;

bean(*Service): bean的id为 "Service"字符串结尾的所有方法;

另外注意上面这些表达式是可以利用 ||, &&, ! 进行自由组合的。比如：execution(public * net.aazj.service..*.getUser(..)) && args(Integer,..)
```

\4. 向注解处理方法传递参数

有时我们在写注解处理方法时，需要访问被拦截的方法的参数。此时我们可以使用 args() 来传递参数，下面看一个例子：

```java
@Aspect
@Component // for auto scan
//@Order(2)
public class LogInterceptor {    
    @Pointcut("execution(public * net.aazj.service..*.getUser(..))")
    public void myMethod(){};
    @Before("myMethod()")
    public void before() {
        System.out.println("method start");
    } 
    @After("myMethod()")
    public void after() {
        System.out.println("method after");
    } 
    @AfterReturning("execution(public * net.aazj.mapper..*.*(..))")
    public void AfterReturning() {
        System.out.println("method AfterReturning");
    } 
    @AfterThrowing("execution(public * net.aazj.mapper..*.*(..))")
//  @Around("execution(public * net.aazj.mapper..*.*(..))")
    public void AfterThrowing() {
        System.out.println("method AfterThrowing");
    } 
    @Around("execution(public * net.aazj.mapper..*.*(..))")
    public Object Around(ProceedingJoinPoint jp) throws Throwable {
        System.out.println("method Around");
        SourceLocation sl = jp.getSourceLocation();
        Object ret = jp.proceed();
        System.out.println(jp.getTarget());
        return ret;
    } 
    @Before("execution(public * net.aazj.service..*.getUser(..)) && args(userId,..)")
    public void before3(int userId) {
        System.out.println("userId-----" + userId);
    }  
    @Before("myMethod()")
    public void before2(JoinPoint jp) {
        Object[] args = jp.getArgs();
        System.out.println("userId11111: " + (Integer)args[0]);
        System.out.println(jp.getTarget());
        System.out.println(jp.getThis());
        System.out.println(jp.getSignature());
        System.out.println("method start");
    }    
}
```

方法：

```java
@Before("execution(public * net.aazj.service..*.getUser(..)) && args(userId,..)")
    public void before3(int userId) {
        System.out.println("userId-----" + userId);
    }
```

它会拦截 net.aazj.service 包下或者子包下的getUser方法，并且该方法的第一个参数必须是int型的， 那么使用切点表达式args(userId,..) 就可以使我们在切面中的处理方法before3中可以访问这个参数。

before2方法也让我们知道也可以通过 JoinPoint 参数来获得被拦截方法的参数数组。 JoinPoint 是每一个切面处理方法都具有的参数， @Around 类型的具有的参数类型为ProceedingJoinPoint。通过 JoinPoint或者 ProceedingJoinPoint 参数可以访问到被拦截对象的一些信息(参见上面的 before2 方法)。

## 92.解释一下什么是 ioc？



## 93.spring 有哪些主要模块？



## 94.spring 常用的注入方式有哪些？



## 95.spring 中的 bean 是线程安全的吗？



## 96.spring 支持几种 bean 的作用域？



## 97.spring 自动装配 bean 有哪些方式？



## 98.spring 事务实现方式有哪些？



## 99.说一下 spring 的事务隔离？



## 100.说一下 spring mvc 运行流程？



## 101.spring mvc 有哪些组件？



## 102.@RequestMapping 的作用是什么？



## 103.@Autowired 的作用是什么？