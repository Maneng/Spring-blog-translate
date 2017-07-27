# Spring @Autowired指南

**目录**  

- [Spring @Autowired指南](#how-to-get-all-spring-managed-beans)
  - [**1.Overview**](#1overview)
  - [**2. Enabling @Autowired Annotations**](#2-enabling-autowired-annotations)
  - [**3. Using @Autowired**](#3-using-autowired)
    - [**3.1. @Autowired on Properties**](#31-autowired-on-properties)
    - [**3.2. @Autowired on Setters**](#32-autowired-on-setters)
    - [**3.3. @Autowired on Constructors**](#33-autowired-on-constructors)
  - [**4. @Autowired and Optional Dependencies**](#4-autowired-and-optional-dependencies)
  - [**5. Autowire Disambiguation**(5.自动消除歧义)](#5-autowire-disambiguation5%E8%87%AA%E5%8A%A8%E6%B6%88%E9%99%A4%E6%AD%A7%E4%B9%89)
    - [**5.1. Autowiring by @Qualifier**](#51-autowiring-by-qualifier)
    - [**5.2. Autowiring by Custom Qualifier** (自定义限定符自动装配)](#52-autowiring-by-custom-qualifier-%E8%87%AA%E5%AE%9A%E4%B9%89%E9%99%90%E5%AE%9A%E7%AC%A6%E8%87%AA%E5%8A%A8%E8%A3%85%E9%85%8D)
    - [**5.2. Autowiring by Name**](#52-autowiring-by-name)
  - [**6. Conclusion**](#6-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




##  **1.Overview**

从Spring 2.5开始，该框架引入了由@Autowired注释驱动的新型依赖注入。此注释允许Spring解析并将bean注入到bean中。

在本教程中，我们将介绍如何启用自动装配，以多种方式连接bean，使bean可选，使用@Qualifier注释解决bean冲突以及潜在的异常情况。

## **2. Enabling @Autowired Annotations**

如果您在应用程序中使用基于Java的配置，则可以通过使用AnnotationConfigApplicationContext来加载spring配置来启用注释驱动的注入，如下所示：

``` java
@Configuration
@ComponentScan("com.baeldung.autowire.sample")
public class AppConfig {}
```
或者，在Spring XML中，可以通过在Spring XML文件中声明它来实现：<context：annotation-config />

## **3. Using @Autowired**

注释注入启用后，可以在属性，设置器和构造函数上使用自动装配。


### **3.1. @Autowired on Properties**

注释可以直接用于属性，因此不需要getter和setter：

``` java
@Component("fooFormatter")
public class FooFormatter {
 
    public String format() {
        return "foo";
    }
}
```


``` java
@Component
public class FooService {
     
    @Autowired
    private FooFormatter fooFormatter;
 
}
```
在上面的例子中，Spring创建FooService时，寻找并注入fooFormatter。

### **3.2. @Autowired on Setters**

@Autowired注释可以用于setter方法。在下面的示例中，当在setter方法上使用注释时，在创建FooService时，setter方法将使用FooFormatter的实例调用：

``` java
public class FooService {
 
    private FooFormatter fooFormatter;
 
    @Autowired
    public void setFooFormatter(FooFormatter fooFormatter) {
            this.fooFormatter = fooFormatter;
    }
}
```
### **3.3. @Autowired on Constructors**

@Autowired注释也可以在构造函数上使用。在下面的示例中，当在构造函数上使用注释时，当创建FooService时，FooFormatter的实例将作为参数注入构造函数中：


``` java
public class FooService {
 
    private FooFormatter fooFormatter;
 
    @Autowired
    public FooService(FooFormatter fooFormatter) {
        this.fooFormatter = fooFormatter;
    }
}
```
## **4. @Autowired and Optional Dependencies**

Spring希望在构建依赖bean时可以使用@Autowired依赖关系。如果框架无法解决用于装配的bean，它将抛出以下引用的异常并阻止Spring容器成功启动：

``` groovy
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.autowire.sample.FooDAO] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: 
{@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

为了避免这种情况发生，可以通过以下指定bean：

``` java
public class FooService {
 
    @Autowired(required = false)
    private FooDAO dataAccessor; 
     
}
```

## **5. Autowire Disambiguation**(5.自动消除歧义)

默认情况下，Spring按类型解析@Autowired条目。如果同一类型的多个bean在容器中可用，则框架将抛出​​一个致命的异常，指示有多个bean可用于自动装配。

### **5.1. Autowiring by @Qualifier**

@Qualifier注释可用于提示和缩小所需的bean：

``` java
@Component("fooFormatter")
public class FooFormatter implements Formatter {
 
    public String format() {
        return "foo";
    }
}
@Component("barFormatter")
public class BarFormatter implements Formatter {
 
    public String format() {
        return "bar";
    }
}
public class FooService {
     
    @Autowired
    private Formatter formatter;
 
}
```

由于Spring容器可以使用两种格式化的实现，所以在构建FooService时，Spring会抛出一个NoUniqueBeanDefinitionException异常：


``` stylus
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type [com.autowire.sample.Formatter] is defined: 
expected single matching bean but found 2: barFormatter,fooFormatter
```


这可以通过使用@Qualifier注释来缩小实现来避免：

``` java
public class FooService {
     
    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
 
}
```


通过使用具体实现的名称指定@Qualifier，在这种情况下，作为fooFormatter，当Spring找到同一类型的多个bean时，我们可以避免歧义。
请注意，@Qualifier注释的值与我们的FooFormatter实现的@Component注释中声明的名称相匹配。 5.2。自定义限定符自动装配

### **5.2. Autowiring by Custom Qualifier** (自定义限定符自动装配)

Spring允许我们创建我们自己的@Qualifier注释。要创建自定义限定符，请定义注释，并在定义中提供@Qualifier注释，如下所示

``` java
@Qualifier
@Target({
  ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface FormatterType {
     
    String value();
 
}
```




一旦定义，可以在各种实现中使用FormatterType来指定自定义值：


``` java
@FormatterType("Foo")
@Component
public class FooFormatter implements Formatter {
 
    public String format() {
        return "foo";
    }
}
```


``` java
@FormatterType("Bar")
@Component
public class BarFormatter implements Formatter {
 
    public String format() {
        return "bar";
    }
}
```
一旦实现注释，自定义限定符注释可以如下使用：


``` java
@Component
public class FooService {
     
    @Autowired
    @FormatterType("Foo")
    private Formatter formatter;
     
}
```
@Target注释中指定的值限制限定词可用于标记注入点的位置。 在上面的代码片段中，限定符可以用于消除Spring可以将bean注入到字段，方法，类型和参数中的点。

### **5.2. Autowiring by Name**

作为回馈，Spring使用bean名称作为默认限定符值。

因此，通过定义bean属性名称，在这种情况下为fooFormatter，Spring与FooFormatter实现相匹配，并在构建FooService时注入该特定实现：

``` java
public class FooService {
     
    @Autowired
    private Formatter fooFormatter;
     
}
```
## **6. Conclusion**
虽然@Qualifier和bean名称的后备匹配都可以用来缩小到一个特定的bean，但是自动装配实际上是关于按类型注入的，这是最好的方式来使用这个容器功能。