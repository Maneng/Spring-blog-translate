# Spring NoSuchBeanDefinitionException

**目录**  

- [Spring NoSuchBeanDefinitionException](#spring-nosuchbeandefinitionexception)
  - [**1.概述**](#1%E6%A6%82%E8%BF%B0)
  - [**2.Cause: No qualifying bean of type […] found for dependency**](#2cause-no-qualifying-bean-of-type--found-for-dependency)
  - [**3.Cause:No qualifying bean of type […] is defined**](#3causeno-qualifying-bean-of-type--is-defined)
  - [**4. Cause: No Bean Named […] is defined**](#4-cause-no-bean-named--is-defined)
    - [4.1 自定义异常](#41-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%BC%82%E5%B8%B8)
  - [**5.Cause: Proxied Beans**](#5cause-proxied-beans)
  - [**5.总结 …**](#5%E6%80%BB%E7%BB%93-)






## **1.概述**

在这片文章中，我们会讨论关于Spring的异常：
> Spring org.springframework.beans.factory.NoSuchBeanDefinitionException  

这是BeanFactory在尝试解析一个简单地在Spring Context中未定义的bean时抛出的常见异常。
我们将说明这个问题和可用解决方案的可能原因。


## **2.Cause: No qualifying bean of type […] found for dependency**

这个异常最常见的原因是试图注入一个未定义的bean。例如 - BeanB是在一个协作者的连线--BanA：


``` java
@Component
public class BeanA {
 
    @Autowired
    private BeanB dependency;
    //...
}
```

现在，如果在Spring Context中没有定义依赖关系BeanB，引导过程将失败，并且产生**the no such bean definition exception**：


``` groovy
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [org.baeldung.packageB.BeanB]
  found for dependency: 
expected at least 1 bean which qualifies as
  autowire candidate for this dependency. 
Dependency annotations: 
  {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

Spring已经很清楚的说明了原因：“**expected at least 1 bean which qualifies as autowire candidate for this dependency**“”

BeanB在上下文中可能不存在的一个原因 - 如果通过类路径扫描自动获取bean，并且如果BeanB被正确地注解为bean（**@Component**，**@Repository**，**@Service**，**@Controller**等），那么它可能是在未被Spring扫描的包中定义：

``` java
package org.baeldung.packageB;
@Component
public class BeanB { ...}
```
然而类路径扫描可能配置如下：


``` java
@Configuration
@ComponentScan("org.baeldung.packageA")
public class ContextWithJavaConfig {
    ...
}
```

如果bean不是通过手动定义自动扫描，那么**BeanB**在当前的**Spring Context**中根本就没有定义。



## **3.Cause:No qualifying bean of type […] is defined**


异常的另一个原因是在上下文中存在两个bean定义，而不是一个。例如，如果接口 - IBeanB由两个Bean（BeanB1和BeanB2）实现：


``` java
@Component
public class BeanB1 implements IBeanB {
    //
}
@Component
public class BeanB2 implements IBeanB {
    //
}
```
现在，如果BeanA自动注入这个接口，Spring将不知道要注入的两个实现中的哪一个：

``` java
@Component
public class BeanA {
 
    @Autowired
    private IBeanB dependency;
    ...
}
```


再次，这将导致**BeanFactory**抛出**NoSuchBeanDefinitionException**

``` stylus
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type
  [org.baeldung.packageB.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```
同样，Spring明确指出了错误的原因：“希望一个bean 但是却发现了两个”。
但是请注意，在这种情况下，抛出的确切异常不是**NoSuchBeanDefinitionException**，而是一个子类 - **NoUniqueBeanDefinitionException**。

在Spring 3.2.1中引入了这个新的异常，正是由于这个原因 - 区分了没有找到bean定义和在上下文中找到几个定义。

Before this change, the exception above was:

``` stylus
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [org.baeldung.packageB.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```
这个问题的一个解决方案是使用@Qualifier注释来精确地指定要连接的bean的名称：

``` java
@Component
public class BeanA {
 
    @Autowired
    @Qualifier("beanB2")
    private IBeanB dependency;
    ...
}
```

现在，Spring有足够的信息来决定要注入哪个bean - BeanB1或BeanB2（BeanB2的默认名称是beanB2）。


## **4. Cause: No Bean Named […] is defined**


### 4.1 自定义异常
当从Spring上下文中的名称请求未定义的bean时，也可以抛出**NoSuchBeanDefinitionException**：
``` java
@Component
public class BeanA implements InitializingBean {
 
    @Autowired
    private ApplicationContext context;
 
    @Override
    public void afterPropertiesSet() {
        context.getBean("someBeanName");
    }
}
```

在这种情况下，“someBeanName”没有bean定义，导致以下异常：


``` oxygene
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No bean named 'someBeanName' is defined
```

Spring清楚简明地说明了失败的原因：“没有定义名为X的bean”。

## **5.Cause: Proxied Beans**

当使用JDK动态代理机制代理上下文中的bean时，代理将不会扩展目标bean（然而，它将实现相同的接口）。

因此，如果bean由一个接口注入，那么它将被正确地连接。但是，如果bean被实际的类注入，那么Spring将不会找到与该类匹配的bean定义 - 因为代理实际上并不是继承类

bean可能被代理的一个很常见的原因是Spring事务支持 - 即使用**@Transactional**注释的bean。

例如，如果**ServiceA**注入**ServiceB**，并且这两个服务都是事务性的，则类定义的注入将不起作用：

``` java
@Service
@Transactional
public class ServiceA implements IServiceA{
 
    @Autowired
    private ServiceB serviceB;
    ...
}
 
@Service
@Transactional
public class ServiceB implements IServiceB{
    ...
}
```

同样的两个服务，这个时候正确的注入接口就可以了：

``` java
@Service
@Transactional
public class ServiceA implements IServiceA{
 
    @Autowired
    private IServiceB serviceB;
    ...
}
 
@Service
@Transactional
public class ServiceB implements IServiceB{
    ...
}
```




## **5.总结 …**

本教程讨论了常见的NoSuchBeanDefinitionException的可能原因的示例，重点是如何在实践中解决这些异常

在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)项目中可以找到一些这些例外示例的实现 
这是一个基于Eclipse的项目，所以应该很容易导入和运行。
