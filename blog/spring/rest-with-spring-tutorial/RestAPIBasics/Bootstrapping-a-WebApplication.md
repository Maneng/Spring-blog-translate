# **Bootstrapping a Web Application**

##  目录

* **1.[概述](#overview)**
* **2.[The Maven pom.xml](#the-maven-pom.xml)**
    * 2.1 **[Justification of the cglib dependency](#justification-of-the-cglib-dependency)**
    * 2.2 **[The cglib dependency in Spring 3.2 and beyond](#the-cglib-dependency-in-Spring-3.2-and-beyond)**
* **3.[ The Java-based web configuration](#the-Java-based-web-configuration)**
    * 3.1 **[The web.xml](#the-web.xml)**

* **4.[Conclusion](#conclusion)**


---
## Overview

本教程将说明如何使用Spring引导Web应用程序，
并讨论如何从XML跳转到Java，而无需完全迁移整个XML配置。

---
## The Maven pom.xml

``` xml
   <project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
      http://maven.apache.org/POM/4.0.0 
      http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>org</groupId>
   <artifactId>rest</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <packaging>war</packaging>
 
   <dependencies>
 
      <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>${spring.version}</version>
         <exclusions>
            <exclusion>
               <artifactId>commons-logging</artifactId>
               <groupId>commons-logging</groupId>
            </exclusion>
         </exclusions>
      </dependency>
       
   </dependencies>
 
   <build>
      <finalName>rest</finalName>
 
      <plugins>
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
               <source>1.6</source>
               <target>1.6</target>
               <encoding>UTF-8</encoding>
            </configuration>
         </plugin>
      </plugins>
   </build>
 
   <properties>
      <spring.version>4.0.5.RELEASE</spring.version>
   </properties>
 
</project>
```
### 2.1. The cglib Dependency Before Spring 3.2

你可能会想知道为什么cglib是一个依赖关系 - 事实证明有一个有效的理由来包含它 
- 如果没有它，整个配置就不能运行。如果删除，Spring会抛出：
导致：**java.lang.IllegalStateException：**需要CGLIB来处理**@Configuration**类。
将CGLIB添加到类路径或删除以下@Configuration bean定义

发生这种情况的原因是Spring处理**@Configuration**类的方式。
这些类实际上是bean，他们需要Context上下文环境，并且要尊重其他bean语义和范围。
通过为每一个具有**@Configuration**类动态创建具有此意识的cglib代理来实现，因此cglib依赖性。


另外，由于这个原因，配置注释类有一些限制：

* Configuration classes should not be final （不能是final类）
* They should have a constructor with no arguments （无参构造方法）

### 2.2. The cglib Dependency in Spring 3.2 and Beyond
从Spring 3.2开始，不再需要添加cglib作为显式依赖。这是因为Spring现在正在内联了cglib，
这将确保所有基于类的代理功能都将与Spring 3.2一起开箱即用。

新的cglib代码放在Spring包下：**org.springframework.cglib**（替换原来的**net.sf.cglib**）。
软件包更改的原因是避免与类路径中已经存在的任何cglib版本冲突。


---
## The Java-based Web Configuration

``` java
@Configuration
@ImportResource({
  "classpath*:/rest_config.xml"
})
@ComponentScan( basePackages = "org.rest" )
@PropertySource({
  "classpath:rest.properties", 
  "classpath:web.properties"
})
public class AppConfig{
 
   @Bean
   public static PropertySourcesPlaceholderConfigurer
     properties() {
  
      return new PropertySourcesPlaceholderConfigurer();
   }
}
```

首先，**@Configuration**注释 - 这是基于Java的Spring配置使用的主要工件;它本身是用@Component进行元注释的，
它使得注释类标准的bean也是组件扫描的候选。
 @Configuration类的主要目的是作为Spring IoC容器的bean定义的源。有关更详细的说明，请参阅官方文档。


然后，**@ImportResource**用于导入现有的基于XML的Spring配置

这可能是仍然从XML迁移到Java的配置，或者简单地您希望保留的旧配置。无论哪种方式，
将其导入容器对于成功迁移至关重要，
从而允许小步骤，而不会有太多的风险。替换的等效XML注释是：

`<import resource=”classpath*:/rest_config.xml” />`


转到@ComponentScan - 这将配置组件扫描指令，有效地替换XML：
`
<context:component-scan base-package="org.rest" />
`
@Configuration类不应该被自动发现，因为它们已经被Container指定和使用 - 
允许它们被重新发现并引入到Spring上下文中将导致以下错误：

```
Caused by: org.springframework.context.annotation.ConflictingBeanDefinitionException: 
Annotation-specified bean name ‘webConfig’ for bean class [org.rest.spring.AppConfig] 
conflicts with existing, non-compatible bean definition of same name and class
 [org.rest.spring.AppConfig]

```

最后，使用@Bean注释配置属性支持 - **PropertySourcesPlaceholderConfigurer**在@Bean注释方法中初始化，
表示它将生成由Container管理的Spring bean。此新配置已替代以下XML：

```

<context:property-placeholder
location="classpath:persistence.properties, classpath:web.properties"
ignore-unresolvable="true"/>
```


---
## The web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="
    http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
       http://java.sun.com/xml/ns/javaee 
       http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    id="rest" version="3.0">
 
   <context-param>
      <param-name>contextClass</param-name>
      <param-value>
         org.springframework.web.context.support.AnnotationConfigWebApplicationContext
      </param-value>
   </context-param>
   <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>org.rest.spring.root</param-value>
   </context-param>
   <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
 
   <servlet>
      <servlet-name>rest</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <init-param>
         <param-name>contextClass</param-name>
         <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
         </param-value>
      </init-param>
      <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>org.rest.spring.rest</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
      <servlet-name>rest</servlet-name>
      <url-pattern>/api/*</url-pattern>
   </servlet-mapping>
 
   <welcome-file-list>
      <welcome-file />
   </welcome-file-list>
 
</web-app>

```
首先，根上下文被定义并配置为使用**AnnotationConfigWebApplicationContext**
而不是默认的**XmlWebApplicationContext**。
较新的**AnnotationConfigWebApplicationContext**接受@Configuration注释类作为Container配置的输入，
并且需要设置基于Java的上下文。


与XmlWebApplicationContext不同，它假定没有默认的配置类位置，因此必须设置Servlet的“contextConfigLocation”init-param。
这将指向@Configuration类所在的java包;还支持类的完全限定名称。
接下来，DispatcherServlet配置为使用相同类型的上下文，唯一的区别是它将配置类加载到不同的包中。

除此之外，web.xml并没有真正从XML转变为基于Java的配置。



---
## Conclusion

所提出的方法允许Spring配置从XML平滑迁移到Java，混合旧的和新的。这对于旧的项目很重要，这些项目可能会有大量基于XML的配置，无法一次迁移。
这样，在迁移过程中，XML bean可以以小的增量移植。

在下一篇关于REST的文章中，我介绍了项目中的MVC设置，HTTP状态代码的配置，有效负载调度和内容协商

一如以往，支持该文章的整个代码可以在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)上获得。
这是一个基于Maven的项目，所以应该很容易导入和运行。

