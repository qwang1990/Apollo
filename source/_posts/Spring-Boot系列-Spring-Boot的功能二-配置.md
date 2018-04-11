---
title: Spring Boot系列---Spring Boot的功能二(配置)
date: 2018-04-09 22:52:52
categories: 
- 后端 
tags: 
- java 
- SpringBoot
---
## Spring Boot的功能(配置)
### 配置外化
SpringBoot让你可以外化你的配置，这样你就可以在不同的环境中跑同一份代码，因为配置已经外化了。你可以使用properties文件，yaml文件，环境变量，或者是命令行参数来外化配置。配置的值可以通过@Value注解注入到bean中，或者通过Environment对象访问，或者通过@ConfigurationProperties来绑定到结构化的对象上去。

Spring Boot提供了一个特别详细的配置覆盖顺序，如下：

- 当前文提到的devtools生效的情况下，devtools的全局配置(~/.spring-boot-devtools.properties)
- Test上的@TestPropertySource注解
- Test上的@SpringBootTest#propertie注解属性
- 命令行参数
- SPRING_APPLICATION_JSON中的属性
- ServeletConfig初始参数
- ServeletConext初始参数
- java:comp/env下的JNDI属性
- Java 系统配置(System.getProperties())
- OS环境变量
- RandomValuePropertySource ，它只有random.\*的属性
- jar包外的[指定环境的应用属性](#1)(application-{profile}.properties and YAML variants)
- jar包内的[指定环境的应用属性](#1)(application-{profile}.properties and YAML variants)
- jar包外的应用属性(application.properties and YAML variants)
- jar包内的应用属性(application.properties and YAML variants)
- @Configuration类上的@PropertySource注解
- 默认属性(通过SpringApplication.setDefaultProperties指定的)

下面看一个实际的例子，假设你有一个@Component，它里面有一个name属性，如下:

```java
import org.springframework.stereotype.*
import org.springframework.beans.factory.annotation.*

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```
在你的项目路径里(比如，在你的jar里)你可以建一个application.properties文件来提供默认的name属性值。当在一个别的环境的时候，可以在jar包外面提供application.properties文件来重写改属性。作为一次性的测试，你可以使用特定的命令行来替换(比如:java -jar app.jar --name="Spring")

>SPRING_APPLICATION_JSON属性可以通过命令行以环境变量的形式来提供。比如，你可以像如下例子在UN*X shell上操作:<br>
$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar<br>
按照上面的操作，你最终会在spring environment中有acem.name属性，且值为test。你也可以以spring.application.json系统属性的形式提供json数据，如下:<br>
$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar<br>
你也可以通过命令行的形式提供JSON数据，如下：<br>
$ java -jar myapp.jar --spring.application.json='{"name":"test"}'<br>
你也可以以JNDI变量的形式提供json数据，如:java:comp/env/spring.application.json

#### 配置随机值
RandomValuePropertySource用于注入随机值。它可以提供Integer,long,uuid,string等，如下所示:
```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

#### 命令行参数
默认情况下SpringApplication会吧命令行可选参数(以--开头的参数，比如--server.port=9000)加到Spring环境中去。正如上文提到的，命令行参数比其他属性源的参数的优先级高。

如果你不想让命令行参数加到环境中，你可以使用SpringApplication.setAddCommandLineProperties(false)来禁用它。

#### 应用属性文件
SpringApplication从下面几个地方加载application.properties文件:

- 当前目录的 /config子目录
- 当前目录
- classpath下的/config包
- classpath根目录

上面是按照优先级顺序排序的，前面的会覆盖后面的
>你也可以用YAML文件替换上面的.properties文件

如果你不喜欢以application.properties作为配置文件的名字，你可以使用spring.config.name环境变量的值来替换它。你也可以使用spring.config.location环境变量来显示的定位文件。下面的例子展示了如何去指定一个不同的文件名:
>$ java -jar myproject.jar --spring.config.name=myproject

如果你想指定两个文件，可以如下方式:
>$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties

注意:因为spring.config.name和spring.config.location是用于决定那个文件将要被加载的，所以他们必须被定义为环境变量(例如:OS的环境变量，系统变量，命令行参数)

如果spring.config.location的值是目录，要以/结尾。

配置检索的顺序是相反的。默认情况下，配置的locations是classpath:/,classpath:/config/,file:./,file:./config/。但是检索顺序是:

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

当你使用spring.config.location自定义location时，它会替换掉默认的位置。比如，如果spring.config.location的值是classpath:/custom-config/,file:./custom-config/，它的检索顺序是:

1. file:./custom-config/
2. classpath:custom-config/

相比而言，如果自定义location使用的是spring.config.additional-location，它的值会被加到默认location后面，但是它的检索顺序会在默认location之前。比如，如果additional-location的值是classpath:/custom-config/,file:./custom-config/，name检索顺序会如下:

1. file:./custom-config/
2. classpath:custom-config/
3. file:./config/
4. file:./
5. classpath:/config/
6. classpath:/

这种检索顺序使得你可以在一个配置文件中放默认值，然后再别的地方重写他。比如你可以在你应用的application.properties文件中放默认值，然后再运行时用自定义路径中的文件的值来替代它。

#### <span id="1">指定环境的应用属性 </span>

未完待续。。。
