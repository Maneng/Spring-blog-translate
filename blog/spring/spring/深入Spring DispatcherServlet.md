

#  **1. Introduction**

简单地说，在前端控制器设计模式中，单个控制器负责将传入的HttpRequest引导到所有应用程序的其他控制器和处理程序。

Spring的DispatcherServlet实现了这种模式，因此负责正确地协调HttpRequests到他们的右边处理程序。

在本文中，我们将检查Spring DispatcherServlet的请求处理工作流程以及如何实现参与此工作流程的多个接口。

# **2. DispatcherServlet Request Processing**

DispatcherServlet本质上处理传入的HttpRequest，委托请求，并根据在Spring应用程序中实现的配置的HandlerAdapter接口以及指定处理程序，控制器端点和响应对象的附注进行处理。

让我们深入了解DispatcherServlet如何处理组件：
- 在DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE下关联到DispatcherServlet的WebApplicationContext被搜索并提供给进程的所有元素
- DispatcherServlet发现使用getHandler（）为您的调度程序配置的HandlerAdapter接口的所有实现 - 每个发现和配置的实现都通过handle（）方法通过该进程的其余部分来处理请求
- LocaleResolver可选地绑定到请求，以使进程中的元素能够解析该区域设置
- ThemeResolver可选地绑定到请求，让元素（如视图）确定要使用哪个主题
- 如果指定了MultipartResolver，则会为MultipartFiles检查该请求 - 任何找到的都包含在MultipartHttpServletRequest中以进一步处理
- 在WebApplicationContext中声明的HandlerExceptionResolver实现在处理请求期间提取异常

# **3. HandlerAdapter Interfaces**

HandlerAdapter接口有助于通过几个特定接口使用控制器，servlet，HttpRequests和HTTP路径。 HandlerAdapter接口因此通过DispatcherServlet请求处理工作流程的许多阶段起着至关重要的作用

首先，每个HandlerAdapter实现都从调度器的getHandler（）方法放入HandlerExecutionChain中。然后，随着执行链的进行，这些实现中的每个实现都会处理（）HttpServletRequest对象。

在以下部分中，我们将更详细地探讨一些最重要和最常用的HandlerAdapter。

## **3.1. Mappings**

要了解映射，我们需要先看看如何注释控制器，因为控制器对于HandlerMapping接口是至关重要的。

SimpleControllerHandlerAdapter允许在没有@Controller注释的情况下显式地实现控制器。

RequestMappingHandlerAdapter支持使用@RequestMapping注释注释的方法。 我们将在此处关注@Controller注释，但是也可以使用一些有用的资源，并使用SimpleControllerHandlerAdapter的几个示例。

@RequestMapping注释设置与它相关联的WebApplicationContext中处理程序可用的特定端点。

我们来看一个公开和处理'/ user / example'端点的控制器的例子：

``` java
@Controller
@RequestMapping("/user")
@ResponseBody
public class UserController {
  
    @GetMapping("/example")
    public User fetchUserExample() {
        // ...
    }
}
```

由@RequestMapping注释指定的路径通过HandlerMapping接口在内部进行管理。

RL结构自然相对于DispatcherServlet本身，并由servlet映射确定。
因此，如果DispatcherServlet映射到“/”，则所有映射将被该映射覆盖。 但是，如果servlet映射是'/ dispatcher'，那么任何@RequestMapping注释都将与该根URL相关。

记住'/'与servlet映射的'/ *'不一样！ '/'是默认映射，并将所有URL公开给调度员的责任区域。


'/ *'让许多较新的Spring开发者感到困惑。它没有指定具有相同URL上下文的所有路径都属于调度员的责任区域。相反，它会覆盖并忽略其他调度器映射。所以'/ example'将会成为404！

因此，除非非常有限的情况（如配置过滤器），否则不应使用'/ *'。


## **3.2. HTTP Request Handling**

DispatcherServlet的核心职责是将传入的HttpRequests发送到使用@Controller或@RestController注释指定的正确处理程序。

作为旁注，@Controller和@RestController的主要区别是如何生成响应 - @RestController默认定义了@ResponseBody。

在这里我们可以找到关于Spring控制器的更深入的写作。


## **3.3. The ViewResolver Interface**

ViewResolver作为ApplicationContext对象上的配置设置附加到DispatcherServlet。 ViewResolver确定调度员提供什么样的视图以及从哪里提供的视图。

以下是我们将放入WebMvcConfigurerAdapter中以呈现JSP页面的示例配置：



``` java
@Configuration
@EnableWebMvc
@ComponentScan("com.baeldung.springdispatcherservlet")
public class AppConfig extends WebMvcConfigurerAdapter {
 
    @Bean
    public UrlBasedViewResolver viewResolver() {
        UrlBasedViewResolver resolver
          = new UrlBasedViewResolver();
        resolver.setPrefix("/WEB-INF/jsp/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        return resolver;
    }
}
```

一个常见的问题是调度员的ViewResolver和整个项目目录结构的关联程度。我们来看看基础知识。

以下是使用Spring的XML配置的InternalViewResolver的示例路径配置：

``` xml
<property name="prefix" value="/jsp/"/>
```

为了我们的例子，我们假设我们的应用程序正在托管在：

``` groovy
http://localhost:8080/
```

这是本地托管的Apache Tomcat服务器的默认地址和端口。 假设我们的应用程序称为dispatcherexample-1.0.0，我们的JSP视图将可以从以下位置访问：

> http://localhost:8080/dispatcherexample-1.0.0/jsp/

在Maven的普通Spring项目中，这些视图的路径是：

``` 1c
src -|
     main -|
            java
            resources
            webapp -|
                    jsp
                    WEB-INF
```



视图的默认位置在WEB-INF中。在上面的代码段中为我们的InternalViewResolver指定的路径确定了您的视图可用的“src / main / webapp”的子目录。

## **3.6. The MultipartResolver Interface**

MultipartResolver实现检查多部分的请求，并将它们包装在MultipartHttpServletRequest中，以便在发现至少一个multipart的过程中进一步处理其他元素。添加到AppConfig：

``` java
@Bean
public CommonsMultipartResolver multipartResolver() 
  throws IOException {
    CommonsMultipartResolver resolver
      = new CommonsMultipartResolver();
    resolver.setMaxUploadSize(10000000);
    return resolver;
}
```
现在我们已经配置了MultipartResolver bean，我们设置一个控制器来处理MultipartFile请求：


``` java
@Controller
public class MultipartController {
 
    @Autowired
    ServletContext context;
 
    @PostMapping("/upload")
    public ModelAndView FileuploadController(
      @RequestParam("file") MultipartFile file) 
      throws IOException {
        ModelAndView modelAndView = new ModelAndView("index");
        InputStream in = file.getInputStream();
        String path = new File(".").getAbsolutePath();
        FileOutputStream f = new FileOutputStream(
          path.substring(0, path.length()-1)
          + "/uploads/" + file.getOriginalFilename());
        int ch;
        while ((ch = in.read()) != -1) {
            f.write(ch);
        }
        f.flush();
        f.close();
        modelAndView.getModel()
          .put("message", "File uploaded successfully!");
        return modelAndView;
    }
}
```


## **3.7. The HandlerExceptionResolver Interface**

Spring的HandlerExceptionResolver为整个Web应用程序，单个控制器或一组控制器提供统一的错误处理。

要提供应用程序范围的自定义异常处理，请创建一个使用@ControllerAdvice注释的类：

``` java
@ControllerAdvice
public class ExampleGlobalExceptionHandler {
 
    @ExceptionHandler
    @ResponseBody
    public String handleExampleException(Exception e) {
        // ...
    }
}
```


使用@ExceptionHandler注释的类中的任何方法将在调度员的责任区域内的每个控制器上都可用。


DispatcherServlet的ApplicationContext中的HandlerExceptionResolver接口的实现可用于在使用@ExceptionHandler作为注释时拦截该调度程序的责任区域下的特定控制器，并将正确的类作为参数传入：

``` java
@Controller
public class FooController{
 
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        // ...
    }
    // ...
}
```
如果发生异常CustomException1或CustomException2，handleException（）方法现在将作为上述示例中的FooController的异常处理程序。 这是一篇关于Spring Web应用程序中的异常处理的文章。