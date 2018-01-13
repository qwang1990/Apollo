---
title: spring AOP
date: 2018-01-13 21:55:50
categories: 
- 后端 
tags: 
- java
- spring
---
# AOP
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

Spring AOP暂时只支持方法类型join point(建议是spring beans中的方法)。字段interception暂时不支持，虽然它可以在不破坏Spring AOP核心api
的基础上添加。如果你想增强字段，可以考虑使用AspectJ。

Spring AOP的实现和其他大多AOP框架不尽相同。它致力于更紧密的结合AOP实现和Spring IoC，而不是提供最完成的AOP实现。

因此Spring AOP通常用于结合Spring IoC。切面可以使用普通的bean的语法。这是Spring AOP和其他AOP最大的不同。当然也有很多事情使用不适合使用
Spring AOP，比如增强一个细粒度的对象？？（例如domain object），这时使用AspectJ是一个更好的选择。

Spring AOP从来不在提供完备的AOP功能上和AspectJ竞争。我相信每种proxy-based框架如Spring AOP和AspectJ都是有价值的。它们的关系应该是互相协作
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
aop:config>
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





















