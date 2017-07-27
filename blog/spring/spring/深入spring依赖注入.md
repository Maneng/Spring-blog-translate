# Intro to Inversion Control and Dependency Injection with Spring

**目录**  

- [Intro to Inversion Control and Dependency Injection with Spring](#intro-to-inversion-control-and-dependency-injection-with-spring)
  - [**1. Overview**](#1-overview)
  - [**2. What is Inversion of Control?**](#2-what-is-inversion-of-control)
  - [**3. What is Dependency Injection?**](#3-what-is-dependency-injection)
  - [**4. The Spring IoC Container**](#4-the-spring-ioc-container)
  - [**5. Dependency Injection in Spring**](#5-dependency-injection-in-spring)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->





## **1. Overview**

在本文中，我们将介绍IoC（控制反转）和DI（依赖注入）的概念，然后我们将介绍如何在Spring框架中实现这些概念。

## **2. What is Inversion of Control?**

控制反转是软件工程中的一个原理，通过该原理将程序的对象或部分的控制转移到容器或框架。它最常用于面向对象编程的上下文中。

与传统的编程相比，我们的自定义代码调用库，IoC使框架能够控制程序的流程，并调用我们的自定义代码。为了使之成为可能，框架是通过抽象定义的，其中附加了一些行为。

**如果我们想添加自己的行为，我们需要扩展我们自己的类的框架或插件的类。**

这种架构的优点是：
- decoupling the execution of a task from its implementation 将任务的执行与其实现分离
- making it easier to switch between different implementations 使其更容易在不同的实现之间切换
- greater modularity of a program 更大的程序模块化
- greater ease in testing a program by isolating a component or mocking its dependencies and allowing components to communicate through contracts 通过隔离组件或模仿假数据其依赖性并允许组件通过约定进行通信来更容易地测试程序

可以通过各种机制实现控制反转：策略设计模式，服务定位器模式，工厂模式和依赖注入（DI）。


## **3. What is Dependency Injection?**

依赖注入是实现IoC的一种模式，其中控制被反转是设置对象的依赖关系。
将对象与其他对象连接的行为，或将对象“注入”到其他对象中，由汇编程序而不是对象本身完成。
以下是传统编程中创建对象依赖关系的方法：


``` java
public class Store {
    private Item item;
  
    public Store() {
        item = new ItemImpl1();    
    }
}
```


在上面的示例中，您可以看到，为了实例化Store对象的项依赖关系，您必须指定在Store类本身中将使用的接口Item的实现。

通过使用DI，我们可以重写示例而不指定我们想要的Item的实现：

``` java
public class Store {
    private Item item;
    public Store(Item item) {
        this.item = item;
    }
}
```


然后将通过元数据提供要注入的项目的实现，我们将在下面的示例中看到。
IoC和DI都是简单的概念，但是对我们系统构建的方式有深刻的影响，所以理解他们是非常有价值的。

## 	**4. The Spring IoC Container**

IoC容器是实现IoC的框架的常见特征。

在Spring框架中，IoC容器由ApplicationContext接口表示。 Spring容器负责实例化，配置和组装称为bean的对象，以及管理其生命周期。

Spring框架提供了ApplicationContext接口的几个实现
- 用于独立应用程序的ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，Web应用程序的WebApplicationContext。

为了组装bean，容器使用配置元数据，其可以是XML配置或注释的形式。

以下是手动实例化容器的一种方式：

``` vhdl
ApplicationContext context
  = new ClassPathXmlApplicationContext("applicationContext.xml");
```

## **5. Dependency Injection in Spring**

为了在上面的例子中设置item属性，我们可以使用将在运行时由Spring容器组装对象的元数据

Spring中的依赖注入可以通过构造函数或设置器（setters）完成。


**5.1. Constructor-Based Dependency Injection**

在基于构造函数的依赖注入的情况下，容器将调用一个构造函数，其中每个变量都表示我们要设置的依赖关系。
解析每个参数主要由参数的类型，后跟名称的属性和索引消歧。 bean的配置及其依赖关系可以使用注释来完成：


``` java
@Configuration
public class AppConfig {
 
    @Bean
    public Item item1() {
        return new ItemImpl1();
    }
 
    @Bean
    public Store store() {
        return new Store(item1());
    }
}
```
@Configuration注释表明该类是一个bean定义的源，可以在多个配置类中使用。 @Bean注释用于定义一个bean的方法。如果不指定自定义名称，将使用方法名称。

如果使用Bean的默认单例范围，则每次调用使用@Bean注释的方法时，Spring首先检查该bean的缓存实例是否已存在，并且仅在不存在的情况下创建一个新的实例。如果指定了原型范围，则为该方法的每次调用返回一个新的bean实例。

创建bean的另一种方法是通过XML配置：

``` nimrod
<bean id="item1" class="org.baeldung.store.ItemImpl1" /> 
<bean id="store" class="org.baeldung.store.Store"> 
    <constructor-arg type="ItemImpl1" index="0" name="item" ref="item1" /> 
</bean>
```

**5.2. Setter-Based Dependency Injection**

对于基于setter的DI，容器将在调用无参数构造函数或无参数静态工厂方法来实例化bean之后调用类的setter方法。让我们用注释创建这个配置：

``` java
@Bean
public Store store() {
    Store store = new Store();
    store.setItem(item1());
    return store;
}
```


我们也可以使用XML来进行相同的Bean配置：

``` xml
<bean id="store" class="org.baeldung.store.Store">
    <property name="item" ref="item1" />
</bean>
```



基于构造器和基于setter的注射类型可以组合为同一个bean。 Spring文档建议使用基于构造函数的注入用于强制依赖，以及基于setter的注入为可选的注入。


**5.3. Autowiring Dependencies**

装配允许Spring容器通过检查已定义的bean来自动解决协作bean之间的依赖关系。

使用XML配置自动连接一个bean的四种模式：
- **no**：默认值 - 这意味着没有自动连接用于bean，并且必须明确命名依赖关系
- **byName**：根据属性的名称完成自动装配，因此Spring将寻找与需要设置的属性具有相同名称的bean
- **byType**： 类似于byName的自动装配，只能基于属性的类型，这意味着Spring会寻找一个具有相同属性类型的bean来设置;如果找到多个这种类型的bean，框架会引发异常
- **constructor**：自动装配是基于构造函数参数完成的，这意味着Spring将寻找与构造函数参数相同类型的bean

例如，让我们通过类型将上面定义的item1 bean自动连接到store bean：


``` java
@Bean(autowire = Autowire.BY_TYPE)
public class Store {
     
    private Item item;
 
    public setItem(Item item){
        this.item = item;    
    }
}
```

您还可以使用@Autowired注释来自动注册bean，方法是输入以下类型：

``` java
public class Store {
     
    @Autowired
    private Item item;
}
```
如果有多个同一类型的bean，您可以添加@Qualifier注释以通过名称引用bean：

``` java
public class Store {
     
    @Autowired
    @Qualifier("item1")
    private Item item;
}
```

让我们通过XML配置按类型自动连接bean：

``` xml
<bean id="store" class="org.baeldung.store.Store" autowire="byType"> </bean>
```

现在让我们通过XML将名为item的bean注入到store bean的item属性中：

``` xml
<bean id="item" class="org.baeldung.store.ItemImpl1" />
 
<bean id="store" class="org.baeldung.store.Store" autowire="byName">
</bean>
```


还可以通过使用构造函数参数或设置器明确定义依赖关系来覆盖自动装配。


**5.4. Lazy Initialized Beans**

默认情况下，容器在初始化期间创建并配置所有单例Bean。为了避免这种情况，您可以在bean配置中指定值为true的lazy-init属性：

``` xml
<bean id="item1" class="org.baeldung.store.ItemImpl1" lazy-init="true" />
```


因此，item1 bean只有在首次被请求时才被初始化，而不是在启动时被初始化。其优点是更快的初始化时间，但是权衡之处在于，只有在请求Bean之后才能发现配置错误，这可能是在应用程序已经运行几个小时甚至几天之后。



**6. Conclusion**

在本文中，我们介绍了控制反转和依赖注入的概念，并在Spring框架中作了例证。

