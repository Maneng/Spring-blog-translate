

## **1. Introduction**

在本文中，我们将重点介绍Spring MVC - 控制器中的核心概念。


## **2. Overview**

我们先来看一下典型的Spring Model View Controller架构中Front Controller的概念。

在高的层次上，我们正在研究的主要职责是：
- 拦截传入请求
- 将请求的有效内容转换为数据的内部结构
- 将数据发送到模型进行进一步处理
- 从模型获取处理后的数据，并将该数据提升到视图进行渲染

以下是Spring MVC中高层次流程图：

![
][1]


  
  您可以看到，DispatcherServlet在架构中扮演前端控制器的角色。
  
  该图适用于典型的MVC控制器以及RESTful控制器 - 具有一些小的差异（如下所述）。
  
  在传统的方法中，MVC应用程序不是面向服务的，因此有一个View Resolver，它根据从Controller获得的数据呈现最终的视图。
  
  RESTful应用程序旨在面向服务，并返回原始数据（通常为JSON / XML）。由于这些应用程序不执行任何视图呈现，因此没有View Resolvers - 通常希望Controller通过HTTP响应直接发送数据。
  
  让我们从MVC0风格的控制器开始吧。
  
  ## **3. Maven Dependencies**
  
  为了能够与Spring MVC一起使用，我们先来处理Maven依赖项：
  

``` vbscript-html
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.1.RELEASE</version>
<dependency>
```

要获取最新版本的库，请查看Maven Central上的spring-webmvc。

## **4. Project Web Config**

现在，在查看控制器本身之前，我们首先需要设置一个简单的Web项目并执行一个快速的Servlet配置。

我们首先看看如何在不使用web.xml的情况下设置DispatcherServlet - 而是使用初始化程序：


``` java
public class StudentControllerConfig implements WebApplicationInitializer {
 
    @Override
    public void onStartup(ServletContext sc) throws ServletException {
        AnnotationConfigWebApplicationContext root = 
          new AnnotationConfigWebApplicationContext();
        root.register(WebConfig.class);
 
        root.refresh();
        root.setServletContext(sc);
 
        sc.addListener(new ContextLoaderListener(root));
 
        DispatcherServlet dv = 
          new DispatcherServlet(new GenericWebApplicationContext());
 
        ServletRegistration.Dynamic appServlet = sc.addServlet("test-mvc", dv);
        appServlet.setLoadOnStartup(1);
        appServlet.addMapping("/test/*");
    }
}
```
要设置没有XML的东西，请确保在您的类路径中具有servlet-api 3.1.0。 web.xml将如下所示：

web.xml将如下所示：

``` vbscript-html
<servlet>
    <servlet-name>test-mvc</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/test-mvc.xml</param-value>
    </init-param>
</servlet>
```

我们正在设置contextConfigLocation属性，指向用于加载Spring上下文的XML文件。如果该属性不存在，Spring将搜索名为{servlet_name} -servlet.xml的文件。

在我们的例子中，servlet_name是test-mvc，所以在这个例子中，DispatcherServlet会搜索一个名为test-mvc-servlet.xml的文件。

最后，我们设置DispatcherServlet并将其映射到特定的URL - 以完成我们的基于Front Controller的系统：

``` gherkin
<servlet-mapping>
    <servlet-name>test-mvc</servlet-name>
    <url-pattern>/test/*</url-pattern>
</servlet-mapping>
```

因此在这种情况下，DispatcherServlet将拦截pattern / test中的所有请求。

## **5. Spring MVC Web Config**

现在让我们看看如何使用Spring Config来设置Dispatcher Servlet：

``` java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages= {
  "org.baeldung.controller.controller",
  "org.baeldung.controller.config" }) 
public class WebConfig extends WebMvcConfigurerAdapter {
     
    @Override
    public void configureDefaultServletHandling(
      DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
  
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver bean = 
          new InternalResourceViewResolver();
        bean.setPrefix("/WEB-INF/");
        bean.setSuffix(".jsp");
        return bean;
    }
}
```

现在我们来看看使用XML设置Dispatcher Servlet。 DispatcherServlet XML文件的快照 - DispatcherServlet用于加载自定义控制器和其他Spring实体的XML文件如下所示：



``` vbscript-html
<context:component-scan base-package="com.baledung.controller" />
<mvc:annotation-driven />
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix">
        <value>/WEB-INF/</value>
    </property>
    <property name="suffix">
        <value>.jsp</value>
    </property>
</bean>
```


基于这种简单的配置，框架当然会初始化它将在类路径中找到的任何控制器bean。

请注意，我们还定义了View Resolver，负责视图呈现 - 我们将在这里使用Spring的InternalResourceViewResolver。这意味着要解析的视图的名称，这意味着通过使用前缀和后缀（两者都在XML配置中定义）来找到相应的页面。


因此，例如，如果Controller返回一个名为“welcome”的视图，则视图解析器将尝试解析WEB-INF文件夹中名为“welcome.jsp”的页面。



## **6. The MVC Controller**

让我们最后不实现MVC风格的控制器。

注意我们如何返回一个ModelAndView对象 - 其中包含模型映射和视图对象; View Resolver将使用这两种方式进行数据呈现：


``` java
@Controller
@RequestMapping(value = "/test")
public class TestController {
 
    @GetMapping
    public ModelAndView getTestData() {
        ModelAndView mv = new ModelAndView();
        mv.setViewName("welcome");
        mv.getModel().put("data", "Welcome home man");
 
        return mv;
    }
}
```

那么，我们在这里设置了什么呢？

首先，我们创建了一个名为TestController的控制器，并将其映射到“/ test”路径。在类中，我们创建了一个返回ModelAndView对象并映射到GET请求的方法，因此以“test”结尾的任何URL调用将由DispatcherServlet路由到TestController中的getTestData方法


当然，我们返回的ModelAndView对象有一些模型数据很好的措施。

视图对象的名称设置为“welcome”。如上所述，View Resolver将在WEB-INF文件夹中搜索名为“welcome.jsp”的页面。


下面你可以看到GET操作的结果：


![
][2]
请注意，URL以“test”结尾。 URL的模式是“/ test / test”。 第一个“/ test”来自Servlet，第二个来自控制器的映射。



## **7. More Spring Dependencies for REST**

现在开始看一个RESTful控制器。当然，一个好的开始是我们需要的额外的Maven依赖关系：


``` vbscript-html
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>4.3.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>4.3.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.8.0</version>
    </dependency>
</dependencies>
```


有关这些依赖关系的最新版本，请参考jackson-core，spring-webmvc和spring-web链接。


Jackson  当然不是强制性的，但它肯定是启用JSON支持的好方法。如果您有兴趣深入了解该支持，请看这里的消息转换器文章。



## **8. The REST Controller**

Spring RESTful应用程序的设置与MVC应用程序的设置相同，唯一的区别是没有View Resolver，没有模型映射。


API通常会简单地将原始数据返回给客户端 - 通常使用XML和JSON表示形式，因此DispatcherServlet绕过视图解析器，并在HTTP响应正文中返回数据。




我们来看看一个简单的RESTful控制器实现：


``` java
@Controller
public class RestController {
 
    @GetMapping(value = "/student/{studentId}")
    public @ResponseBody Student getTestData(@PathVariable Integer studentId) {
        Student student = new Student();
        student.setName("Peter");
        student.setId(studentId);
 
        return student;
    }
}
```


请注意该方法上的@ResponseBody注释 - 它指示Spring绕过视图解析器，并基本上将输出直接写入HTTP响应的正文。

![
][3]


上述输出是将GET请求发送到学生ID为1的API的结果。 这里一个简单的说法是 - @RequestMapping注释是这些中心注释之一，您将需要深入研究才能充分发挥其潜力。



## **9. Spring Boot and the @RestController Annotation**

来自Spring Boot的@RestController注释基本上是一个快捷的快捷方式，可以帮助我们避免一定要定义@ResponseBody

现在我们使用@RestController来写先前的例子，现在像这样：


``` java
@RestController
public class RestAnnotatedController {
    @GetMapping(value = "/annotated/student/{studentId}")
    public Student getData(@PathVariable Integer studentId) {
        Student student = new Student();
        student.setName("Peter");
        student.setId(studentId);
 
        return student;
    }
}
```

## **10. Conclusion**

在本指南中，我们将从Spring中了解到使用控制器的基础知识，无论是从典型的MVC应用程序的角度还是RESTful API。


  [1]: http://www.baeldung.com/wp-content/uploads/2016/08/SpringMVC.png
  [2]: http://www.baeldung.com/wp-content/uploads/2016/08/result_final.png
  [3]: http://www.baeldung.com/wp-content/uploads/2016/08/16th_july-3.png