# springframework 有关的面试问题
**目录**  

- [springframework 有关的面试问题](#springframework-%E6%9C%89%E5%85%B3%E7%9A%84%E9%9D%A2%E8%AF%95%E9%97%AE%E9%A2%98)
  - [**1. Introduction**](#1-introduction)
  - [**2. Spring Core**](#2-spring-core)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




## **1. Introduction**

在本文中，我们将介绍在面试过程中可能出现的一些最常见的Spring相关问题

## **2. Spring Core**


**Q1. What is Spring Framework?**

Spring是Java Enterprise Edition应用程序开发中最广泛使用的框架。 Spring的核心功能可用于开发任何Java应用程序

我们可以使用其扩展来构建Java EE平台之上的各种Web应用程序，或者我们可以在简单的独立应用程序中使用其依赖注入的功能


**Q2. What are the benefits of using Spring?**

Spring的目标是使Java EE开发更容易。以下是使用它的优点：
- **Lightweight**（轻量级）在开发中使用框架只有一个轻微的开销
- **Inversion of Control (IoC):** Spring容器负责各种对象的依赖管理关系，而不是创建或查找依赖对象
- **Aspect Oriented Programming (AOP):**  Spring支持AOP将业务逻辑与系统服务分开
- **MVC framework:** 用于创建能够返回XML / JSON响应的Web应用程序或RESTful Web服务
- **Transaction management:** 通过使用Java注释或Spring Bean XML配置文件来减少JDBC操作，文件上传等中的相同代码量
- **Exception Handling:** Spring提供了一种方便的API来将技术特定的异常转换为未检查的异常

**Q3. What Spring sub-projects do you know? Describe them briefly.**
- **Core** -提供框架的基本部分的关键模块，如IoC或DI
- **JDBC**  - 该模块支持JDBC抽象层，无需为特定的供应商数据库执行JDBC编码
- **ORM integration** - 为流行的对象关系映射API（如JPA，JDO和Hibernate）提供集成层
- **Web**  - 面向Web的集成模块，提供多文件上传，Servlet监听器和面向Web的应用程序上下文功能
- **MVC framework**  - 实现模型视图控制器设计模式的Web模块
- **AOP module**  - 面向方面的编程实现，允许定义清除方法拦截器和切入点

**Q4. What is Dependency Injection?**

依赖注入是控制反转（IoC）的一个方面，它是一个通用概念，说明您不是手动创建对象，而是描述如何创建对象。如果需要，IoC容器将实例化所需的类。


**Q5. How can we inject beans in Spring?**

存在几个不同的选择：
- Setter Injection
- Constructor Injection
- Field Injection

可以使用XML文件或注释来完成配置。


**Q6. Which is the best way of injecting beans and why?**
推荐的方法是使用构造函数参数作为强制依赖关系和setters候选. 。构造器注入允许将值注入到不可变字段，并使测试更容易。

**Q7. What is the difference between BeanFactory and ApplicationContext?**

BeanFactory是一个表示提供和管理bean实例的容器的接口。 getBean（）被调用时默认实现实例化bean。


ApplicationContext是一个表示容器中的所有信息，元数据和bean的容器的接口。
它还扩展了BeanFactory接口，但默认实现在应用程序启动时实时实例化bean。可以针对自己的单个bean覆盖此行为。


**Q8. What is a Spring Bean?**

Spring Bean是由Spring IoC容器初始化的Java对象。

**Q9. What is the default bean scope in Spring framework?**

默认情况下，Spring Bean被初始化为单例。


**Q10. How to define the scope of a bean?**

要设置Spring Bean的范围，我们可以在XML配置文件中使用@Scope注释或“scope”属性。有五个支持的范围：
- singleton
- prototype
- request
- session
- global-session


**Q11. Are singleton beans thread-safe?**

不，单例bean不是线程安全的，因为线程安全性是关于执行的，而单例是一种专注于创建的设计模式。线程安全性只取决于bean实现本身。

**Q12. What does the Spring bean lifecycle look like?**

首先，需要根据Java或XML bean定义实例化一个Spring bean。还可能需要执行一些初始化以使其进入可用状态。之后，当bean不再需要时，它将从IoC容器中删除

所有初始化方法的整个周期都显示在图像（源）上：

![
][1]


  [1]: http://www.baeldung.com/wp-content/uploads/2017/06/Spring-Bean-Life-Cycle.jpg
  
  
  **Q13. What is the Spring Java-Based Configuration?**
  
  它是以类型安全的方式配置基于Spring的应用程序的方法之一。这是基于XML的配置的替代方法。
  
  **Q14. Can we have multiple Spring configuration files in one project?**
  
  是的，在大型项目中，建议使用多个Spring配置来提高可维护性和模块性。
  您可以加载多个基于Java的配置文件:

``` java
  @Configuration
@Import({MainConfig.class, SchedulerConfig.class})
public class AppConfig {
```
或加载一个将包含所有其他配置的XML文件：

``` java
ApplicationContext context = new ClassPathXmlApplicationContext("spring-all.xml");
```

在这个XML文件里面你可以包含：

``` xml
<import resource="main.xml"/>
<import resource="scheduler.xml"/>
```


**Q15. What is Spring Security?**

Spring Security是Spring框架的一个单独的模块，专注于在Java应用程序中提供身份验证和授权方法。它还特别关注了了大多数常见的安全漏洞，如CSRF攻击。

要在Web应用程序中使用Spring Security，您可以开始使用简单的注释：@EnableWebSecurity。 您可以在Baeldung找到与安全有关的一系列文章。

**Q16. What is Spring Boot?**

Spring Boot是一个项目，它提供了一组预先配置的框架，以减少样板配置，以便您可以使用最小的代码来启动和运行Spring应用程序。

**Q17. Name some of the Design Patterns used in the Spring Framework?**
- **Singleton Pattern:** Singleton-scoped beans
- **Factory Pattern:** Bean Factory classes
- **Prototype Pattern:** Prototype-scoped beans
- **Adapter Pattern:** Spring Web and Spring MVC
- **Proxy Pattern:** Spring Aspect Oriented Programming support
- **Template Method Pattern:** dbcTemplate, HibernateTemplate, etc.
-  **Front Controller:** Spring MVC DispatcherServlet
- **Data Access Object:** Spring DAO support
- **Model View Controller:** Spring MVC


**Q18. How does the scope Prototype work?**

范围原型意味着每次调用Bean的实例时，Spring将创建一个新的实例并返回。这与默认单例范围不同，其中每个Spring IoC容器实例化一个对象实例。


**3. Spring MVC**

**Q19. How to Get ServletContext and ServletConfig Objects in a Spring Bean?**

你可以这么做：
- Implementing Spring-aware interfaces. The complete list is available here.
- Using @Autowired annotation on those beans:


``` java
@Autowired
ServletContext servletContext;
 
@Autowired
ServletConfig servletConfig;
```
**Q20. What is the role of the @Required annotation?**

@Required注释用于setter方法，它表示具有此注释的bean属性必须在配置时填充。否则，Spring容器将抛出一个BeanInitializationException异常。

另外@Required与@Autowired不同，因为它仅限于setter，而@Autowired不是。 @Autowired可以用于连接构造函数和字段，而@Required只检查属性是否设置。
我们来看一个例子：


``` java
public class Person {
    private String name;
  
    @Required
    public void setName(String name) {
        this.name = name;
    }
}
```
现在，需要在XML配置中设置Person bean的名称，如下所示：

``` xml
<bean id="person" class="com.baeldung.Person">
    <property name="name" value="Joe" />
</bean>
```
请注意，@Required默认情况下不支持基于Java的@Configuration类。如果您需要确保所有属性都已设置，您可以在@Bean注释方法中创建bean时执行此操作。


**Q21. What is the role of the @Autowired annotation?**

@Autowired注释可以用于通过类型注入bean的字段或方法。此注释允许Spring解析并将bean注入到bean中。
有关详细信息，请参阅本教程。


**Q22. What is the Role of the @Qualifier Annotation?**

它与@Autowired注释同时使用，以避免在存在多个bean类型的实例时出现混淆。
我们来看一个例子。我们在XML配置中声明了两个类似的bean：

``` xml
<bean id="person1" class="com.baeldung.Person" >
    <property name="name" value="Joe" />
</bean>
<bean id="person2" class="com.baeldung.Person" >
    <property name="name" value="Doe" />
</bean>
```
当我们尝试连接bean时，我们将得到一个org.springframework.beans.factory.NoSuchBeanDefinitionException。要解决它，我们需要使用@Qualifier来告诉Spring应该连接哪个bean：


**Q23. How to handle exceptions in Spring MVC environment?**

Spring MVC中有三种处理异常的方法：
- 在控制器级别使用@ExceptionHandler - 此方法具有一个主要功能 - @ExceptionHandler注释方法仅对该特定控制器有效，而不是整个应用程序的全局
- 使用HandlerExceptionResolver - 这将解决应用程序抛出的任何异常
- 使用@ControllerAdvice - Spring 3.2为带有@ControllerAdvice注释的全局@ExceptionHandler提供了支持，该注释启用了一种从旧的MVC模型中脱离出来的机制，并利用ResponseEntity以及@ExceptionHandler的类型安全性和灵活性

**Q24. How to validate if the bean was initialized using valid values?**

Spring支持基于JSR-303注释的验证。 JSR-303是用于bean验证的Java API的规范，它是JavaEE和JavaSE的一部分，它确保bean的属性符合特定条件，使用注释（如@NotNull，@Min和@Max）。关于JSR-303的文章可在这里找到。
此外，Spring提供了用于创建自定义验证器的验证器界面。例如，你可以看看这里。


**Q25. What is Spring MVC Interceptor and how to use it?**

Spring MVC拦截器允许我们拦截一个客户端请求并在处理，处理或完成之后（视图呈现时）处理三个请求。

拦截器可用于交叉切换问题，并避免重复的处理程序代码，如日志记录，更改Spring模型中的全局使用参数等。

有关详细信息和各种实现，请查看本系列。


**Q26. What is a Controller in Spring MVC?**

简单地说，DispatcherServlet处理的所有请求都被定向到用@Controller注释的类。每个控制器类将一个或多个请求映射到使用提供的输入来处理和执行请求的方法。
如果您需要退后一步，我们建议您在典型的Spring MVC架构中查看Front Controller的概念。


**4. Spring Web**

**Q27. How does the @RequestMapping annotation work?**

@RequestMapping注释用于将Web请求映射到Spring Controller方法。除了简单的用例之外，我们还可以使用它来映射HTTP头，将URI的部分绑定到@PathVariable，并使用URI参数和@RequestParam注释。

**Q28. What’s the Difference Between @Controller, @Component, @Repository, and @Service Annotations in Spring?**

根据官方的Spring文档，@Component是任何Spring管理的组件的通用构造型。 @Repository，@Service和@Controller分别是针对更多特定用例的@Component的特殊化，例如，在持久性，服务和表示层中。
- **@Controller** - 表示该类用于控制器的角色，并在类中检测@RequestMapping注释
- **@Service** - 表示该类保存业务逻辑，并在存储库层中调用方法
- **@Repository** - 表示该类定义了数据存储库;它的任务是捕获特定于平台的异常，并将其重新抛弃为Spring统一的未经检查的异常之一


**Q29. What are DispatcherServlet and ContextLoaderListener?**

简单地说，在前端控制器设计模式中，单个控制器负责将传入的HttpRequest引导到所有应用程序的其他控制器和处理程序。

Spring的DispatcherServlet实现了这种模式，因此负责正确地协调HttpRequests到正确的处理程序。

另一方面，ContextLoaderListener启动并关闭Spring的根WebApplicationContext。它将ApplicationContext的生命周期与ServletContext的生命周期联系起来。我们可以使用它来定义在不同的Spring上下文中工作的共享bean。

**Q30. What is ViewResolver in Spring?**

ViewResolver使应用程序能够在浏览器中呈现模型，而不将实现与特定的视图技术相结合 - 通过将视图名称映射到实际视图。

**Q31. What is a MultipartResolver and when is it used?**

MultipartResolver接口用于上传文件。 Spring框架提供了一个MultipartResolver实现，用于Commons FileUpload，另一个用于Servlet 3.0 multipart请求解析。 使用这些，我们可以在我们的Web应用程序中支持文件上传。

**5. Spring Data Access**

**Q32. What is Spring JDBCTemplate class and how to use it?**

Spring JDBC模板是主要API，通过它可以访问我们感兴趣的数据库操作逻辑：
- creation and closing of connections
- executing statements and stored procedure calls
- iterating over the ResultSet and returning results

要使用它，我们需要定义DataSource的简单配置：

``` java
@Configuration
@ComponentScan("org.baeldung.jdbc")
public class SpringJdbcConfig {
    @Bean
    public DataSource mysqlDataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/springjdbc");
        dataSource.setUsername("guest_user");
        dataSource.setPassword("guest_password");
  
        return dataSource;
    }
}
```


**Q33. How would you enable transactions in Spring and what are their benefits?**

配置事务有两种不同的方式 - 注释或使用面向对象编程（AOP） - 每个都有其优点。

根据官方文档，使用Spring Transactions的好处是：
- 在不同的事务API（如JTA，JDBC，Hibernate，JPA和JDO）之间提供一致的编程模型
- 支持声明式事务管理
- 为程序化事务管理提供比一些复杂的事务API（如JTA）更简单的API
- 与Spring的各种数据访问抽象集成很好

**Q34. What is Spring DAO?**

Spring数据访问对象是Spring提供的支持，以一致和简单的方式处理JDBC，Hibernate和JPA等数据访问技术。


**6. Spring Aspect-Oriented Programming (AOP)**

通过为已经存在的代码添加额外的行为而不修改受影响的类，方面可以实现横向关注的模块化，例如跨多个类型和对象的事务管理。
以下是基于方面的执行时间记录的示例

**Q36. What are Aspect, Advice, Pointcut, and JoinPoint in AOP?**

- **Aspect** 一个类实现横切关注点,如事务管理
- **Advice** 在应用程序中达到具有匹配Pointcut的特定JoinPoint时执行的方法
- **Pointcut** 一组与JoinPoint匹配的正则表达式，以确定是否需要执行Advice
- **JoinPoint** 在执行程序期间的一点，例如执行方法或处理异常


**Q37. What is Weaving?**
根据官方文档,织入是一个将切面与其他应用程序类型或对象链接以创建建议的对象的过程。可以在编译时,加载时间,或者在运行时。Spring AOP,像其他纯Java的AOP框架,在运行时执行织入。


**6. Conclusion**
在这篇广泛的文章中，我们探讨了关于Spring的一些技术面试的一些最重要的问题。
我们希望这篇文章能够在即将到来的Spring面试中帮助您。祝你好运！