---
title: Spring Boot系列---Spring Boot的功能一
date: 2018-04-07 22:21:56
categories: 
- 后端 
tags: 
- java 
- SpringBoot
---
## Spring Boot的功能
### SpringApplication
SpringApplication class提供了一个方便启动Spring应用的方式，大多数情况下你只需要在main方法中运行SpringApplication.run静态方法就行：
```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```
应用启动后，你应该能看到类似输出:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.0.0.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下是Info级别的日志

#### 自定义Banner
可以在你的classpath中添加banner.txt文件或设置spring.banner.location属性来这只文件名。如果文件的编码不是UTF-8，你可以使用spring.banner.charset来修改该编码。除了text文件之外，你还可以在你的classpath中添加banner.gif,banner.jps,banner.png文件，或这只spring.banner.image.location属性。图片会被转化成ASCII表达，并且在text banner上面打印。

在banner.txt文件中，你可以使用如下占位符:
< img width="1153" alt="2018-04-07 11 41 04" src="https://user-images.githubusercontent.com/13915081/38450913-a2d915e2-3a58-11e8-803e-8f39991c6e72.png">

打印banner的功能是名为springBootBanner的单类bean提供的。

#### 自定义SpringApplication
如果默认的SpringApplication不满足你的需求，你可以创建一个本地实例，然后自定义它。比如想要关闭banner，你可以按照如下这么写:
```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

#### 流式构建API
如果你需要构建一个有层级的ApplicationContext或者你想要用流式构建API，你可以使用SpringApplicationBuilder。
SpringApplicationBuilder可以使用parent或child方法调用方法，用法如下:
```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```
当创建ApplicationContext层级时有很多的限制。比如，web组件必须包含在child context中，parent和child上下文必须使用同一个Environment。具体内容参见[SpringApplicationBuilder](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/api/org/springframework/boot/builder/SpringApplicationBuilder.html)。

#### Application event 和 listener
除了Spring frame常用的事件，如ContextRefreshedEvent等，SpringApplication还有很多其他的应用事件。
>有些事件实际上是在ApplicationContext创建之触发的，所以你不能使用@Bean为这些事件注册监听，你可以用SpringApplication.addListeners(…​) 或 SpringApplicationBuilder.listeners(…​) 。
如果这些listener自动注册，你可以添加META-INF/spring.factories文件，然后使用org.springframework.context.ApplicationListener这个key来关联你的listener。
如下所示:
org.springframework.context.ApplicationListener=com.example.project.MyListener

应用事件以如下顺序发起:

- ApplicationStartingEvent在所有处理之前发起，除了注册listeners和初始化。
- ApplicationEnvironmentPreparedEvent在确定context使用的environment时发起，这时context还未创建。
- ApplicationPreparedEvent在refresh开始之前，但是在bean定义被加载之后
- ApplicationStartedEvent在context被刷新之后，但是在所有应用或command-line runners被调用之前。
- ApplicationReadyEvent在任意应用或command-line runner被调用之后。它表明系统已经做好服务的准备了
- ApplicationReadyEvent在系统启动异常的时候发起

>一般用户不会使用application事件，但是明白他们存在是很重要的。Spring boot在内部使用它们做很多的事情。

应用事件使用的是Spring框架的时间发布机制。这种机制保证发布到child context的监听器的事件同样会发布到他的祖先上下文中。因此，如果你的应用使用的是层级式的SpringApplication实例，一个lister肯能会收到同一个应用事件的多个实例。

为了能让你的listener区分出事件的上下文，listener需要获得它的注入context，然后对比该上下文和事件上下文。上下文可以实现ApplicationContextAware来注入或者如果listener是bean的话，可以使用@Autowire。

#### Web环境
SpringApplication尝试创建出最合适的ApplicationContext。默认情况下，AnnotationConfigApplicationContext 或 AnnotationConfigServletWebServerApplicationContext将会被使用，取决于你是否在开发web应用。

判断是否是web环境的算法十分简单(它基于指定的几个类是否存在)。如果你需要重写这种默认算法，你可以使用setWebEnvironment(boolean webEnvironment)。

你也可以使用setApplicationContextClass(…​)来完全的控制ApplicationContext。
>当使用JUnit时，通常setWebEnvironment(false)

#### 读取Application参数
如果你需要读取传递到SpringApplication.run(...)中的参数，你可以注入org.springframework.boot.ApplicationArguments bean。ApplicationArguments接口提供以String[]形式来访问可选和必填参数的方法，如下所示:
```java
import org.springframework.boot.*
import org.springframework.beans.factory.annotation.*
import org.springframework.stereotype.*

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```

未完待续。。。。。。！！！