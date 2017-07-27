# Spring MVC中的ViewResolver指南
**目录**  

- [Spring MVC中的ViewResolver指南](#spring-mvc%E4%B8%AD%E7%9A%84viewresolver%E6%8C%87%E5%8D%97)
  - [**1. Overview**](#1-overview)
  - [**2. The Spring Web Configuration**](#2-the-spring-web-configuration)
  - [**3. Add an InternalResourceViewResolver**](#3-add-an-internalresourceviewresolver)
  - [**4. Add a ResourceBundleViewResolver**](#4-add-a-resourcebundleviewresolver)
  - [**5. Add an XmlViewResolver**](#5-add-an-xmlviewresolver)
  - [**6. Chaining ViewResolvers and Define an Order Priority**](#6-chaining-viewresolvers-and-define-an-order-priority)
  - [**7. Conclusion**](#7-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## **1. Overview**

所有MVC框架都提供了一种处理视图的方法。

Spring通过视图解析器执行此操作，使您能够在浏览器中呈现模型，而不将实现与特定视图技术相结合。

ViewResolver将视图名称映射到实际视图。

而且Spring框架还有很多视图解析器。 InternalResourceViewResolver，XmlViewResolver，ResourceBundleViewResolver等几个。

这是一个简单的教程，介绍如何设置最常见的视图解析器，以及如何在同一配置中使用多个ViewResolver。

## **2. The Spring Web Configuration**

我们从Web配置开始吧！我们将使用@EnableWebMvc，@Configuration和@ComponentScan来注释它：

``` java
@EnableWebMvc
@Configuration
@ComponentScan("org.baeldung.web")
public class WebConfig extends WebMvcConfigurerAdapter {
    // All web configuration will go here
}
```


在这里，我们将在配置中设置视图解析器。


## **3. Add an InternalResourceViewResolver**

此ViewResolver允许我们为视图名称设置诸如前缀或后缀的属性以生成最终视图页面URL：

``` java
@Bean
public ViewResolver internalResourceViewResolver() {
    InternalResourceViewResolver bean = new InternalResourceViewResolver();
    bean.setViewClass(JstlView.class);
    bean.setPrefix("/WEB-INF/view/");
    bean.setSuffix(".jsp");
    return bean;
}
```
为了实例的这种简单性，我们不需要一个控制器来处理请求。 我们只需要一个简单的jsp页面，放置在/ WEB-INF / view文件夹中，如配置中所定义：



``` html
<html>
    <head></head>
    <body>
        <h1>This is the body of the sample view</h1>
    </body>
</html>
```

## **4. Add a ResourceBundleViewResolver**

由于此解析器的名称建议ResourceBundleViewResolver在ResourceBundle中使用bean定义。 首先，我们将ResourceBundleViewResolver添加到以前的配置中：

``` java
@Bean
public ViewResolver resourceBundleViewResolver() {
    ResourceBundleViewResolver bean = new ResourceBundleViewResolver();
    bean.setBasename("views");
    return bean;
}
```
bundle 通常在属性文件中定义，位于类路径中。以下是views.properties文件：

``` stata
sample.(class)=org.springframework.web.servlet.view.JstlView
sample.url=/WEB-INF/view/sample.jsp
```
我们可以使用上述示例中定义的简单jsp页面进行此配置。


## **5. Add an XmlViewResolver**

ViewResolver的这种实现使用与Spring的XML bean工厂相同的DTD来接受用XML编写的配置文件：

``` java
@Bean
public ViewResolver xmlViewResolver() {
    XmlViewResolver bean = new XmlViewResolver();
    bean.setLocation(new ClassPathResource("views.xml"));
    return bean;
}
```
下面是配置文件，views.xml：


``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">
  
    <bean id="xmlConfig" class="org.springframework.web.servlet.view.JstlView">
        <property name="url" value="/WEB-INF/view/xmlSample.jsp" />
    </bean>
  
</beans>
```
对于前面的例子，我们可以使用我们以前定义的简单的jsp页面。


## **6. Chaining ViewResolvers and Define an Order Priority**

Spring MVC还支持多个视图解析器。
这允许您在某些情况下覆盖特定视图。我们可以通过向配置添加多个解析器来简单地链接视图解析器。

一旦这样做，我们需要为这些解析器定义一个顺序。 order属性用于定义链中调用顺序。order属性（最大order号）越高，视图解析器在链中的位置越晚。


要定义顺序，我们可以将以下代码行添加到我们的视图解析器的配置中：

>bean.setOrder(0);

要注意order优先级，因为InternalResourceViewResolver应该有更高的order - 因为它的目的是表示非常明确的映射。如果其他解析程序具有更高的顺序，那么可能永远不会调用InternalResourceViewResolver。

## **7. Conclusion**

在本教程中，我们使用Java配置配置了一组视图解析器。通过玩优先顺序，我们可以设置它们的调用顺序。