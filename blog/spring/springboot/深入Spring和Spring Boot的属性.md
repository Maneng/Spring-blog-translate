**目录**  

- [Spring和Spring Boot的属性](#spring%E5%92%8Cspring-boot%E7%9A%84%E5%B1%9E%E6%80%A7)
  - [Overview](#overview)
  - [Register in XML](#register-in-xml)
    - [2.1 多个<property-placeholder>](#21-%E5%A4%9A%E4%B8%AAproperty-placeholder)
  - [Register in java annotations](#register-in-java-annotations)
  - [Use Inject properties](#use-inject-properties)
    - [4.1 属性搜索优先级](#41-%E5%B1%9E%E6%80%A7%E6%90%9C%E7%B4%A2%E4%BC%98%E5%85%88%E7%BA%A7)
  - [Properties in springboot](#properties-in-springboot)
    - [5.1 `application.properties` - 默认属性文件](#51-applicationproperties---%E9%BB%98%E8%AE%A4%E5%B1%9E%E6%80%A7%E6%96%87%E4%BB%B6)
    - [5.2 环境特定属性文件(Environment Specific Properties File)](#52-%E7%8E%AF%E5%A2%83%E7%89%B9%E5%AE%9A%E5%B1%9E%E6%80%A7%E6%96%87%E4%BB%B6environment-specific-properties-file)
    - [5.3 测试特定属性文件](#53-%E6%B5%8B%E8%AF%95%E7%89%B9%E5%AE%9A%E5%B1%9E%E6%80%A7%E6%96%87%E4%BB%B6)
    - [5.4 `@TestPropertySource`注释](#54-testpropertysource%E6%B3%A8%E9%87%8A)
    - [5.5 分层属性](#55-%E5%88%86%E5%B1%82%E5%B1%9E%E6%80%A7)
    - [5.6 YAML文件](#56-yaml%E6%96%87%E4%BB%B6)
    - [5.7 命令行参数的属性](#57-%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%8F%82%E6%95%B0%E7%9A%84%E5%B1%9E%E6%80%A7)
    - [5.8 环境变量的属性](#58-%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E7%9A%84%E5%B1%9E%E6%80%A7)
    - [5.9 随机的属性值](#59-%E9%9A%8F%E6%9C%BA%E7%9A%84%E5%B1%9E%E6%80%A7%E5%80%BC)
    - [5.10 属性来源的其他类型](#510-%E5%B1%9E%E6%80%A7%E6%9D%A5%E6%BA%90%E7%9A%84%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B)
  - [Behind the scenes](#behind-the-scenes)
    - [6.1 在Spring3.1 之前](#61-%E5%9C%A8spring31-%E4%B9%8B%E5%89%8D)
    - [6.2 在Spring3.1 之后](#62-%E5%9C%A8spring31-%E4%B9%8B%E5%90%8E)
  - [Before spring3.1](#before-spring31)
    - [7.2. XML configuration](#72-xml-configuration)
  - [After spring3.1](#after-spring31)
    - [8.1. Java configuration](#81-java-configuration)
    - [8.2. XML configuration](#82-xml-configuration)
  - [Parent child contexts](#parent-child-contexts)
    - [9.1. 如果属性文件在XML中使用`<property-placeholder>`定义](#91-%E5%A6%82%E6%9E%9C%E5%B1%9E%E6%80%A7%E6%96%87%E4%BB%B6%E5%9C%A8xml%E4%B8%AD%E4%BD%BF%E7%94%A8property-placeholder%E5%AE%9A%E4%B9%89)
    - [9.2。如果使用`@PropertySource`在Java中定义属性文件](#92%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8propertysource%E5%9C%A8java%E4%B8%AD%E5%AE%9A%E4%B9%89%E5%B1%9E%E6%80%A7%E6%96%87%E4%BB%B6)
  - [Conclusion](#conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Spring和Spring Boot的属性


---
## Overview


这篇博客将介绍如何通过XML和`<property-placeholder>`
或Java配置和`@PropertySource`在Spring中设置和使用属性。

在Spring3.1之前，将新的属性文件添加到Spring并使用属性值并不像现在那样灵活和可靠，
从Spring 3.1开始，新的Environment和PropertySource抽象简化了整个过程。



---
## Register in XML

在XML中，Spring可以通过`<context：property-placeholder ...>`命名空间元素访问新的属性文件：

```$xslt
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xsi:schemaLocation="
      http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
      http://www.springframework.org/schema/context 
      http://www.springframework.org/schema/context/spring-context-4.2.xsd">
 
      <context:property-placeholder location="classpath:foo.properties" />
 
</beans>
```

foo.properties文件应放在/ src / main / resources下，以便它在运行时在类路径上可用

### 2.1 多个<property-placeholder>
如果在Spring上下文中存在多个`<property-placeholder>`元素，则需要遵循以下几个最佳做法：

- 需要指定order属性来修正Spring所处理的顺序
- 所有属性占位符减去最后一个（最高次序）应该设置ignore-unresolvable =“true”，允许解析机制在上下文中传递给其他人，而不会抛出异常


---
## Register in java annotations
Spring 3.1还引入了新的`@PropertySource`注释，作为向环境添加属性源的便利机制。
此注释将与基于Java的配置和`@Configuration`注释结合使用：

```$xslt
@Configuration
@PropertySource("classpath:foo.properties")
public class PropertiesWithJavaConfig {
 
   @Bean
   public static PropertySourcesPlaceholderConfigurer
     propertySourcesPlaceholderConfigurer() {
      return new PropertySourcesPlaceholderConfigurer();
   }
}
```

与使用XML命名空间元素相反，
Java `@PropertySource`注释不会使用Spring自动注册`PropertySourcesPlaceholderConfigurer`

相反，bean必须在配置中显式定义，
以获取属性解析机制。这个意想不到的行为背后的原因是设计并记录在这个问题上。

注册新的属性文件的另一个非常有用的方式是使用占位符来允许您在运行时动态选择正确的文件,例如：

```
@PropertySource({ 
  "classpath:persistence-${envTarget:mysql}.properties"
})
...
```


---
## Use Inject properties

使用`@Value`注释注入属性很简单：

```$xslt
@Value( "${jdbc.url}" )
private String jdbcUrl;
```


也可以指定属性的默认值：
```$xslt
@Value( "${jdbc.url:aDefaultUrl}" )
private String jdbcUrl;
```
在Spring XML配置中这样使用属性：

```$xslt
<bean id="dataSource">
  <property name="url" value="${jdbc.url}" />
</bean>
```

旧版`PropertyPlaceholderConfigurer`和Spring 3.1中新增的`PropertySourcesPlaceholderConfigurer`都会解析bean定义属性值和`@Value`注释中的`$ {...}`占位符。
最后使用新的`Environment API`获取属性的值：

```$xslt
@Autowired
private Environment env;
...
dataSource.setUrl(env.getProperty("jdbc.url"));
```

这里一个非常重要的注意事项是使用`<property-placeholder>`不会将属性暴露给Spring环境 
这意味着检索这样的值将不起作用 - 它将返回null：


```$xslt
env.getProperty("key.something")
```

### 4.1 属性搜索优先级
默认情况下，在Spring 4中，本地属性最后搜索环境属性源，包括属性文件。
这里的一个简单的说法是，本地属性是通过基础`PropertiesLoaderSupport`类（setProperties，setLocation等）
中的setter API手动/编程配置的属性。

可以通过`PropertySourcesPlaceholderConfigurer`的`localOverride`属性覆盖此行为，
该属性可以设置为true，以允许本地属性覆盖文件属性。

在Spring 3.0和更早版本中，旧的`PropertyPlaceholderConfigurer`也尝试在手动定义的源和系统属性中查找属性。
查询优先级也可以通过configurer的`systemPropertiesMode`属性这样来定制：

- never 不要检查系统属性
- fallback(默认) 如果在指定的属性文件中不可解析，则检查系统属性
- override  首先检查系统属性，然后再尝试指定的属性文件。这允许系统属性覆盖任何其他属性源

最后，请注意，如果通过`@PropertySource`定义的两个或多个文件中定义了一个属性，则最后一个定义将胜出并覆盖以前的一个。
这使得确切的属性值难以预测，因此如果覆盖很重要，则可以使用PropertySource API。

在进行配置的低级别之前，我们来做一些使用属性文件的实际工作。




---
## Properties in springboot

在我们进入更多高级配置选项的属性之前，我们花一些时间查看Spring Boot中的新属性支持。
总的来说，与标准Spring相比，这种新的支持涉及的配置较少,这当然也是Boot的主要目标之一。

### 5.1 `application.properties` - 默认属性文件

引导应用是约定由于配置的，
这意味着我们可以简单地在“`src / main / resources`”目录中放置一个“`application.properties`”文件
，并将自动检测。然后我们可以正常注入任何加载的属性。

所以，通过使用这个默认文件，我们不必明确注册一个PropertySource，也不用提供一个属性文件的路径。

此外，如果需要，我们还可以在运行时使用环境属性来配置不同的文件：
```$xslt
java -jar app.jar --spring.config.location=classpath:/another-location.properties
```

### 5.2 环境特定属性文件(Environment Specific Properties File)
如果我们需要针对不同的环境，那么在Boot中有一个内置机制。
我们可以在“`src / main / resources`”目录中简单地定义一个“`application-environment.properties`”文件，
然后设置一个具有相同环境名称的Spring配置文件。

例如，如果我们定义一个“暂存”环境，那意味着我们必须定义一个暂存配置文件，
然后定义一个`application-staging.properties`。

该`env`文件将被加载，并将优先于默认的属性文件。请注意，默认文件仍将被加载，
只是当有属性冲突时，环境特定属性文件优先。

### 5.3 测试特定属性文件
当我们的应用程序被测试时，我们也可能需要使用不同的属性值。
`Spring Boot`在测试运行期间通过查看我们的“`src / test / resources`”目录来处理这个问题。同样，默认属性仍然是正常注入的，
但是如果存在冲突，那么这些属性将被覆盖。

### 5.4 `@TestPropertySource`注释

如果我们需要对测试属性进行更精细的控制，那么我们可以使用`@TestPropertySource`注释。

这允许我们为特定的测试上下文设置测试属性，优先于默认的属性源：

``` java
@ContextConfiguration
@TestPropertySource("/my-test.properties")
public class IntegrationTests {
    // tests
}
```

如果我们不想使用文件，我们可以直接指定名称和值：

``` java
@ContextConfiguration
@TestPropertySource("foo=bar", "bar=foo")
public class IntegrationTests {
    // tests
}
```
我们也可以使用`@SpringBootTest`注释的`properties`参数来实现类似的效果:


``` java
@SpringBootTest(properties = {"foo=bar", "bar=foo"})
public class IntegrationTests {
    // tests
}
```

### 5.5 分层属性
如果我们将属性分组在一起，我们可以使用`@ConfigurationProperties`注释，
将这些属性层次结构映射到Java对象图。
```
database.url=jdbc:postgresql:/localhost:5432/instance
database.username=foo
database.password=bar

```
然后让我们使用注释将它们映射到数据库对象：
``` java
@ConfigurationProperties(prefix = "database")
public class Database {
    String url;
    String username;
    String password;
 
    // standard getters and setters
}
```

Spring Boot再次使用它的约定优于配置，自动映射属性名称及其对应的字段。我们至需要提供的是属性前缀。

如果您想深入了解配置属性，请查看深入的文章。[the in-depth article](http://www.baeldung.com/configuration-properties-in-spring-boot)


### 5.6 YAML文件 

还支持YAML文件。 所有相同的命名规则适用于特定于测试，环境特定和默认属性文件。
唯一的区别是文件扩展名，并且对您的类路径上的SnakeYAML库的依赖。
 YAML特别适用于分层属性存储，以下属性文件：
 
 ```$xslt
database.url=jdbc:postgresql:/localhost:5432/instance
database.username=foo
database.password=bar
secret: foo
```

与以下YAML文件同义：
```$xslt
database:
  url: jdbc:postgresql:/localhost:5432/instance
  username: foo
  password: bar
secret: foo
```

值得我们注意的是，YAML文件不支持`@PropertySource`注释，
因此如果使用注释是必需的，它将限制我们使用属性文件。

### 5.7 命令行参数的属性
与使用文件相反，属性可以直接在命令行中传递：
```$xslt
java -jar app.jar --property="value"
```
您还可以通过系统属性来执行此操作，这些属性在-jar命令之前提供，而不是之后:
```$xslt
java -Dproperty.name="value"-jar app.jar
```

### 5.8 环境变量的属性

Spring Boot还将检测环境变量，将其视为属性：

```$xslt
export name=value
java -jar app.jar
```

### 5.9 随机的属性值
如果我们不想要确定性属性值，则可以使用`RandomValuePropertySource`来随机分配属性值：

```$xslt
random.number=${random.int}
random.long=${random.long}
random.uuid=${random.uuid}
```

### 5.10 属性来源的其他类型
Spring Boot支持大量的属性来源，实现一个很好的排序，来明智的覆盖。想要深入可以深入官方文档。

---

## Behind the scenes

### 6.1 在Spring3.1 之前
Spring 3.1引入了使用注释定义属性源的方便选项，但在此之前，XML配置是必需的。

在上下文中，`<context：property-placeholder>` XML元素自动注册一个新的`PropertyPlaceholderConfigurer bean`。为了向后兼容，
Spring 3.1及更高版本中的XSD架构尚未升级到指向新的3.1 XSD版本。

### 6.2 在Spring3.1 之后
Spring 3.1起，XML `<context：property-placeholder>`将不再注册旧的PropertyPlaceholderConfigurer
，而是新引入的`PropertySourcesPlaceholderConfigurer`。这个替换类被创建为更灵活，
更好地与新引入的`Environment`和`PropertySource`机制进行交互。

对于使用Spring 3.1或更高版本的应用程序，应将其视为标准。


---
## Before spring3.1
除了将属性添加到Spring - 注释和XML命名空间的方便之外，还可以手动定义和注册属性配置bean。
使用`PropertyPlaceholderConfigurer`可以让我们完全控制配置，而不必要的是更冗长和更多的时间。
7.1 Java configuration

``` java
   @Bean
   public static PropertyPlaceholderConfigurer properties() {
     PropertyPlaceholderConfigurer ppc
       = new PropertyPlaceholderConfigurer();
     Resource[] resources = new ClassPathResource[]
       { new ClassPathResource( "foo.properties" ) };
     ppc.setLocations( resources );
     ppc.setIgnoreUnresolvablePlaceholders( true );
     return ppc;
   }

```

### 7.2. XML configuration
```
<bean
  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath:foo.properties</value>
        </list>
    </property>
    <property name="ignoreUnresolvablePlaceholders" value="true"/>
</bean>

```


---
## After spring3.1

类似地，在Spring 3.1中，还可以手动配置新的`PropertySourcesPlaceholderConfigurer`：

### 8.1. Java configuration

```$xslt
@Bean
public static PropertySourcesPlaceholderConfigurer properties(){
  PropertySourcesPlaceholderConfigurer pspc
    = new PropertySourcesPlaceholderConfigurer();
  Resource[] resources = new ClassPathResource[ ]
    { new ClassPathResource( "foo.properties" ) };
  pspc.setLocations( resources );
  pspc.setIgnoreUnresolvablePlaceholders( true );
  return pspc;
}


```

### 8.2. XML configuration

```$xslt

<bean
  class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath:foo.properties</value>
        </list>
    </property>
    <property name="ignoreUnresolvablePlaceholders" value="true"/>
</bean>
```


---
## Parent child contexts
这个问题一再出现 - 当您的Web应用程序有父级和子级上下文时会发生什么？
父上下文可能具有一些常见的核心功能和bean，然后一个（或多个）
子上下文可能包含特定于servlet的bean。 在这种情况下，
定义属性文件并将它们包含在这些上下文中的最佳方式是什么？
还有什么 - 如何从Spring最好地检索这些属性？这是简单的分解。

### 9.1. 如果属性文件在XML中使用`<property-placeholder>`定义

如果文件在父上下文中定义：
- @Value在Child context中工作：NO
- @Value works in Parent context: YES

如果文件在子上下文中定义：
- @Value在Child context中工作：YES
- @Value在Child context中工作：NO

最后，如前所述，<property-placeholder>不会将属性暴露给环境，所以：
- environment.getProperty在任一上下文中使用：NO

### 9.2。如果使用`@PropertySource`在Java中定义属性文件
如果文件在父上下文中定义：
- @Value在Child context中工作：YES
- @Value works in Parent context: YES
- environment.getProperty in Child context: YES
- environment.getProperty in Parent context: YES

如果文件在子上下文中定义：
- @Value在Child context中工作：YES
- @Value works in Parent context: NO
- environment.getProperty in Child context: YES
- environment.getProperty in Parent context: NO


---
## Conclusion 

本文介绍了在Spring中使用属性和属性文件的几个示例。
一如以往，支持该文章的整个代码可以在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)上获得。
这是一个基于Maven的项目，所以应该很容易导入和运行。