# How to Get All Spring-Managed Beans?


# **1.概述**
在本文中，我们将探讨如何使用容器显示所有Spring管理的不同的bean。

# **2. The IoC Container**
一个bean是Spring管理的应用程序的基础;所有的bean都与IOC容器一起居住，IOC容器负责管理其生命周期。

我们可以通过两种方式获取此容器中所有bean的列表：

- 使用ListableBeanFactory接口
- 使用 Spring Boot Actuator

# **3. Using ListableBeanFactory Interface**
ListableBeanFactory接口提供getBeanDefinitionNames（）方法，该方法返回此工厂中定义的所有bean的名称。
该接口由预先加载其bean定义以枚举其所有bean实例的所有bean工厂实现。
您可以在官方文档中找到所有已知子接口及其实现类的列表。
对于这个例子，我们将使用一个Spring引导应用程序。
首先，我们将创建一些Spring bean。我们来创建一个简单的Spring Controller FooController：

``` java
@Controller
public class FooController {
 
    @Autowired
    private FooService fooService;
     
    @RequestMapping(value="/displayallbeans") 
    public String getHeaderAndBody(Map model){
        model.put("header", fooService.getHeader());
        model.put("message", fooService.getBody());
        return "displayallbeans";
    }
}
```
这个Controller依赖于另一个Spring bean FooService：

``` java
@Service
public class FooService {
     
    public String getHeader() {
        return "Display All Beans";
    }
     
    public String getBody() {
        return "This is a sample application that displays all beans "
          + "in Spring IoC container using ListableBeanFactory interface "
          + "and Spring Boot Actuators.";
    }
}
```
请注意，我们在这里创建了两个不同的bean：
- fooController
- fooService

在执行此应用程序时，我们将使用applicationContext对象并调用其getBeanDefinitionNames（）方法，该方法将返回我们的applicationContext容器中的所有bean：

``` java
@SpringBootApplication
public class Application {
    private static ApplicationContext applicationContext;
 
    public static void main(String[] args) {
        applicationContext = SpringApplication.run(Application.class, args);
        displayAllBeans();
    }
     
    public static void displayAllBeans() {
        String[] allBeanNames = applicationContext.getBeanDefinitionNames();
        for(String beanName : allBeanNames) {
            System.out.println(beanName);
        }
    }
}
```
这将打印来自applicationContext容器的所有bean：

``` java
fooController
fooService
//other beans
```
请注意，随着我们定义的bean，它还将记录在此容器中的所有其他bean。为了清楚起见，我们在这里省略了，因为它们有很多。



# **4. Using Spring Boot Actuator**

Spring Boot Actuator功能提供端点，用于监视我们应用程序的统计信息。

它包括许多内置的端点，包括/ beans。这将显示我们应用程序中所有Spring管理bean的完整列表。您可以在官方文档中找到现有端点的完整列表。
现在我们将在我们的application.properties中配置我们的beans端点：

``` stylus
endpoints.beans.id=springbeans
endpoints.beans.sensitive=false
```

在这里，我们正在设置bean端点的id。这个springbeans id现在将映射到一个URL，该URL将用于通过HTTP访问它。我们将敏感属性设置为false，以便我们可以在不进行身份验证的情况下访问它。如果我们只想要验证的用户查看数据，我们可以将其保留为默认值true。

现在，我们只需点击URL `http：// <address>：<management-port> / springbeans`。如果没有指定任何单独的管理端口，我们可以使用我们的默认服务器端口。这将返回显示Spring IoC容器中所有Bean的JSON响应：


``` json
[
    {
        "context": "application:8080",
        "parent": null,
        "beans": [
            {
                "bean": "fooController",
                "aliases": [],
                "scope": "singleton",
                "type": "com.baeldung.displayallbeans.controller.FooController",
                "resource": "file [E:/Workspace/tutorials-master/spring-boot/target
                  /classes/com/baeldung/displayallbeans/controller/FooController.class]",
                "dependencies": [
                    "fooService"
                ]
            },
            {
                "bean": "fooService",
                "aliases": [],
                "scope": "singleton",
                "type": "com.baeldung.displayallbeans.service.FooService",
                "resource": "file [E:/Workspace/tutorials-master/spring-boot/target/
                  classes/com/baeldung/displayallbeans/service/FooService.class]",
                "dependencies": []
            },
            // ...other beans
        ]
    }
]
```
当然，这也包含许多其他豆类，它们位于同一个spring容器中，但是为了清楚起见，我们在这里省略了它们。

# **5. Conclusion**
在本文中，我们学习了如何使用ListableBeanFactory界面和Spring Boot Actuator在Spring IoC容器中显示所有bean。