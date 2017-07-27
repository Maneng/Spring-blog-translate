# 怎么自定义Spring aop注解
**目录**  

- [怎么自定义Spring aop注解](#%E6%80%8E%E4%B9%88%E8%87%AA%E5%AE%9A%E4%B9%89spring-aop%E6%B3%A8%E8%A7%A3)
  - [**1. Introduction**](#1-introduction)
  - [**2. What is an AOP Annotation?**](#2-what-is-an-aop-annotation)
  - [**3. Maven Dependency**](#3-maven-dependency)
  - [**4. Creating our Custom Annotation**](#4-creating-our-custom-annotation)
  - [**5. Creating our Aspect**](#5-creating-our-aspect)
  - [**6. Creating our Pointcut and Advice**](#6-creating-our-pointcut-and-advice)
  - [**7. Logging our Execution Time**](#7-logging-our-execution-time)
  - [**8. Conclusion**](#8-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## **1. Introduction**

在本文中，我们将使用Spring中的AOP支持来实现自定义AOP注释。

首先，我们将对AOP进行高级概述，说明它是什么以及它的优点。接下来，我们将逐步实施注释，逐步建立对AOP概念的更深入的了解。 结果将是对AOP的更好理解和将来创建我们的定制Spring注释的能力。


## **2. What is an AOP Annotation?**

要快速总结，AOP代表面向方面的编程。基本上，它是一种在不修改该代码的情况下向现有代码添加行为的方式。 有关AOP的详细介绍，有关于AOP切入点和建议的文章。本文假设我们已经有了基础知识

我们将在本文中实现的AOP的类型是注释驱动的。如果我们使用了Spring @Transactional注释，我们可能已经熟悉了这一点：

``` java
@Transactional
public void orderGoods(Order order) {
   // A series of database calls to be performed in a transaction
}
```

这里的关键是非侵略性。通过使用注解元数据，我们的核心业务逻辑不会被我们的交易代码污染。这使得更容易理解，重构和隔离测试。

有时，开发Spring应用程序的人可以将其看作是“弹性魔法”，而不必考虑如何运作。在现实中，发生的事情并不是特别复杂。但是，一旦我们完成了本文中的步骤，我们将能够创建自己的自定义注释，以了解和利用AOP。

## **3. Maven Dependency**

首先，我们添加我们的Maven依赖关系。 对于这个例子，我们将使用Spring Boot，因为它的配置方法的惯例让我们尽可能快地起床和运行：


``` vbscript-html
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
</parent>
 
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```


请注意，我们已经包括了AOP启动器，它引入了我们开始实现方面所需的库。

## **4. Creating our Custom Annotation**

我们要创建的注释是用于记录执行方法所需的时间量的注释。我们创建我们的注释：

``` java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
 
}
```


虽然相对简单的实现，值得注意的是使用两个元注释。

@Target注释告诉我们我们的注释将适用于何处。在这里我们使用ElementType.Method，这意味着它只适用于方法。如果我们试图在其他地方使用注释，那么我们的代码将无法编译。这种行为是有道理的，因为我们的注释将用于记录方法执行时间。

而@Retention只是说明注释是否在运行时可用于JVM。默认情况下不是这样，所以Spring AOP将无法看到注释。这就是为什么它被重新配置。

## **5. Creating our Aspect**

现在我们有了我们的注释，我们来创建我们的方面。这只是封装我们交叉关切的模块，我们的方法是执行时间记录。它是一个类，用@Aspect注释：

``` java
@Aspect
@Component
public class ExampleAspect {
 
}
```

## **6. Creating our Pointcut and Advice**

现在，我们来创建我们的切入点和建议。这将是一个注释的方法，它存在于我们的方面：

``` java
@Around("@annotation(LogExecutionTime)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    return joinPoint.proceed();
}
```

从技术上讲，这并没有改变任何事情的行为，但仍然有很多需求分析。

首先，我们用@Around注释了我们的方法。这是我们的建议，围绕建议意味着我们在方法执行之前和之后添加额外的代码。还有其他类型的建议，例如前后，但这些建议将被排除在本文的范围之外。

接下来，我们的@Around注释有一个切点参数。我们的切入点只是说，'应用这个建议任何方法用@LogExecutionTime注释'。还有很多其他类型的切入点，但是如果范围，它们将再次被忽略。

方法logExecutionTime（）本身就是我们的建议。有一个参数，即ProceedingJoinPoint。在我们的例子中，这将是一个已经用@LogExecutionTime注释的执行方法。

最后，当我们的注释方法最终被调用时，会发生什么，我们的建议将被首先调用。那么由我们的建议决定下一步做什么。在我们的例子中，我们的建议是除了调用proceed（）之外什么也没做，只是调用原来的注释方法。


## **7. Logging our Execution Time**

现在我们有了我们的骨架，我们需要做的就是为我们的建议添加一些额外的逻辑。除了调用原始方法之外，这将记录执行时间。让我们再补充一点：


``` java
@Around("@annotation(LogExecutionTime)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
 
    Object proceed = joinPoint.proceed();
 
    long executionTime = System.currentTimeMillis() - start;
 
    System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
    return proceed;
}
```


再次，我们没有做任何在这里特别复杂的事情。我们刚刚记录了当前的时间，执行了该方法，然后打印出控制台所需的时间。我们还记录了使用连接点实例提供的方法签名。如果我们想要，我们也可以访问其他信息位，例如方法参数。

现在，让我们尝试用@LogExecutionTime注释一个方法，然后执行它来看看会发生什么。请注意，这必须是一个Spring Bean才能正常工作：

``` java
@LogExecutionTime
public void serve() throws InterruptedException {
    Thread.sleep(2000);
}
```


执行后，我们应该看到以下记录到控制台：
> void org.baeldung.Service.serve() executed in 2030ms

## **8. Conclusion**

在本文中，我们利用Spring Boot AOP创建自定义注释，我们可以将其应用于Spring bean，以便在运行时向其注入额外的行为。


