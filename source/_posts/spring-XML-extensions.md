---
title: spring XML extensions
date: 2017-11-19 17:27:56
categories: 
- 后端 
tags: 
- java
- spring
---
# 背景
最近看dubbo源码，其中使用spring加载配置的时候用到了自定义xml标签的功能。所以就看了一下。其实从spring2.0开始就已经有该功能了。

# 步骤
其实自定义xml标签十分简单，只需要一下四步

 - 描述自定义元素的xml schema
 - NamespaceHandler
 - BeanDefinitionParse
 - 在spring上注册上面这些元素
 
# 编写xml schema
创建文件myns.xsd
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.mycompany.com/schema/myns"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:beans="http://www.springframework.org/schema/beans"
        targetNamespace="http://www.mycompany.com/schema/myns"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="dateformat">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType">
                    <xsd:attribute name="lenient" type="xsd:boolean"/>
                    <xsd:attribute name="pattern" type="xsd:string" use="required"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>

```
[具体语法](https://docs.spring.io/spring/docs/4.3.13.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#xsd-configuration)

# NamespaceHandler
用于解析我们上面定义的命名空间
常规用法为每一个顶级的xml元素定义一个bean definition，比如上面的xml schema中dateformat就对应一个bean definition
```java
package org.springframework.samples.xml;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class MyNamespaceHandler extends NamespaceHandlerSupport {

    //在handler被调用前调用
    public void init() {
        //此处为xml顶级元素dateformat注册了一个bean definition
        registerBeanDefinitionParser("dateformat", new SimpleDateFormatBeanDefinitionParser());
    }

}
```
在这个例子中NamespaceHandler中并没有太多逻辑，它其实就是一个分发作用，可以在其上注册很多的bean definition，对应每一个自定义标签。具体的处理逻辑在bean definition中。

# BeanDefinitionParse 
BeanDefinitionParse用于解析一个自定义schema中单独的顶级xml元素。
```java
package org.springframework.samples.xml;

import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser;
import org.springframework.util.StringUtils;
import org.w3c.dom.Element;

import java.text.SimpleDateFormat;

public class SimpleDateFormatBeanDefinitionParser extends AbstractSingleBeanDefinitionParser { 

    //解析后的目标class
    protected Class getBeanClass(Element element) {
        return SimpleDateFormat.class; 
    }

    //如何解析xml
    protected void doParse(Element element, BeanDefinitionBuilder bean) {
        // this will never be null since the schema explicitly requires that a value be supplied
        String pattern = element.getAttribute("pattern");
        bean.addConstructorArg(pattern);

        // this however is an optional property
        String lenient = element.getAttribute("lenient");
        if (StringUtils.hasText(lenient)) {
            bean.addPropertyValue("lenient", Boolean.valueOf(lenient));
        }
    }

}

```

# 在spring上注册上面这些元素
这一步是最后一步，就是把前面的工作串起来，其实就是配置两个文件: 

- META-INF/spring.handlers
  
    配置xml schema 和 namespace handler class的关系，本例中配置如下：
    
    http\://www.mycompany.com/schema/myns=org.springframework.samples.xml.MyNamespaceHandler
    
    > 1. ':'在java的properties文件中是关键字，所以要转移
    > 2. key值http\://www.mycompany.com/schema/myns 对应schema，value就是namespace handler的实现类
    
    
    
- META-INF/spring.schemas 
    
    指定xml schema文件的位置，本例中配置如下：
    http\://www.mycompany.com/schema/myns/myns.xsd=org/springframework/samples/xml/myns.xsd
    
# 使用
经过上面4步以后，在程序就就可以使用自定义的标签了，用法如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:myns="http://www.mycompany.com/schema/myns"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.mycompany.com/schema/myns http://www.mycompany.com/schema/myns/myns.xsd">

    <!--注意这里需要引用标签 xmlns:myns="http://www.mycompany.com/schema/myns"-->
    <!--并且在schemaLocation中加入自定义标签的文职-->


    <!-- as a top-level bean -->
    <myns:dateformat id="defaultDateFormat" pattern="yyyy-MM-dd HH:mm" lenient="true"/>

</beans>

```





