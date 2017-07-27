# **No bean named ‘springSecurityFilterChain’ is defined**

**目录**  

- [**No bean named ‘springSecurityFilterChain’ is defined**](#no-bean-named-springsecurityfilterchain-is-defined)
  - [**1. The Problem**](#1-the-problem)
  - [**2. The Cause**](#2-the-cause)
    - [**3. The Solution**](#3-the-solution)
  - [**4. Conclusion**](#4-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




## **1. The Problem**

本文讨论了Spring Security配置问题 - 应用程序引导过程抛出以下异常：

``` stylus
SEVERE: Exception starting filter springSecurityFilterChain
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No bean named 'springSecurityFilterChain' is defined
```

## **2. The Cause**

这个异常的原因很简单 - Spring Security会查找一个名为**springSecurityFilterChain**（默认情况下）的bean，但是找不到它。这个bean是由web.xml中定义的主要Spring安全过滤器（**DelegatingFilterProxy**）所必需的：

``` stylus
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
这只是将其所有逻辑委托给**springSecurityFilterChain bean**的代理。


### **3. The Solution**

上下文中缺少此bean的最常见原因是安全性XML配置没有定义`<http>`元素：


``` vbscript-html
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:sec="http://www.springframework.org/schema/security"
  xsi:schemaLocation="
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.1.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
 
</beans:beans>
```
如果XML配置正在使用安全命名空间（如上例），则声明一个简单的`<http>`元素将确保过滤器bean被创建，并且所有启动都正确：

另一个可能的原因是安全配置根本不会导入到Web应用程序的整体上下文中。

如果安全XML配置文件名为**springSecurityConfig.xml**，请确保资源已导入：

``` stylus
@ImportResource({"classpath:springSecurityConfig.xml"})
```
或在XML中

``` nix
<import resource="classpath:springSecurityConfig.xml" />
```

最后，可以在web.xml中更改过滤器bean的默认名称 - 通常使用Spring Security的现有过滤器：

``` livecodeserver
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
      org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>customFilter</param-value>
    </init-param>
</filter>
```

## **4. Conclusion**

本文讨论了一个非常具体的Spring Security问题 - 缺少的过滤器链bean - 并显示了这个常见问题的解决方案。