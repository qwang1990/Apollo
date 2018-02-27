---
title: spring AOP
date: 2018-01-13 21:55:50
categories: 
- 后端 
tags: 
- java
- spring
---
# Spring AOP
## demo 链接
[demo](https://github.com/qwang1990/spring5)
## 概念
- Aspect(切面): 一个衡切很多class的模块。事物管理就是一个很好的例子。在Spring AOP中，切面由普通的class实现(schema-based approach)
或带注解@Aspect的不同class实现。
- Join point(连接点):程序执行中的点，如方法的调用或异常的抛出，在Spring AOP中专指程序的调用。
- Advice(增强): 切面在特定的切入点执行的动作。类型有:'around','before','after'。很多AOP框架(包括Spring)把advice已interceptor的形式
来实现，管理一个围绕着连接点的interceptors串。
- Pointcut(切入点):符合切入条件的连接点。advice是和pointcut表达式相关联的，在连接点符合条件时执行。
- Introduction(引入): 将方法和字段添加到被处理的类中。Spring允许你将新的接口(和对应的实现)引入到任何被处理的对象中。比如你可以使任何对象
实现IsModified接口来简化缓存。
- Target Object(目标对象):被增强的对象。如果AOP框架使用动态代理实现，它也被称为被代理的对象。
- AOP proxy(AOP 代理): AOP框架创建的对象，用于实现增强。在Spring Framework中，AOP代理有可能是JDK的动态代理或者是CGLIB代理。
- Weaving(植入):链接其他对象或应用创建增强对象的过程。可以在编译期(如:AspectJ compiler)，加载期，或运行期。Spring AOP和其他
纯Java AOP框架都是在运行期进行植入的。

## advice的类型
- Before advice:在join point之前执行增强，它没有中断join point后续执行的能力(除非抛异常)
- After returning advice:在join point正常执行完后执行增强
- After throwing advice:当方法抛出异常的时候执行增强
- After(finally) advice:无论join point如何结束都执行增强
- Around advice:在方法执行的前后来增强。这种类型的增强是能力最大的，它可以在方法执行的前或后加入自定义的行为。它也可以是否继续执行或直接
返回或抛出异常。

## Spring AOP的能力和目标
Spring AOP是用纯java实现的，不需要其他的特别编译，不用控制类加载机制，所以适合用在servlet容器和应用中。

Spring AOP暂时只支持方法类型join point(建议是spring beans中的方法)。字段interception暂时不支持，虽然它可以在不破坏Spring AOP核心api的基础上添加。如果你想增强字段，可以考虑使用AspectJ。

Spring AOP的实现和其他大多AOP框架不尽相同。它致力于更紧密的结合AOP实现和Spring IoC，而不是提供最完成的AOP实现。

因此Spring AOP通常用于结合Spring IoC。切面可以使用普通的bean的语法。这是Spring AOP和其他AOP最大的不同。当然也有很多事情使用不适合使用
Spring AOP，比如增强一个细粒度的对象（例如domain object），这时使用AspectJ是一个更好的选择。

Spring AOP从来不在提供完备的AOP功能上和AspectJ竞争。我相信每种proxy-based框架如Spring AOP和成熟的(type-based)框架如AspectJ都是有价值的。它们的关系应该是互相协作
而不是竞争。Spring为了能使所有的AOP用户无缝集成了Spring AOP，IoC和AspectJ。

## AOP 代理
Spring AOP默认是使用标准的JDK动态代理。它可以代理任意的interface。当Spring AOP需要代理一个类而不是接口时，它也可以使用CGLIB代理。当业务对象
没有实现接口的时候默认使用CGLIB。你也可以强制使用CGLIB(希望不要经常这样)

## 支持@AspectJ注解
@AspectJ用来把常规java类标声明为切面。它是在AspectJ 5 release版本被引入的，Spring使用了同样的注解，使用AspectJ提供的库来解析匹配切点。尽管如此
AOP用的还是纯的Spring AOP，没有依赖AspectJ的编译或植入。

## 开启@AspectJ
Spring可以通过XML或JAVA配置类的方式来支持@AspectJ。但是无论哪种形式，你要首先确保在你项目的类路径里有aspectjweaver.jar这个包

- 使用JAVA配置类的形式
```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```
- 使用XML配置的形式
```xml
<aop:aspectj-autoproxy/>
```

## 声明一个aspect
在你的程序中的任意一个class，只要以@AspectJ为注解就能被Spring自动检测到称为切面。
```java
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
@Component
public class NotVeryUsefulAspect {

}
```
切面和其他的类一样可以有方法和字段。它们也可以包含pointcut，advice，introduction。
> 注意：切面不可再被增强。当class带有@Aspect注解以后，他就不会被auto-proxying了。

## 声明pointcut
回忆一下pointcut决定着对哪个join points感兴趣，因此也控制了增强执行的时机。Spring AOP只支持方法执行的join point，所以你可以认为pointcut
就是匹配方法的。一个pointcut声明包含两部分:1.签名包含名字和参数，2.一个pointcut表达式，用来决定感兴趣的方法。在@AspectJ注解风格的AOP中，
签名是由一个普通的方法定义提供的，pointcut表达式是包含在注解里的。
> 为pointcut的方法返回值必须为void

下面这个例子就是一个pointcut，名字为anyOldTransfer，它会匹配所有名为'transfer'的方法
```java
@Pointcut("execution(* transfer(..))")// pointcut表达式
private void anyOldTransfer() {}// 签名
```
pointcut表达式是常规的AspectJ5切点表达式，详情关注:[AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html)

### 支持的Pointcut选择器
- execution - 用于匹配方法执行的join points，这个是Spring AOP主要使用的选择器
- within - 匹配在某个特定类型内的join points（在SpringAOP中值在某个类型中的方法）
- this - 匹配AOP代理是指定类型的实例的所有join points(在SpringAOP中只有方法)
- target - 匹配目标类型是指定类型的实例的全部join points(在SpringAOP中只有方法)
- args - 按参数类型匹配
- @target - 匹配带有指定注解的类
- @args - 匹配运行时参数有指定注解的类
- @within - 配置带有指定注解的类型
- @annotation - 匹配带有指定注解的方法（因为SpringAOP中join point都是方法）


因为Spring AOP仅针对方法join point，所以上述选择器的定义范围会比AspectJ中的要窄。另外，AspectJ是type-based的语义，所以this和target
指向同一个对象(执行该方法的对象)。Spring AOP是proxy-based体系，所以this和target是不同的，this指的是proxy对象，target指的是目标对象。

Spring AOP提供了一个新的pointcut选择器'bean'。它允许你匹配指定名称的spring bean(使用通配符时可以匹配一系列的bean)
> bean(idOrNameOfBean)

### 组合pointcut表达式
pointcut表达式可以使用&&，|| 和！组合。它们也可以使用名字被引用。
```java
//匹配所有public方法
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}

//匹配在trading下的所有join point
@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}

//匹配trading下的所有public方法
@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```
像上面那样使用简单的名字来标识负责的pointcut表达式是一种很好的习惯。当使用切点的名字来指代表达式时遵循java可见性规则(private只在同一类型可见，
protected在继承中可见，public都可见)。可见性不影响切点的匹配。

### 共享的common切点定义
在一个企业级的应用中，建议定义一个SystemArchitecture切面来声明共享的pointcut
```java
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * join point是web层，定义在com.xyz.someapp.web或其子package下的方法
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * join point是服务层，定义在com.xyz.someapp.service或其子packege下的方法
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```
定义在上述切面中的切点可以在任何地方被访问到。比如想在服务层做一个事物管理，可以这么写:
```xml
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```
&lt;aop:config&gt; 和 &lt;aop:advisor&gt;元素，transaction元素会在后面讲到。

### 举例
Spring AOP用户最常用的就是execution切点选择器。格式如下:
> execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
              throws-pattern?)
              
上面例子中，除了ret-type-pattern,name-pattern,param-pattern外其余都是可选的。你经会用*来表示return type，它可以匹配所以返回类型。
name-pattern用来匹配方法名。param-pattern稍微有点复杂:()匹配无参数,(..)匹配任意数量的参数(0或很多),(\*)表示有一个任意类型的参数,
(\*,String)匹配两个参数，第一个是任意类型，第二个是String。

下面看一些常见的例子:

- 所有public方法
> execution(public * *(..))

- 所有已set开头的方法
> execution(* set*(..))

- 所有AccountService接口里的方法
> execution(* com.xyz.service.AccountService.*(..))

- 所有定义在service包下的方法
> execution(* com.xyz.service.\*.\*(..))

- 所有service包和其子包的方法
> execution(* com.xyz.service..\*.\*(..))

- 在service包下的所有join point(在Spring AOP中只有方法)
> within(com.xyz.service.*)

- 在service包及其子包下的所有join point(在Spring AOP中只有方法)
> within(com.xyz.service..*) 

- 所有代理实现AccountService接口的join point
> this(com.xyz.service.AccountService)

- 所有目标对象实现AccountService接口的
> target(com.xyz.service.AccountService)

- 匹配一个参数，并且运行时参数实现Serializable
> args(java.io.Serializable)
注意：改切点和execution(* *(java.io.Serializable))是不同的，args匹配运行时传入的是Serializable的，execution匹配的是声明为Serializable的

- 目标对象有@Transaction注解
> @target(org.springframework.transaction.annotation.Transactional)

- 对象声明了@Transaction注解
> @within(org.springframework.transaction.annotation.Transactional)

- 方法有@Transaction注解
> @annotation(org.springframework.transaction.annotation.Transactional)

- 方法带有一个参数，并且运行时传入的参数带有@Classified注解
> @args(com.xyz.security.Classified)

- Spring bean名为tradeService的所有方法
> bean(tradeService)

- 所有匹配通配符表达式的bean的方法
> bean(*Service)

### 写一个好的切点
在编译期AspectJ会处理pointcuts来优化它的匹配效率。测试代码然后确认那些joint point(静态或动态)匹配给定的pointcuts是一个耗时的过程。动态
匹配意味着匹配不能再静态分析的时候完全确定。当第一次遇到一个pointcut时，AspectJ会把它写入一个用于匹配的优化的表里。这意味着什么？简单来说
pointcuts被重写在DNF(disjunctive normal form)中，pointcuts中的部分被分别存储，然后那些检测起来容易的部分会先检测。这就意味着你不必操心
每一种pointcut选择器的性能，你只需要一个个的声明它们就行。
尽管如此，AspectJ也只能做到这些了，为了更加优化匹配效率你应该思考这些选择器的目的是什么，然后尽可能的缩小选择的范围。目前选择期可以分为三大类:
kinded,scoping,context。

- kinded:选择某一个特定类型的join point。比如:execution,get,set,call,handler。
- scoping:选择一组join point。比如:within，wihtincode
- contextual:依赖上下文来选择的。比如:this,target,@annotation

一个好的pointcut最好能至少包含前两种类型(kinded,scoping),当然在有些情况下contextual选择器还是需需要用的。仅仅提供kinded或contextual选择器
可能会因为额外的处理和分析而导致影响植入效率(时间和内存使用上)。scoping选择器可以很快的匹配，并且用了它可以很快的去除很多不用分析的join point，
这就是一个好的joint point最好能有一个它。


## 声明advice
advice和切点表达式是相关的。它会在匹配切点的方法执行的前，后，或上下来执行。pointcut表达式可以被简单的以名字来引用或直接写到括号里。

- Before advice
Before advice就是使用@Before声明在切面里的方法
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }
    
    //使用切点表达式
    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
           // ...
    }

}
```

- After returning advice
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

    //当你需要方法的返回值的时候
    //这时参数值必须和returning属性值一致。
     @AfterReturning(
            pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
            returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
}
```

- After throwing
在匹配方法抛出异常是执行
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }
    
    //指定异常
     @AfterThrowing(
            pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
            throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```

- After (finally) advice
处理正常或异常情况，典型的用法是释放资源
```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```
- Around advice
这时最后一个类型的增强。他可以在方法的前后执行，并且能够决何时，如何执行方法。Around advice常常用于需要在方法执行前后共享状态。
Around 增强的注解为@Around。它的第一个参数一定要是ProceedingJoinPoint类型的。在增强的方法体里，调用ProceedingJoinPoint的proceed()
方法来使剩下的方法执行。proceed方法还可以参入一个Object[]的参数，这个是方法执行的参数。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }

}
```

- Advice parameters
Spring提供了全部类型的advice，这意味着你可以在advice签名中声明你需要的参数(向我们上面的returning和throwing一样)而不是一直用Object[]。
下面我们会看到如何在advice中使用参数和上下文。首先我们看一下如何写一个通用的advice来发现当前被增强的方法。

- 访问当前join point
每一个增强的第一个参数都可以是org.aspectj.lang.JoinPoint(注意around advice的第一个参数是ProceedingJoinPoint，它是JoinPoint的子类。)

- 向advice传递参数
我们已经看到如何绑定返回值和异常值到advice。要想绑定参数值，你可以用args。如果参数名代替类型出现在args表达式中，当advice触发时对应参数的值就会传入
到该增强中。看下面一个例子。假设你想增强dao的执行，并且希望Account对象为第一个参数，你还想再advice体中访问这个参数。你可以写成下面形式:
```java
@Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```
arg(account,..)有两个意图:第一，它限制匹配至少一个参数，并且参数是Account实例；第二，它使advice可以通过account参数来访问实际的Account参数。
另一种写法就是声明一个切点，提供Account对象。此时advice只需要引用这个切点名就行了。
```java
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```
代理对象(this),目标对象(target),注解(@within,@target,@annotation,@args)也可以用类似的方式绑定。下面一个例子展示和如何匹配@Auditable注解
和怎么抽取数据。

首先定义@Auditable注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```
接着定义advice(增强)来匹配@Auditable方法:
```java
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

- Advice泛型参数
Spring AOP可以处理泛型类和方法。假设你有一个泛型类如下:
```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```
使用advice参数，你可以限制拦截的方法的参数
```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```
上述方法不能用在泛型集合上。比如下面这种写法就是<font color=#dc143c size=3>错的！</font>
```java
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```
为了使上面工作，我们必须检查集合中的每一个元素，这个是不合理的就像在泛型中出现null一样。为了达到类似的效果，你可以使用参数Collection<?>，然后
手动的一个个的检查参数类型。

- 决定参数名字
advice触发时的参数依赖于pointcut表达式中的参数名和声明在(advice和pointcut)方法签名中参数名的匹配。但是参数名在java反射中是无法获取的，所以Spring AOP用了下面机制来确定方法名。
    1. 用户显示声明参数值，advice和pointcut注解都有一个可选的"argNames"属性，这些参数运行时可用。比如
    ```java
    @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
            argNames="bean,auditable")
    public void audit(Object bean, Auditable auditable) {
        AuditCode code = auditable.value();
        // ... use code and bean
    }
    ```
    如果第一个参数是JoinPoint, ProceedingJoinPoint, 或 JoinPoint.StaticPart类型，你可以在argNames属性中省略改名字。比如如果上面的例子接收一个
    join point对象。
    ```java
    @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
            argNames="bean,auditable")
    public void audit(JoinPoint jp, Object bean, Auditable auditable) {
        AuditCode code = auditable.value();
        // ... use code, bean, and jp
    }
    ```
    这种方式对那些不需要其他join point上下文的增强特别方便，就不必写argNames属性了。
    
    2. 使用'argNames'属性有点笨拙，所以如果'argNames'属性没有声明，Spring AOP会看debug信息，从本地变量表中决定参数名称。这个信息只要class编译时
    加参数'-g:vars'就可以了。加这个参数会有如下3个影响:(1)你的代码会更容易理解。(2)class文件会稍微大点。(3)去除无用本地变量的优化将不会开启。总而言之，
    这个对你没啥影响。
    > 如果@AspectJ被AspectJ编译器(ajc)编译，这不需要debug information。
    
    3. 如果代码编译没有debug信息，Spring AOP会尝试着推断(比如，如果切点表达式只有一个参数，增强方法也只有一个参数，显然就是它)。如果参数绑定是模棱两可的，
    者会抛出AmbiguousBindingException异常。
    
    4. 如果上述机制全都失败，者会抛出IllegalArgumentException异常。
   
 - Advice顺序
  当一个join point有多个advice时会有什么情况？SpringAOP和AspectJ使用的优先级顺序实现同的。在进入时高优先级的先执行，在离开时高优先级的后执行。
  当两个advice定义在不同的aspects中并且增强同一个join point，除非你标明顺序，否则他们的执行顺序是不定的。你可以控制执行顺序。使用Spring的org.springframework.core.Ordered接口或Order注解即可完成。两个aspects，Ordered.getValue()或注解值返回值小的优先级高。
  当两个advice定义在同一个aspects中并且增强同一个join point，顺序是无法确定的(因为java反射无法确认声明顺序)。所以在同一个aspect类中，对同一个join point的增强最好放在同一个方法中，或者把它们重构到不同的aspect方法中。
 
### 引入
  引入(在AspectJ中成为inter-type声明)赋予aspect给被加强对象指定一个接口并提供接口实现的能力。
 
  引入是由@DeclareParents注解定义的。这个注解用来声明匹配的类型有一个新的parent。例如：现在有个接口UsageTracked，实现为DefaultUsageTracked，下面的的aspect声明全部service下的类也实现了UsageTracked接口。
 
```java
 @Aspect
 public class UsageTracking {
 
     @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
     public static UsageTracked mixin;
 
     @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
     public void recordUsage(UsageTracked usageTracked) {
         usageTracked.incrementUseCount();
     }
 
 }
```
 上面的例子中，所有匹配的类型都会实现UsageTracked接口。这时service bean可以像UsageTracked接口一样使用。如果你想访问它可以写成下面这样:
 > UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
 
### aspect(切面)的实例化模型
 默认情况下在应用上下文中每个切面只有一个实例。AspectJ称之为单类模型。aspect可以声明别的声明周期:Spring支持prethis和pretarget。(percflow,percflowbelow,pertypewithin 暂时还不支持)  
 
 下面看一个prethis的例子。
 
```java

   @Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
   public class MyAspect {
   
       private int someState;
   
       @Before(com.xyz.myapp.SystemArchitecture.businessService())
       public void recordServiceUsage() {
           // ...
       }
   
   }
```
 prethis会为每一个执行business服务的对象(AOP对象)创建一个aspect实例。 该实例会在服务对象调用方法是创建。当服务对象离开作用域时，切面对象也会一起离开作用域。在切面对象创建之前，增强不会被执行。
 
 pretarget类似，只是它为每一个目标对象创建一个切面实例。
 
### 举例
 很多业务服务可能会因为高并发失败。并且这些业务重试可以很快成功。对于这种适合重试的业务(幂等操作不需要反馈给用户)，我们选择透明的重试而不让用户看到PessimisticLocking FailureException异常。
 
 因为我们需要重试操作，所以我们需要使用around增强。
```java
 @Aspect
 public class ConcurrentOperationExecutor implements Ordered {
 
     private static final int DEFAULT_MAX_RETRIES = 2;
 
     private int maxRetries = DEFAULT_MAX_RETRIES;
     private int order = 1;
 
     public void setMaxRetries(int maxRetries) {
         this.maxRetries = maxRetries;
     }
 
     public int getOrder() {
         return this.order;
     }
 
     public void setOrder(int order) {
         this.order = order;
     }
 
     @Around("com.xyz.myapp.SystemArchitecture.businessService()")
     public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
         int numAttempts = 0;
         PessimisticLockingFailureException lockFailureException;
         do {
             numAttempts++;
             try {
                 return pjp.proceed();
             }
             catch(PessimisticLockingFailureException ex) {
                 lockFailureException = ex;
             }
         } while(numAttempts <= this.maxRetries);
         throw lockFailureException;
     }
 
 }
```
 注意这个切面实现了Ordered接口，所以为我们可以让他的优先级高于transaction增强(我们希望每次都是一个新的事物)。maxRetries和order都可以被配置。
 对应的Spring配置:
```xml
   <aop:aspectj-autoproxy/>
   
   <bean id="concurrentOperationExecutor" class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
       <property name="maxRetries" value="3"/>
       <property name="order" value="100"/>
   </bean>
```
 为了使切面只重试幂等操作，我们需要定义一个Idempotent注解:
```java
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Idempotent {
       // marker annotation
   }
```
 然后使用annotation修改上面的切面，然他只匹配@Idempotent操作。
```java
   @Around("com.xyz.myapp.SystemArchitecture.businessService() && " +
           "@annotation(com.xyz.myapp.service.Idempotent)")
   public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
       ...
   }
```
 
## 基于XML的AOP
 Spring提供了"aop"命名空间。它支持和@AspectJ相同的切点表达式和增强类型。为了使用aop命名空间，你需要在XML配置中加入spring-aop schema:
```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="
           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd"> <!-- bean definitions here -->
   </beans>
```
 在Spring配置中，所有的切面相关的元素信息都要放在&lt;aop:config&gt;中(在上下文中可以声明多个&lt;aop:config&gt;)。一个&lt;aop:config&gt;中可以包含pointcut，advisor，aspect元素(注意这些必须按照上面的顺序声明)
 
 >&lt;aop:config&gt;样式的配置严重依赖Spring的auto-proxy机制。如果你使用诸如BeanNameAutoProxyCreator的方式显示的auto-proxy就会出现问题。所以建议是要么全部使用xml或注解。
 
### 声明一个切面
 使用配置的aspect就是一个在应用上下文上定义的一个普通java对象。他的状态和行为是对象的域和方法决定的，pointcut和advice信息是xml配置确定的。
 
 一个切面使用&lt;aop:config&gt;定义的，它背后的bean是靠ref属性引用的。
```xml
 <aop:config>
     <aop:aspect id="myAspect" ref="aBean">
         ...
     </aop:aspect>
 </aop:config>
 
 <bean id="aBean" class="...">
     ...
 </bean>
```
### 声明pointcut
 可以在&lt;aop:config&gt;中声明一个有名字的pointcut，这样它就可以在多个切面中共享。
 例如下面这个切点用于所有service层。
```xml
 <aop:config>
 
     <aop:pointcut id="businessService"
         expression="execution(* com.xyz.myapp.service.*.*(..))"/>
 
 </aop:config>
```
 xml中的切点表达式和@AspectJ中的一样。如果你使用xml形式的配置，你可以引用在@Aspects中定义的切点所以当你有一个[SystemArchitecture](https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/core.html#aop-common-pointcuts)的切面时,上面的切点定义就可以写成下面这样:
```xml
 <aop:config>
 
     <aop:pointcut id="businessService"
         expression="com.xyz.myapp.SystemArchitecture.businessService()"/>
 
 </aop:config>
```
 
 在切面中定义切点和在顶层定义一样:
```xml
 <aop:config>
 
     <aop:aspect id="myAspect" ref="aBean">
 
         <aop:pointcut id="businessService"
             expression="execution(* com.xyz.myapp.service.*.*(..))"/>
 
         ...
 
     </aop:aspect>
 
 </aop:config>
```
 和@AspectJ一样，基于xml配置的切点也可以获取join point上线问。比如下面这个切点就获取join point的'this'对象，然后传递给advice:
```xml
 <aop:config>
     <aop:aspect id="myAspect" ref="aBean">
         <aop:pointcut id="businessService"
             expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>
         <aop:before pointcut-ref="businessService" method="monitor"/>
         ...
     </aop:aspect>
 </aop:config>
```
 此时advice必须可以接收这个参数：
```java
 public void monitor(Object service) {
     ...
 }
```
 显然上面在xml中使用’&&‘是很笨拙的，所以可以用'and','or'和'not'来代替'&&','||','!'。所以上面就可以写成:
```xml
 <aop:config>
 
     <aop:aspect id="myAspect" ref="aBean">
 
         <aop:pointcut id="businessService"
             expression="execution(* com.xyz.myapp.service.*.*(..)) **and** this(service)"/>
 
         <aop:before pointcut-ref="businessService" method="monitor"/>
 
         ...
     </aop:aspect>
 </aop:config>
```
 >注意这种方式定义的切点是通过XML id来引用，所以不能使用名字来组成合成切点。 因此基于xml配置的有名切点比@AspectJ方式的多了一些限制。
 
### 声明增强(advice)
xml配置支持@AspectJ风格的全部advice，而且他们的语义也基本相同。

- before advice
  在匹配方法之前执行。它用&lt;aop:before&gt;元素始声明在&lt;aop:aspect&gt;中,这里dataAccessOperation是一个切点的id。也可以直接把切点内嵌在增强中。
```xml
    <aop:aspect id="beforeExample" ref="aBean">
        <aop:before
            pointcut-ref="dataAccessOperation"
            method="doAccessCheck"/>
        ...
        <!-- 切点内嵌 -->
        <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>
    </aop:aspect>
```
  method属性中定义的doAccessCheck方法提供了增强的实现。该方法必须定义在aspect bean中。

- after returning advice
  after returning advice在匹配方法正常完成时执行。
```xml
    <aop:aspect id="afterReturningExample" ref="aBean">
        <aop:after-returning
            pointcut-ref="dataAccessOperation"
            method="doAccessCheck"/>
        ...

        <!-- 如果想要获得返回值 -->
        <aop:after-returning
            pointcut-ref="dataAccessOperation"
            returning="retVal"
            method="doAccessCheck"/>
    </aop:aspect>
```
  在doAccessCheck方法中必须声明一个名为retVal的参数。这个参数的匹配规则和@AfterReturning是一样的。
```java
    public void doAccessCheck(Object retVal) {...
```
- After throwing advice
 当匹配方法抛出异常是执行
```xml
    <aop:aspect id="afterThrowingExample" ref="aBean">
    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>
    ...

    <!-- 获取抛出异常 -->
    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>
    ...
    </aop:aspect>
```
 同上，增强方法也需要有dataAccessEx参数。
```java
    public void doRecoveryActions(DataAccessException dataAccessEx) {...
```

- Around advice
 around advice使用aop:around元素声明。第一个参数是ProceedingJoinPoint类型的。在advice方法中调用ProceedingJoinPoint的proceed()方法来触发后续的方法执行。proceed方法可以接收Object[]参数。
 ```xml
    <aop:aspect id="aroundExample" ref="aBean">
        <aop:around
            pointcut-ref="businessService"
            method="doBasicProfiling"/>
        ...
    </aop:aspect>
 ```
doBasicProfiling方法的实现和@AspectJ例子中方法实现一样。
```java
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }
```

- advice 参数
xml模式的注解对注解参数的支持和@AspectJ一样。它使用arg-names属性来代替argNames。
```xml
    <aop:before
    pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
    method="audit"
    arg-names="auditable"/>
    <!-- arg-names支持以逗号分隔的方式声明一些列参数 -->
```

 下面的例子展示了一个接收多个不同类型的参数的around增强。
 ```java
    package x.y.service;
    public interface FooService {
        Foo getFoo(String fooName, int age);
    }
    public class DefaultFooService implements FooService {
        public Foo getFoo(String name, int age) {
            return new Foo(name, age);
        }
    }
 ```
下一步定义切面，注意增强方法profile(..)接收一系列不同类型的参数，但是第一个参数要是join point。
```java
    package x.y;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.springframework.util.StopWatch;

    public class SimpleProfiler {

        public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
            StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
            try {
                clock.start(call.toShortString());
                return call.proceed();
            } finally {
                clock.stop();
                System.out.println(clock.prettyPrint());
            }
        }
    }
```

最后，这里是对应的xml配置:

```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
      <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
      <bean id="fooService" class="x.y.service.DefaultFooService"/>
      <!-- this is the actual advice itself -->
      <bean id="profiler" class="x.y.SimpleProfiler"/>
      <aop:config>
          <aop:aspect ref="profiler">
              <aop:pointcut id="theExecutionOfSomeFooServiceMethod"
                  expression="execution(* x.y.service.FooService.getFoo(String,int))
                  and args(name, age)"/>

              <aop:around pointcut-ref="theExecutionOfSomeFooServiceMethod"
                  method="profile"/>
          </aop:aspect>
      </aop:config>
  </beans>
```

如果你按照上述方法配置，这下面代码会在标准输出中输出:
```java
    import org.springframework.beans.factory.BeanFactory;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import x.y.service.FooService;
    public final class Boot {
        public static void main(final String[] args) throws Exception {
            BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
            FooService foo = (FooService) ctx.getBean("fooService");
            foo.getFoo("Pengo", 12);
        }
    }
    //  output:
    //      StopWatch 'Profiling for 'Pengo' and '12'': running time (millis) = 0
   //       -----------------------------------------
   //       ms     %     Task name
   //       -----------------------------------------
   //       00000  ?  execution(getFoo) 
```

- Advice 顺序
参见@AspectJ

### Introductions
introduction赋予aspect给被增强对象声明实现接口的能力，并未其提供实现。它有aop:aspect中的aop:declare元素实现，它用来给匹配类型声明父类。比如，给定一个接口UsageTracked和它的实现DefaultUsageTracked，下面切面声明了全部的service接口都实现了UsageTraced接口。
```xml
    <aop:aspect id="usageTrackerAspect" ref="usageTracking">

        <aop:declare-parents
            types-matching="com.xzy.myapp.service.*+"
            implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
            default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

        <aop:before
            pointcut="com.xyz.myapp.SystemArchitecture.businessService()
                and this(usageTracked)"
                method="recordUsage"/>

    </aop:aspect>
```
和@AspectJ类似

- Aspect instantiation models
xml模式只支持singleton模型。

- Advisors
advisors这个概念是Spring AOP引入的，在AspectJ中并没有对等的概念。一个advisor像一个自包含的只有个一增强的aspect。这个增强自身就是一个bean，它必须实现[advice type in spring](https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/core.html#aop-api-advice-types)中的任意一个接口。

Spring通过&lt;aop:advisor&gt;元素来支持advisor。它经常用于transaction增强:
```xml
    <aop:config>
    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>
    <aop:advisor
        pointcut-ref="businessService"
        advice-ref="tx-advice"/>
    </aop:config>
    <tx:advice id="tx-advice">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
```
你也可以用pointcut属性来替换上面的pointcut-ref属性。advisor也可以通过order属性来控制执行优先级。

## 选择哪种风格的AOP更适合
### Spring AOP or Full AspectJ
能满足需求，越简单越好！Spring AOP比完整的AspectJ要简单的多，它不需要在你的开发和构建过程中引入AspectJ的编译/植入。如果你只需要增强你的Spring bean的功能，Spring AOP是一个好的选择。如果你不仅仅需要增强Spring容器中的对象(domain对象)，或者当你希望出方法之外的其他类型的join point时，你就需要使用AspectJ。

当使用AspectJ时，你可以选择AspectJ语法(也就是code style)或@AspectJ风格。如果你用的java版本小于java 5，那你只能用code style了。如果切面在你系统的比例非常大，并且你可以在eclipse里使用[AspectJ Development Tools (AJDT)](https://www.eclipse.org/ajdt/)插件，这时悬着AspectJ语法是对的选择。如果你没有使用eclipse或切面在你的系统中只是很小的一部分，那么这时使用@AspectJ风格，然后在构建脚本中添加切面的植入过程就可以了。

### @AspectJ or XML
当你选择使用SpringAOP时，选择@AspectJ还是XML风格就有很多事情需要考虑了。
当你用AOP来配置企业级服务的时候，XML是一个好的选择。你可以很好的控制切面的可见性。但是XML风格有两个缺点。第一，切面的实现没有封闭在一个地方，这个违背了DRY原则。第二，XML风格对比与@AspectJ稍微有更多的限制，比如XML只支持singleton的切面，它也不能使用切点的名字来组合切点（因为切点是以id声明的），例如使用@AspectJ你可以这么写:
```java
@Pointcut(execution(* get*()))
public void propertyAccess() {}

@Pointcut(execution(org.xyz.Account+ *(..))
public void operationReturningAnAccount() {}

@Pointcut(propertyAccess() && operationReturningAnAccount())
public void accountPropertyAccess() {}
```
使用XML，你只能声明前两个：
```xml
<aop:pointcut id="propertyAccess"
        expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
        expression="execution(org.xyz.Account+ *(..))"/>

```
@AspectJ还有个优点是它可以被Spring AOP和AspectJ识别。

## 代理机制
Spring AOP使用JDK的动态代理或CGLIB来给目标对象创建代理(无论何时JDK的动态代理是首选的)
只要目标对象实现了一个接口，就会使用JDK的动态代理。目标对象对接口的所有实现都会被代理。如果目标对象没有实现任何接口就会使用CGLIB代理。

如果你想强制使用CGLIB代理(比如，你想代理目标对象的全部方法而不仅仅是接口中的)，但是你要注意下列问题:
- 方法无法增强，因为他们不能重载
- Spring 3.2不需要在classpath中加入CGLIB了，因为CGLIB已经被打包到spring-core jar里了。这就意味着基于CGLIB的代理和JDk的动态代理在使用上并无不同了。
- Spring 4.0你的被代理对象的构造函数不会再被调用两次了，因为CGLIB代理将通过Objenesis来创建对象。但是如果你的JVM不允许绕开构造器，你就会看到两次构造函数的调用和对应的日志。
可以通过设置&lt;aop:config&gt;元素的proxy-target-class标签来强制使用CGLIB代理。
```xml
    <aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
    </aop:config>
```
当使用@AspectJ的autoproxy支持时，如果想强制使用CGLIB代理，者需要设置&lt;aop:aspectj-autoproxy&gt;元素的proxy-target-class标签：
```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

### 理解AOP代理
Spring AOP是基于代理的。理解这点非常重要。看下面这个场景：
```java
    public class SimplePojo implements Pojo {
        public void foo() {
            // this next method invocation is a direct call on the 'this' reference
            this.bar();
        }
        public void bar() {
            // some logic...
        }
    }
```
如果你在一个对象引用上调用方法，这个方法调用会直接作用到方法上，如下:
![simple](https://user-images.githubusercontent.com/13915081/35133097-57301e4c-fd09-11e7-8dc1-fac88840fb33.png)
```java
    public class Main {

        public static void main(String[] args) {

            Pojo pojo = new SimplePojo();

            // this is a direct method call on the 'pojo' reference
            pojo.foo();
        }
    }
```
但是当引用是代理的时候，事情就会有所不同。

![aop-proxy-call](https://user-images.githubusercontent.com/13915081/35153184-0a3d76f2-fceb-11e7-8edc-eaa2179a7946.png)

```java
    public class Main {

        public static void main(String[] args) {

            ProxyFactory factory = new ProxyFactory(new SimplePojo());
            factory.addInterface(Pojo.class);
            factory.addAdvice(new RetryAdvice());

            Pojo pojo = (Pojo) factory.getProxy();

            // this is a method call on the proxy!
            pojo.foo();
        }
    }
```
理解这里的关键是main方法中对foo()的调用是通过proxy的，它意味着代理能够调用所有和该方法关联的增强。但是一旦方法最终到达了目标对象，在本例中就是SimplePojo，任何方法调用他都只会调用自身的，比如this.bar()或this.foo()，会使用this引用而不是proxy。这个非常重要，它意味着自身引用就无法执行增强。

我们怎么解决它呢？最好的解决办法是让你的代码中不包含自引用。这可能会增加一点你的工作量，但是这种方法是最好的，侵入性最小的。
```java
    public class SimplePojo implements Pojo {

        public void foo() {
            // this works, but... gah!
            ((Pojo) AopContext.currentProxy()).bar();
        }

        public void bar() {
            // some logic...
        }
    }
```

它使你的代码和Spring AOP耦合了，而且让class对自己正在使用AOP有感知。并且它还需要创建代理的时候做一些特别的配置:
```java
    public class Main {

        public static void main(String[] args) {

            ProxyFactory factory = new ProxyFactory(new SimplePojo());
            factory.adddInterface(Pojo.class);
            factory.addAdvice(new RetryAdvice());
            //这里是新加入的配置。。。
            factory.setExposeProxy(true);

            Pojo pojo = (Pojo) factory.getProxy();

            // this is a method call on the proxy!
            pojo.foo();
        }
    }
```
最后，请记住在AspectJ中不存在这种自引用的问题，因为AspectJ不是基于proxy的AOP框架。

## 以编码方式创建@AspectJ代理
除了使用&lt;aop:config&gt;或&lt;aop:aspectj-autoproxy&gt;配置来声明切面以外，也可以用编码的形式为目标对象创建代理。下面我们关注如何使用@AspectJ切面自动的创建代理。

org.springframework.aop.aspectj.annotation.AspectJProxyFactory可以用来为那些被一个或多个@AspectJ切面增强的目标对象创建代理。这个类的用法十分简单。
```java
    // create a factory that can generate a proxy for the given target object
    AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

    // add an aspect, the class must be an @AspectJ aspect
    // you can call this as many times as you need with different aspects
    factory.addAspect(SecurityManager.class);

    // you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
    factory.addAspect(usageTracker);

    // now get the proxy object...
    MyInterfaceType proxy = factory.getProxy();
```
























































