---
title: Spring Boot系列---构建系统
date: 2018-04-05 22:15:10
categories: 
- 后端 
tags: 
- java 
- SpringBoot
---

## 构建系统
构建项目的一点细节，不看这个按照后面的demo来也可以使用spring boot，但是个人建议看一下。

### 依赖管理
每个版本的springboot都会提供一个它支持的依赖列表，所以在使用中你不需要在你的pom文件中为那些依赖指定版本。使用maven的用户可以继承spring-boot-starter-parent以获得如下好处：

- 使用java8位默认的编译级别
- 源码的encoding为UTF-8
- 获得上面说到的管理常用依赖的版本的好处
- 提供resource filtering功能（针对application*.properties 和 application*.yml）
- 提供插件配置功能

``` xml
	<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.0.RELEASE</version>
	</parent>
```

> 你需要在这里声明spring boot的版本，后面再引入starters时就可以不写版本号了。

你也可以在你的项目中重写单独的依赖。比如你想单独升级其他的spring data版本，可以使用如下写法：

``` xml
	<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
	</properties>
```

### 不使用parent pom
可能在一些情况下你不想或无法使用parent pom文件，这时你依旧可以通过下列手段获取依赖管理的能力。

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

在前面的例子中你无法重写单独的依赖，为了获得这种功能你需要在spring-boot-dependencies前面加上自己需要的依赖，如下所示:

```xml 
    <dependencyManagement>
        <dependencies>
            <!-- Override Spring Data release train provided by Spring Boot -->
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-releasetrain</artifactId>
                <version>Fowler-SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

### 使用spring boot maven插件
spring boot有一个maven插件，它可以把当前项目打包成一个可执行jar包。

```xml 
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

> gradle 和 ant 的大家自己查资料吧。
 
### starters
starters是一系列方便的依赖描述符。有了它你可以获得一站式的依赖服务。比如，你想使用spring和jpa，你可以在项目中添加依赖spring-boot-starter-data-jpa。

[点击这里](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#using-boot-starter) 查看startet。

## 代码结构
spring没有特别指定编码结构，但是下面是一种很好的实践。

### “default“ package
当你个class没有声明package时，它就被认为是default package。不建议使用default package，因为它会对使用@ComponentScan，@EntityScan或@SpringBootApplication等注解造成影响。

### main class的位置
建议把main class放在根package。因为@EnableAutoConfiguration注解一般放在main class，它隐式定义了应用检索报的基础位置。如果main class是根package，你也可以使用@SpringBootApplication注解。
下面展示一种典型的布局:

```xml
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

Application.java按如下方式文件声明main方法:

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
//@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan

public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

### configuration class
spring boot比较偏爱java-base的配置，尽管上面的SpringApplication也可以使用xml配置，但是通常建议你的主配置是一个@Configuration class。通常定义main方法的类是主@Configuration类的好的选择。

你不需要把全部的@Configuration放在一个class中。 @Import注解用来引入其他的Configuration类。你也可以使用@ComponentScan来自动的收集包含@Configuration类。

如果你必须使用xml类型的配置，建议你依然由@Configuration为开始类，然后使用@ImportResource注解来加载XML配置文件。

### auto-configuration
spring boot的auto-configuration机制会根据你添加的jar包依赖自动的配置你的应用。比如，如果HSQLDB在你的classpath中，并且你没有手动的配置任何数据库连接bean，那么spring boot就会自动配置一个内存数据库。
你可以通过在你的@Configuration类中添加@EnableAutoConfiguration或@SpringBootApplication注解来开启auto-configuration。
> 你应该永远只添加一个@EnableAutoConfiguration注解。我们通常建议你把它放到主@Configuration类中。

### 优雅的替换auto-Configuration
auto-configuration是非侵入性的。在任何时刻，你都可以定义自己的配置来替换auto-configuration的默认配置。比如，如果你添加自己的DataSource bean，那么默认的内嵌数据库的支持就移除了。

如果你想发现当前auto-configuration实现的都是什么，你可以在启动任务的时候添加--debug。

### 禁用指定的auto-configuration类
如果你不想使用指定的auto-configuration类，你可以使用@EnableAutoConfiguration的exclude属性来禁用它。

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```
如果class不在classpath中，你可以使用excludeName属性指定全限定名。最后，你也可以通过spring.autoconfigure.exclude属性来控制auto-configuration禁用的class。

