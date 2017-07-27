# Spring BeanCreationException

**目录**  

- [Spring BeanCreationException](#spring-beancreationexception)
  - [**1.概述**](#1%E6%A6%82%E8%BF%B0)
  - [**2.Cause: org.springframework.beans.factory.NoSuchBeanDefinitionException**](#2cause-orgspringframeworkbeansfactorynosuchbeandefinitionexception)
  - [**3.Cause:org.springframework.beans.factory.NoUniqueBeanDefinitionException**](#3causeorgspringframeworkbeansfactorynouniquebeandefinitionexception)
  - [**4. Cause: org.springframework.beans.BeanInstantiationException**](#4-cause-orgspringframeworkbeansbeaninstantiationexception)
    - [4.1 自定义异常](#41-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%BC%82%E5%B8%B8)
    - [4.2 java.lang.InstantiationException](#42-javalanginstantiationexception)
    - [4.3. java.lang.NoSuchMethodException](#43-javalangnosuchmethodexception)
  - [**6.org.springframework.beans.NotWritablePropertyException**](#6orgspringframeworkbeansnotwritablepropertyexception)
  - [**6.org.springframework.beans.CannotLoadBeanClassException**](#6orgspringframeworkbeanscannotloadbeanclassexception)
  - [**7.org.springframework.beans.Children of BeanCreationException**](#7orgspringframeworkbeanschildren-of-beancreationexception)
    - [7.1. The org.springframework.beans.factory.BeanCurrentlyInCreationException](#71-the-orgspringframeworkbeansfactorybeancurrentlyincreationexception)
    - [7.2. The org.springframework.beans.factory.BeanIsAbstractException](#72-the-orgspringframeworkbeansfactorybeanisabstractexception)
  - [**8.总结 …**](#8%E6%80%BB%E7%BB%93-)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## **1.概述**

在这片文章中，我们会讨论关于Spring的异常：
> Spring org.springframework.beans.factory.BeanCreationException 

当**BeanFactory**创建定义的**bean**的时候，本文中将会讨论这种常见的异常，
以及它们解决方案。

## **2.Cause: org.springframework.beans.factory.NoSuchBeanDefinitionException**

**BeanCreationException**最常见的原因是Spring试图注入上下文中不存在的bean。

例如，BeanA尝试注入BeanB：

``` java
@Component
public class BeanA {
 
    @Autowired
    private BeanB dependency;
    ...
}
```
如果在上下文中找不到BeanB，则会抛出以下异常（创建Bean时出错）：

``` oxygene
Error creating bean with name 'beanA': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private org.baeldung.web.BeanB org.baeldung.web.BeanA.dependency; 
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [org.baeldung.web.BeanB] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```
要诊断这种类型的问题 - 首先，确保bean被声明：

- 在使用`<bean />`元素的XML配置文件中
- 或通过**@Bean**注解在Java @Configuration类中
- 或者注解为：**@Component**，**@Repository**，**@Service**，**@Controller**和类路径扫描对于该包是active的

还要检查配置文件或类真正正确的由Spring获取到并加载到主上下文中。



## **3.Cause:org.springframework.beans.factory.NoUniqueBeanDefinitionException**

bean创建异常的另一个类似原因是Spring试图通过类型（即其接口）注入一个bean，并在上下文中找到两个或更多个bean来实现该接口。 例如，BeanB1和BeanB2都实现了相同的接口：


``` java
@Component
public class BeanB1 implements IBeanB { ... }
@Component
public class BeanB2 implements IBeanB { ... }
 
@Component
public class BeanA {
 
    @Autowired
    private IBeanB dependency;
    ...
}
```
这将导致Spring bean工厂抛出以下异常：


``` oxygene
Error creating bean with name 'beanA': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private org.baeldung.web.IBeanB org.baeldung.web.BeanA.b; 
nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type [org.baeldung.web.IBeanB] is defined: 
expected single matching bean but found 2: beanB1,beanB2
```


## **4. Cause: org.springframework.beans.BeanInstantiationException**


### 4.1 自定义异常

``` java
@Component
public class BeanA {
 
    public BeanA() {
        super();
        throw new NullPointerException();
    }
    ...
}
```

如预期的那样，这将导致Spring很快的失败，并且抛出异常：


``` oxygene
Error creating bean with name 'beanA' defined in file [...BeanA.class]: 
Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [org.baeldung.web.BeanA]: 
Constructor threw exception; 
nested exception is java.lang.NullPointerException
```
### 4.2 java.lang.InstantiationException

**BeanInstantiationException**的另一个可能的发生是将抽象类定义为XML中的bean;这必须在XML中，因为在Java **@Configuration**文件中没有办法这样做，而类路径扫描将忽略抽象类：


``` java
@Component
public abstract class BeanA implements IBeanA { ... }
```

bean的XML定义：

``` xml
<bean id="beanA" class="org.baeldung.web.BeanA" />
```


此设置将导致类似异常：

``` oxygene
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'beanA' defined in class path resource [beansInXml.xml]: 
Instantiation of bean failed; 
nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [org.baeldung.web.BeanA]: 
Is it an abstract class?; 
nested exception is java.lang.InstantiationException
```



### 4.3. java.lang.NoSuchMethodException
如果一个bean没有默认构造函数，并且Spring尝试通过查找该构造函数实例化它，这将导致运行时异常;例如：

``` java
@Component
public class BeanA implements IBeanA {
 
    public BeanA(final String name) {
        super();
        System.out.println(name);
    }
}
```
当这个bean被类路径扫描机制获取时，将会导致：

``` oxygene
Error creating bean with name 'beanA' defined in file [...BeanA.class]: Instantiation of bean failed; 
nested exception is org.springframework.beans.BeanInstantiationException: 
Could not instantiate bean class [org.baeldung.web.BeanA]: 
No default constructor found; 
nested exception is java.lang.NoSuchMethodException: org.baeldung.web.BeanA.<init>()
```
当类中的Spring依赖关系不具有相同的版本时，可能会出现类似异常但是更难诊断的异常是由于API更改，此类版本不兼容可能会导致**NoSuchMethodException**异常。解决这个问题的方法是确保所有的Spring库在项目中都有完全相同的版本。

## **6.org.springframework.beans.NotWritablePropertyException**

另一个可能性是定义一个bean - BeanA - 引用另一个Bean - BeanB--在BeanA中没有相应的setter方法：

``` java
@Component
public class BeanA {
    private IBeanB dependency;
    ...
}
@Component
public class BeanB implements IBeanB { ... }
```
Spring XML配置：

``` xml
<bean id="beanA" class="org.baeldung.web.BeanA">
    <property name="beanB" ref="beanB" />
</bean>
```
再次，这只能发生在XML配置中，因为在使用Java **@Configuration**时，编译器会使此问题无法再现。


当然，为了解决这个问题，需要为IBeanB添加setter：

``` java
@Component
public class BeanA {
    private IBeanB dependency;
 
    public void setDependency(final IBeanB dependency) {
        this.dependency = dependency;
    }
}
```

## **6.org.springframework.beans.CannotLoadBeanClassException**
当Spring无法加载定义的bean的类时，抛出此异常 - 如果Spring XML配置包含一个根本没有相应类的bean，则可能会发生此异常。例如，如果类BeanZ不存在，以下定义将导致异常：

``` javascript
<bean id="beanZ" class="org.baeldung.web.BeanZ" />
```

根本原因是**ClassNotFoundException**这种异常：

``` oxygene
nested exception is org.springframework.beans.factory.BeanCreationException: 
...
nested exception is org.springframework.beans.factory.CannotLoadBeanClassException: 
Cannot find class [org.baeldung.web.BeanZ] for bean with name 'beanZ'
defined in class path resource [beansInXml.xml]; 
nested exception is java.lang.ClassNotFoundException: org.baeldung.web.BeanZ
```


## **7.org.springframework.beans.Children of BeanCreationException**

### 7.1. The org.springframework.beans.factory.BeanCurrentlyInCreationException

**BeanCreationException**的子类之一是**BeanCurrentlyInCreationException**;这通常在使用构造函数注入时出现 - 例如，在循环依赖性的情况下：


``` java
@Component
public class BeanA implements IBeanA {
    private IBeanB beanB;
 
    @Autowired
    public BeanA(final IBeanB beanB) {
        super();
        this.beanB = beanB;
    }
}
@Component
public class BeanB implements IBeanB {
    final IBeanA beanA;
 
    @Autowired
    public BeanB(final IBeanA beanA) {
        super();
        this.beanA = beanA;
    }
}
```
Spring将无法解决这种情况，最终的结果将是：

``` vbnet
org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: Is there an unresolvable circular reference?
```
完整的异常是非常冗长的：

``` oxygene
org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'beanA' defined in file [...BeanA.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [org.baeldung.web.IBeanB]: : 
Error creating bean with name 'beanB' defined in file [...BeanB.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [org.baeldung.web.IBeanA]: : 
Error creating bean with name 'beanA': Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: 
Error creating bean with name 'beanB' defined in file [...BeanB.class]: 
Unsatisfied dependency expressed through constructor argument with index 0 
of type [org.baeldung.web.IBeanA]: : 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: 
Is there an unresolvable circular reference?; 
nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: 
Error creating bean with name 'beanA': 
Requested bean is currently in creation: Is there an unresolvable circular reference?
```
### 7.2. The org.springframework.beans.factory.BeanIsAbstractException

当Bean Factory尝试检索和实例化被声明为抽象的bean时，可能会发生此实例化异常。例如：

``` java
public abstract class BeanA implements IBeanA {
   ...
}
```
在XML配置中声明为：

``` scala
<bean id="beanA" abstract="true" class="org.baeldung.web.BeanA" />
```

现在，如果我们尝试通过名称从Spring上下文中检索BeanA，例如实例化另一个bean：

``` java
@Configuration
public class Config {
    @Autowired
    BeanFactory beanFactory;
 
    @Bean
    public BeanB beanB() {
        beanFactory.getBean("beanA");
        return new BeanB();
    }
}
```
这将导致以下异常：

``` oxygene
org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'beanB' defined in class path resource 
[org/baeldung/spring/config/WebConfig.class]: Instantiation of bean failed; 
nested exception is org.springframework.beans.factory.BeanDefinitionStoreException: 
Factory method 
[public org.baeldung.web.BeanB org.baeldung.spring.config.WebConfig.beanB()] threw exception; 
nested exception is org.springframework.beans.factory.BeanIsAbstractException: 
Error creating bean with name 'beanA': Bean definition is abstract
```

## **8.总结 …**

在本文末尾，我们应该有一个清晰的地图来浏览可能导致Spring中的BeanCreationException异常的原因和问题，以及如何解决所有这些问题。

在[Github](https://github.com/eugenp/tutorials/tree/master/spring-all)项目中可以找到一些这些例外示例的实现 
这是一个基于Eclipse的项目，所以应该很容易导入和运行。