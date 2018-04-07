---
title: Spring Boot系列---开发工具
date: 2018-04-06 22:20:21
categories: 
- 后端 
tags: 
- java 
- SpringBoot
---
## 开发工具
spring boot提供了一系列的工具。可以用如下方式来引入:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
>当执行一个full packaged application时开发中工具会自动禁用，如果你使用java -jar来启动程序或使用特定的classloader来启动它，它就会被认为是生产环境应用。

### property defaults
spring boot中很多的类库都有缓存功能来提高性能。比如，template engines可以缓存编译的模板来避免多次解析模板文件。又如Spring MVC也可以在返回的静态资源的http头中添加caching header。

尽管caching在生产中特别有帮助，但是在开发中会起反作用，它会让你看不到你对程序的修改。所以spring-boot-devtools默认禁止caching选项。

cache选项通常可以通过application.properties来配置。比如，thymeleaf提供了spring.thymeleaf.cache属性。除此之外，你还可以用spring-boot-devtools来自动的智能的管理这些配置。
> devtools的完整属性清单可以[点击这里](https://github.com/spring-projects/spring-boot/blob/v2.0.0.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)

### 自动restart
应用可以使用spring-boot-devtools在classpath中的文件有变化的时候自动重启。这个功能在使用IDE时特别好用。默认情况下，classpath中任何的变化都会导致重启，但是有些资源变化的时候可能不需要重启，比如view template。可以使用[排除监控资源](#static)来实现。

>因为devtools监控classpath资源，所以唯一触发重启的方式就是更新classpath。在IDEA中，可以使用Build -> Make Project

>devTools在重启过程中依赖application的上下文shutdown钩子。所以你如果禁用了shutdown hook，它将无法正常工作(SpringApplication.setRegisterShutdownHook(false))

>在监控classpath时，devtools会自动忽视名字为spring-boot, spring-boot-devtools, spring-boot-autoconfigure, spring-boot-actuator, spring-boot-starter的项目

>devtools需要定制applicationContext使用的ResourceLoader。所以如果你的应用已经提供了一个，它将被包装。

>restart vs reload
>spring boot的重启是依靠两个classloader实现的。那些没有变化的class会放在一个base classloader中。那些变化的class会被装载进restart classloader。当应用重启以后，restart classloader会被丢弃，然后生成一个新的。这种实现意味着restart会比"cold starts"快很多，因为base classloader始终可用。

>如果你发现restart不够快或你遇到了classloading的问题，你可以使用reloading技术比如[JRebel](https://zeroturnaround.com/software/jrebel/)，它基于重写加载过的class。

#### 变更日志
默认情况下，每次系统重启都会有环境评估日志打印。也可以用如下方法禁用log。
>spring.devtools.restart.log-condition-evaluation-delta=false

#### <span id="static">排除监控资源</span>
有些静态资源在变更时不希望重启，这时可以用如下方法:
>spring.devtools.restart.exclude=static/**,public/**

#### 监控别的路径
有时不仅仅希望监控classpath，可能还希望其他的路径，可以使用spring.devtools.restart.additional-paths属性。你可以把spring.devtools.restart.exclude=用于spring.devtools.restart.additional-paths之前，来标明新增路径中哪些是不想引入的。

#### 禁用restart
如果你不想使用restart功能，可以用spring.devtools.restart.enabled属性禁用它。通常你可以在application.properties中设置该属性(虽然这是restart classloader依然启动，但是它不监控任何别的变化了)。

如果你想要完全禁用restart功能，你需要使用System的spring.devtools.restart.enabled属性
```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 使用trigger file
如果你只想在特定的时间重启，你可以使用trigger file，它是一个特殊的文件，在你重启检查的时候必须被修改。
>spring.devtools.restart.trigger-file

#### 自定义restart classloader
正如前文讲到的，restart是用两个classloader实现的。在大多数情况下都是可以的。但是有时会有问题。

默认情况下，所有在IDE打开的项目都在restart classloader中，常规的.jar文件都在base classloader中。但是当你工作在一个多module的项目中时，不是所有项目都在你的IDE中，这时你就要自定义一些东西。为了能这样做，你可以创建一个META-INF/spring-devtools.properties文件

spring-devtools.properties文件可以包含前缀为restart.exclude和restart.include的属性。include表示元素应该被放到restart classloader中，exclude表示元素应该被放到base classloader中。属性的值可以使正则表达式:
```xml

restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar

```

#### 限制
restart在处理ObjectInputStream的deserialized时表现不佳。如果你想deserialize对象，你最好配合着Thread.currentThread().getContextClassLoader()使用Spring的ConfigurableObjectInputStream。

不幸的是很多第三方的library在deserialize时没有关联context classloader。所以如果你遇到这种问题，你需要想原作者提一个fix request。

### LiveReload
spring-boot-devtools有一个内置的liveReload服务器，可以用来当资源变更时刷新浏览器，LiveReload浏览器插件可以在[livereload.com](https://livereload.com/extensions/)上获得。
如果你不想启用LiveReload功能，就把spring.devtools.livereload.enabled属性设置为false。

### 全局设置
如果你想配置全局的devtools设置，你只需要在自己的home目录下添加一个 .spring-boot-devtools.properties文件。所有在这个文件中添加的属性都会被用到全部的spring boot应用。

### remote 应用
Spring boot的开发者工具不仅仅针对本地发开，在远程运行应用时你也可以使用很多开发者工具的功能。远程支持是选择性的功能。你可以启用它:
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后你需要设置spring.devtools.remote.secret属性：
> spring.devtools.remote.secret=mysecret

> 在远程应用中启用spring-boot-devtools是有安全问题的，所以你一定不要在生产中使用！

#### 运行remote client应用
远程client应用设计用于在你的本地跑远程的应用。你需要运行org.springframework.boot.devtools.RemoteSpringApplication，并且和你链接的远程项目有相同的classpath。应用的唯一必须参数是你想要连接的URL
假设你想要跑一个项目名为my-app的项目你需要如下步骤：

- 选择在run菜单选择run configuration
- 创建java application launch Configuration 
- 找到my-app项目
- 使用org.springframework.boot.devtools.RemoteSpringApplication为main class
- 添加https://myapp.cfapps.io 为程序参数(或者是你自己的远程URL)