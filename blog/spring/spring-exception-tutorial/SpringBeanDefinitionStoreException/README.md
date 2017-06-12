# Spring BeanDefinitionStoreException

## **1.概述**

在这片文章中，我们会讨论关于Spring的异常：
> org.springframework.beans.factory.BeanDefinitionStoreException

当**bean**定义无效的时候，通常是 **BeanFactory**的责任，加载的**bean**是有问题的，本文中将会讨论这种常见的异常，
以及它们解决方案。

## **2.Cause: java.io.FileNotFoundException**

**BeanDefinitionStoreException**可能是由底层**IOException**引起的多种可能的原因：

1.从**ServletContext**资源解析XML文档的**IOException**

这通常发生在Spring Web应用程序中，当在Spring MVC的web.xml中设置DispatcherServlet时：

```$xml
    <servlet>  
       <servlet-name>mvc</servlet-name>  
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
    </servlet>
```

默认情况下，Spring将在Web应用程序的/ WEB-INF目录中查找一个名为“springMvcServlet-servlet.xml”的文件。

如果此文件不存在，则将抛出以下异常：

``` 
    org.springframework.beans.factory.BeanDefinitionStoreException: 
    IOException parsing XML document from ServletContext resource [/WEB-INF/mvc-servlet.xml]; 
    nested exception is java.io.FileNotFoundException: 
    Could not open ServletContext resource [/WEB-INF/mvc-servlet.xml]
```

解决方案当然要确保mvc-servlet.xml文件确实存在于/WEB-INF下;如果没有，则可以简单的创建一个：

```$xslt
<?xml version="1.0" encoding="UTF-8"?>
<beans
   xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
      http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans-3.2.xsd" >
 
</beans>
```

2. IOException parsing XML document from class path resource

这通常发生在应用程序中的某些内容指向不存在的XML资源，或者不在其中应用的位置。
指向这样的资源可能以各种方式发生,我们可以用java Configuration这样来创建一个：

```$xslt
@Configuration
@ImportResource("beans.xml")
public class SpringConfig {...}
```

在XML中，看起来像这样：

```$xslt
<import resource="beans.xml"/>
```

或者像这样手动来创建Spring XML上下文：

```$xslt
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
```

如果文件不存在，所有这些将导致相同的异常：

```$xslt
org.springframework.beans.factory.BeanDefinitionStoreException: 
IOException parsing XML document from ServletContext resource [/beans.xml]; 
nested exception is java.io.FileNotFoundException: 
Could not open ServletContext resource [/beans.xml]
```

这个解决方案是创建文件并将其放在项目的/ src / main / resources目录下，
这样，该文件将存在于类路径中，并将在Spring中找到并使用


## **2.Cause: Could not resolve placeholder …**

当Spring尝试解析属性但是由于许多可能的原因之一时并不能解析时，会发生此错误。
我们可能会在XML中这样使用属性：

```$xslt
... value="${some.property}" ..
```

当然，我们也可以在java代码中使用这样的属性，像这样：

```$xslt
@Value("${some.property}")
private String someProperty;
```

发生这个异常的时候，首先要检查的是，该属性的名称实际上与属性定义是否匹配。
在这个例子中，我们需要定义以下属性：

```$xslt
some.property=someValue
```

然后，我们需要检查在Spring中定义属性文件的位置.
这在我的属性与Spring教程中有详细描述。
最好的做法是将所有属性文件放在应用程序的/ src / main / resources目录下，并通过以下方式加载它们：
