# Spring @RequestMapping新的快捷注解

**目录**  

- [Spring @RequestMapping新的快捷注解](#spring-requestmapping%E6%96%B0%E7%9A%84%E5%BF%AB%E6%8D%B7%E6%B3%A8%E8%A7%A3)
  - [**1. Overview**](#1-overview)
  - [**2. New Annotations**](#2-new-annotations)
  - [**3. How It Works**](#3-how-it-works)
  - [**4. Implementation**](#4-implementation)
    - [**4.1. @GetMapping**](#41-getmapping)
    - [**4.2. @PostMapping**](#42-postmapping)
    - [**4.3. @PutMapping**](#43-putmapping)
    - [**4.4. @DeleteMapping**](#44-deletemapping)
    - [**4.5. @PatchMapping**](#45-patchmapping)
  - [**5. Testing the Application**](#5-testing-the-application)
  - [**6. Conclusion**](#6-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## **1. Overview**

spring4.3。介绍了一些非常酷的方法级组合注释，以更优雅的处理典型的Spring MVC项目中的@RequestMapping。

## **2. New Annotations**

通常，如果我们要使用传统的@RequestMapping注释来实现URL处理程序，那么它将是这样的：

> @RequestMapping(value = "/get/{id}", method = RequestMethod.GET)

新的方法可以简化为：

>@GetMapping("/get/{id}")

Spring目前支持五种类型的内置注释，用于处理GET，POST，PUT，DELETE和PATCH等不同类型的传入HTTP请求方法。这些注释是：

- **@GetMapping**
- **@PostMapping**
- **@PutMapping**
- **@DeleteMapping**
- **@PatchMapping**


从命名约定可以看出，每个注解都是为了处理各自的传入请求方法类型，即@GetMapping用于处理请求方法的GET类型，@PostMapping用于处理POST类型的请求方法等。

## **3. How It Works**

所有上述注释已经在内部使用@RequestMapping进行注释，并且在方法元素中设置了相应的值。

例如，如果我们来看看@GetMapping注解的源代码，我们可以看到它已经通过RequestMethod.GET以下列方式注释：

``` java
@Target({ java.lang.annotation.ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = { RequestMethod.GET })
public @interface GetMapping {
    // abstract codes
}
```

所有其他注释以相同的方式创建，即@PostMapping用RequestMethod.POST注释，@PutMapping用RequestMethod.PUT等注释。


## **4. Implementation**

我们尝试使用这些注释构建一个快速的REST应用程序。

请注意，由于我们将使用Maven构建项目和Spring MVC来创建我们的应用程序，所以我们需要在pom.xml中添加必要的依赖项：

``` xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.6.RELEASE</version>
</dependency>
```
现在，我们需要创建控制器来映射传入的请求URL。在这个控制器中，我们将逐个使用所有这些注释。












### **4.1. @GetMapping**


``` java
@GetMapping("/get")
public @ResponseBody ResponseEntity<String> get() {
    return new ResponseEntity<String>("GET Response", HttpStatus.OK);
}


@GetMapping("/get/{id}")
public @ResponseBody ResponseEntity<String>
  getById(@PathVariable String id) {
    return new ResponseEntity<String>("GET Response : "
      + id, HttpStatus.OK);
}
```



### **4.2. @PostMapping**

``` java
@PostMapping("/post")
public @ResponseBody ResponseEntity<String> post() {
    return new ResponseEntity<String>("POST Response", HttpStatus.OK);
}
```


### **4.3. @PutMapping**

``` java
@PutMapping("/put")
public @ResponseBody ResponseEntity<String> put() {
    return new ResponseEntity<String>("PUT Response", HttpStatus.OK);
}
```

### **4.4. @DeleteMapping**


``` java
@DeleteMapping("/delete")
public @ResponseBody ResponseEntity<String> delete() {
    return new ResponseEntity<String>("DELETE Response", HttpStatus.OK);
}
```


### **4.5. @PatchMapping**

``` java
@PatchMapping("/patch")
public @ResponseBody ResponseEntity<String> patch() {
    return new ResponseEntity<String>("PATCH Response", HttpStatus.OK);
}
```


注意事项：
- 我们已经使用必要的注释来处理适当的传入HTTP方法与URI。例如，@GetMapping处理“/ get”URI，@PostMapping处理“/ post”URI等
- 由于我们正在制作一个基于REST的应用程序，所以我们用200个响应代码返回一个常量字符串（每个请求类型唯一），以简化应用程序。在这种情况下，我们使用了Spring的@ResponseBody注释。
- 如果我们不得不处理任何URL路径变量,我们可以用少得多的方式使用@RequestMapping用来做的。



## **5. Testing the Application**

要测试应用程序，我们需要使用JUnit创建几个测试用例。我们将使用SpringJUnit4ClassRunner启动测试类。我们将创建五个不同的测试用例来测试每个注释和我们在控制器中声明的每个处理程序。

让我们来简单了解一下测试@GetMapping的例子

``` java
@Test
public void giventUrl_whenGetRequest_thenFindGetResponse() 
  throws Exception {
 
    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders
      .get("/get");
 
    ResultMatcher contentMatcher = MockMvcResultMatchers.content()
      .string("GET Response");
 
    this.mockMvc.perform(builder).andExpect(contentMatcher)
      .andExpect(MockMvcResultMatchers.status().isOk());
 
}
```
我们可以看到，一旦我们点击GET URL“/ get”，我们期待着一个恒定的字符串“GET Response”。 现在，我们创建测试用例来测试@PostMapping：

``` java
@Test
public void givenUrl_whenPostRequest_thenFindPostResponse() 
  throws Exception {
     
    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders
      .post("/post");
     
    ResultMatcher contentMatcher = MockMvcResultMatchers.content()
      .string("POST Response");
     
    this.mockMvc.perform(builder).andExpect(contentMatcher)
      .andExpect(MockMvcResultMatchers.status().isOk());
     
}
```
以同样的方式，我们创建了其余的测试用例来测试所有的HTTP方法。

或者，我们可以随时使用任何常见的REST客户端，例如PostMan，RESTClient等来测试我们的应用程序。在这种情况下，我们需要在使用其余的客户端时仔细选择正确的HTTP方法类型。否则会抛出405错误状态。

## **6. Conclusion**

在本文中，我们快速介绍了使用传统Spring MVC框架进行快速Web开发的不同类型的@RequestMapping快捷方式。我们可以利用这些快捷方式创建一个干净的代码库。