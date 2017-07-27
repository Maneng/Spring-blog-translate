# JOOQ和Springboot支持

**目录**  

- [JOOQ和Springboot支持](#jooq%E5%92%8Cspringboot%E6%94%AF%E6%8C%81)
  - [**1. 概述**](#1-%E6%A6%82%E8%BF%B0)
  - [**2. Maven配置**](#2-maven%E9%85%8D%E7%BD%AE)
    - [2.1. 依赖管理](#21-%E4%BE%9D%E8%B5%96%E7%AE%A1%E7%90%86)
    - [2.2. Dependencies](#22-dependencies)
  - [3.Spring Boot Configuration](#3spring-boot-configuration)
    - [3.1.Initial Boot Config](#31initial-boot-config)
    - [3.3. Bean Configuration](#33-bean-configuration)
  - [Using Spring Boot with jOOQ](#using-spring-boot-with-jooq)
  - [5. Conclusion](#5-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




## **1. 概述**

本教程是对[介绍JOOQ和Spring](/介绍JOOQ和Spring)的简介的介绍，介绍了jOOQ可以在Spring Boot应用程序中使用的方式。

如果您还没有阅读该教程，请查看它并按照Maven依赖关系中的第2节和代码生成的第3部分中的说明进行操作。这将生成代表示例数据库中的表的Java类的源代码，包括Author，Book和AuthorBook。


## **2. Maven配置**

除了上一个教程中的依赖和插件之外，还需要在Maven POM文件中包含几个其他组件，以使jOOQ与Spring Boot一起使用。



### 2.1. 依赖管理

使用Spring Boot的最常见的方法是通过在父元素中声明来继承spring-boot-starter-parent项目。然而，这种方法并不总是适合的，因为它强加了一个继承链，这在许多情况下可能不是用户想要的。



本教程使用另一种方法：将依赖关系管理委托给Spring Boot。为了实现这一点，只需将以下依赖管理元素添加到POM文件中：


```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.3.5.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```


### 2.2. Dependencies


为了使Spring Boot控制jOOQ，需要声明对spring-boot-starter-jooq工件的依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jooq</artifactId>
    <version>1.3.5.RELEASE</version>
</dependency>

```







## 3.Spring Boot Configuration

### 3.1.Initial Boot Config

在我们获得jOOQ支持之前，我们将开始使用Spring Boot来准备工作。

首先，我们将在标准的application.properties文件中利用Boot中的持久性支持和改进以及数据访问信息。

这样，我们可以跳过定义bean并通过单独的属性文件进行配置。

我们将添加URL和凭据来定义我们的嵌入式H2数据库：

```java
spring.datasource.url=jdbc:h2:~/jooq
spring.datasource.username=sa
spring.datasource.password=
```




我们还将定义一个简单的引导应用程序：


```java

@SpringBootApplication
@EnableTransactionManagement
public class Application {
     
}
```

我们将这个简单空白，我们将在另一个配置类 - InitialConfiguration中定义所有其他bean声明。

### 3.3. Bean Configuration

现在我们来定义这个InitialConfiguration类：

```java
@Configuration
public class InitialConfiguration {
    // Other declarations
}
```

Spring Boot已经根据application.properties文件中的属性自动生成并配置了dataSource bean，所以我们不需要手动注册它。以下代码允许将自动配置的DataSource bean注入到一个字段中，并显示如何使用此bean：


```java
@Autowired
private DataSource dataSource;
 
@Bean
public DataSourceConnectionProvider connectionProvider() {
    return new DataSourceConnectionProvider
      (new TransactionAwareDataSourceProxy(dataSource));
}
```

由于名为transactionManager的bean也由Spring Boot自动创建和配置，所以我们不需要像上一个教程一样声明DataSourceTransactionManager类型的其他bean，以利用Spring事务支持。 DSLContext bean的创建方式与前面教程的PersistenceContext类相同：


```java

@Bean
public DefaultDSLContext dsl() {
    return new DefaultDSLContext(configuration());
}
```


最后，需要向DSLContext提供配置实现。由于Spring Boot能够通过在类路径中存在H2工件来识别正在使用的SQL方言，所以不再需要方言配置：


```java

public DefaultConfiguration configuration() {
    DefaultConfiguration jooqConfiguration = new DefaultConfiguration();
    jooqConfiguration.set(connectionProvider());
    jooqConfiguration
      .set(new DefaultExecuteListenerProvider(exceptionTransformer()));
 
    return jooqConfiguration;
}
```

## Using Spring Boot with jOOQ

为了更容易地对jOOQ的Spring Boot支持进行演示，本教程前言中的测试用例对其类级注释进行了一些改动：


```java

@SpringApplicationConfiguration(Application.class)
@Transactional("transactionManager")
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringBootTest {
    // Other declarations
}
```
很明显，Spring Boot不是采用@ContextConfiguration注释，而是使用@SpringApplicationConfiguration来利用SpringApplicationContextLoader上下文加载器来测试应用程序。


插入，更新和删除数据的测试方法与上一教程完全相同。请参阅有关使用jOOQ with Spring的文章的第5部分了解更多信息。所有的测试都应该用新配置成功执行，证明jOOQ完全受到Spring Boot的支持



## 5. Conclusion

本教程深入介绍了使用jOOQ与Spring。它介绍了Spring Boot应用程序利用jOOQ以类型安全的方式与数据库进行交互的方式。