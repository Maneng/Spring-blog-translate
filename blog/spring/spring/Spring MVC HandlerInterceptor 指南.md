# Spring MVC HandlerInterceptor 指南
**目录**  

- [Spring MVC HandlerInterceptor 指南](#spring-mvc-handlerinterceptor-%E6%8C%87%E5%8D%97)
  - [**1. Introduction**](#1-introduction)
  - [**2. Spring MVC Handler**](#2-spring-mvc-handler)
  - [**3. Maven Dependencies**](#3-maven-dependencies)
  - [**4. Spring Handler Interceptor**](#4-spring-handler-interceptor)
  - [**5. Custom Logger Interceptor**](#5-custom-logger-interceptor)
  - [**5.1. Method preHandle()**](#51-method-prehandle)
  - [**5.2. Method postHandle()**](#52-method-posthandle)
  - [**5.3. Method afterCompletion()**](#53-method-aftercompletion)
  - [**6. Configuration**](#6-configuration)
  - [**7. Conclusion**](#7-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## **1. Introduction**

在本教程中，我们将专注于了解Spring MVC HandlerInterceptor以及如何正确使用它。

## **2. Spring MVC Handler**

为了了解拦截器，让我们退一步，看看HandlerMapping。这将一个方法映射到URL，以便DispatcherServlet在处理请求时能够调用它。

DispatcherServlet使用HandlerAdapter来真正的调用该方法。

现在我们了解整体上下文 - 这就是处理程序拦截器所在的位置。我们将在处理之前使用HandlerInterceptor，在处理完成之后或完成后（当视图呈现时）执行操作。

拦截器可用于交叉切换的问题，并避免重复的处理程序代码，如：日志记录，在Spring模型中更改全局使用的参数。

在接下来的几节中，正是我们将要看的 - 各种拦截器实现之间的差异。

## **3. Maven Dependencies**

为了使用Interceptor，您需要在pom.xml文件的依赖项部分中包含以下部分：

``` xml-html
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>4.3.2.RELEASE</version>
</dependency>
```

## **4. Spring Handler Interceptor**

在框架上使用HandlerMapping的拦截器必须实现HandlerInterceptor接口。

此接口包含三个主要方法：
- prehandle（） - 在执行实际处理程序之前调用，但视图尚未生成
- postHandle（） - 执行处理程序后调用
- afterCompletion（） - 在完成请求完成并生成视图后调用

这三种方法提供了各种前后处理的灵活性。

一个简单的说明 - HandlerInterceptor和HandlerInterceptorAdapter之间的主要区别在于，在第一个方面，我们需要覆盖所有三种方法：preHandle（），postHandle（）和afterCompletion（），而在第二种情况下，我们可以只实现自己需要的方法。

以下是一个简单的preHandle（）实现：

``` java
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object handler) throws Exception {
    // your code
    return true;
}
```

注意，该方法返回一个布尔值 - 它告诉Spring是否应该由处理程序（true）或（false）进一步处理该请求。


接下来，我们有一个postHandle（）的实现：

``` java
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView modelAndView) throws Exception {
    // your code
}
```

在HandlerAdapter处理请求之后，在生成视图之前，立即调用此方法。

它当然可以以许多方式使用 - 例如，我们可以将一个记录的用户的头像添加到模型中。 我们需要在自定义HandlerInterceptor实现中实现的最终方法是afterCompletion（）：


``` java
@Override
public void afterCompletion(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, Exception ex) {
    // your code
}
```

当视图成功生成时，我们可以使用此钩子来处理与请求相关的附加统计信息。


最后要注意的是，HandlerInterceptor注册到DefaultAnnotationHandlerMapping bean，该bean负责将拦截器应用于任何标有@Controller注释的类。此外，您可以在Web应用程序中指定任意数量的拦截器。

## **5. Custom Logger Interceptor**

在这个例子中，我们将专注于登录我们的Web应用程序。首先，我们的类需要扩展HandlerInterceptorAdapter：

``` java
public class LoggerInterceptor extends HandlerInterceptorAdapter {
    ...
}
```

我们还需要在拦截器中启用日志记录：

``` java
private static Logger log = LoggerFactory.getLogger(LoggerInterceptor.class);
```

这允许Log4J显示日志，以及指示当前哪个类将信息记录到指定的输出。

接下来，我们关注自定义拦截器实现：

## **5.1. Method preHandle()**

``` java
@Override
public boolean preHandle(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object handler) throws Exception {
     
    log.info("[preHandle][" + request + "]" + "[" + request.getMethod()
      + "]" + request.getRequestURI() + getParameters(request));
     
    return true;
}
```


我们可以看到，我们正在记录有关请求的一些基本信息。

如果我们在这里遇到密码，我们需要确保我们不记录。

一个简单的选择是用*号替换密码和任何其他敏感类型的数据。 这是一个快速实现如何做到这一点：

这是一个快速实现如何做到这一点：

``` java
private String getParameters(HttpServletRequest request) {
    StringBuffer posted = new StringBuffer();
    Enumeration<?> e = request.getParameterNames();
    if (e != null) {
        posted.append("?");
    }
    while (e.hasMoreElements()) {
        if (posted.length() > 1) {
            posted.append("&");
        }
        String curr = (String) e.nextElement();
        posted.append(curr + "=");
        if (curr.contains("password") 
          || curr.contains("pass")
          || curr.contains("pwd")) {
            posted.append("*****");
        } else {
            posted.append(request.getParameter(curr));
        }
    }
    String ip = request.getHeader("X-FORWARDED-FOR");
    String ipAddr = (ip == null) ? getRemoteAddr(request) : ip;
    if (ipAddr!=null && !ipAddr.equals("")) {
        posted.append("&_psip=" + ipAddr); 
    }
    return posted.toString();
}
```


最后，我们的目标是获取HTTP请求的源IP地址。 这是一个简单的实现：


``` java
private String getRemoteAddr(HttpServletRequest request) {
    String ipFromHeader = request.getHeader("X-FORWARDED-FOR");
    if (ipFromHeader != null && ipFromHeader.length() > 0) {
        log.debug("ip from proxy - X-FORWARDED-FOR : " + ipFromHeader);
        return ipFromHeader;
    }
    return request.getRemoteAddr();
}
```
## **5.2. Method postHandle()**

当HandlerAdapter被调用处理程序但DispatcherServlet尚未呈现视图时，此钩子将运行。 我们可以使用此方法向ModelAndView添加附加属性，或者确定处理方法处理客户端请求所花费的时间。

在我们的例子中，我们只需要在DispatcherServlet渲染视图之前记录一个请求。


``` java
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView modelAndView) throws Exception {
     
    log.info("[postHandle][" + request + "]");
}
```


## **5.3. Method afterCompletion()**

当请求完成并且呈现视图时，我们可以获得请求和响应数据以及有关异常的信息（如果有）：


``` java
@Override
public void afterCompletion(
  HttpServletRequest request, HttpServletResponse response,Object handler, Exception ex) 
  throws Exception {
    if (ex != null){
        ex.printStackTrace();
    }
    log.info("[afterCompletion][" + request + "][exception: " + ex + "]");
}
```
## **6. Configuration**


要将拦截器添加到Spring配置中，我们需要覆盖扩展WebMvcConfigurerAdapter的WebConfig类中的addInterceptors（）方法:

``` java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoggerInterceptor());
}
```
我们可以通过编辑我们的XML Spring配置文件来实现相同的配置：

``` java
<mvc:interceptors>
    <bean id="loggerInterceptor" class="org.baeldung.web.interceptor.LoggerInterceptor"/>
</mvc:interceptors>
```

在此配置为活动状态下，拦截器将处于活动状态，并且应用程序中的所有请求都将被正确记录。

请注意，如果配置了多个Spring拦截器，则以配置顺序执行preHandle（）方法，而以相反的顺序调用postHandle（）和afterCompletion（）方法。


## **7. Conclusion**

本教程是使用Spring MVC Handler Interceptor拦截HTTP请求的快速介绍。 所有示例和配置可在GitHub上获得。