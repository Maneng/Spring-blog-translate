## **1. Overview**


在本文中，我们将重点介绍Spring MVC - @RequestMapping中的一个主要注释。 简单地说，注释用于将Web请求映射到Spring Controller方法。

## **2. @RequestMapping Basics**

我们从一个简单的例子开始 - 将HTTP请求映射到使用一些基本条件的方法。

###  **2.1. @RequestMapping – by Path**

``` java
@RequestMapping(value = "/ex/foos", method = RequestMethod.GET)
@ResponseBody
public String getFoosBySimplePath() {
    return "Get some Foos";
}
```

要使用简单的curl命令测试此映射，请运行：

curl -i http://localhost:8080/spring-rest/ex/foos


### **2.2. @RequestMapping – the HTTP Method**

HTTP方法参数没有默认值 - 所以如果我们没有指定值，它将映射到任何HTTP请求。

这是一个简单的例子，类似于上一个例子 - 但这次映射到一个HTTP POST请求：

``` java
@RequestMapping(value = "/ex/foos", method = POST)
@ResponseBody
public String postFoos() {
    return "Post some Foos";
}
```

通过curl命令测试POST：

``` stylus
curl -i -X POST http://localhost:8080/spring-rest/ex/foos
```

## **3. RequestMapping and HTTP Headers**

### **3.1. @RequestMapping with the headers Attribute**


可以通过指定请求的标题来进一步缩小映射：

``` java
@RequestMapping(value = "/ex/foos", headers = "key=val", method = GET)
@ResponseBody
public String getFoosWithHeader() {
    return "Get some Foos with Header";
}
```


甚至通过@RequestMapping的头属性的多个头文件：

``` java
@RequestMapping(
  value = "/ex/foos", 
  headers = { "key1=val1", "key2=val2" }, method = GET)
@ResponseBody
public String getFoosWithHeaders() {
    return "Get some Foos with Header";
}
```

要测试操作，我们将使用curl header 支持：

``` stylus
curl -i -H "key:val" http://localhost:8080/spring-rest/ex/foos
```

请注意，对于用于分离头文件和头值的语法是一个冒号，与HTTP规范相同，而在Spring中则使用等号。


###  **3.2. @RequestMapping Consumes and Produces**

映射由控制器方法生成的媒体类型值得特别注意 - 我们可以通过上面介绍的@RequestMapping头部属性来映射基于其Accept头部的请求：



``` java
@RequestMapping(
  value = "/ex/foos", 
  method = GET, 
  headers = "Accept=application/json")
@ResponseBody
public String getFoosAsJsonFromBrowser() {
    return "Get some Foos with Header Old";
}
```

用于定义Accept标头的这种匹配是灵活的 - 它使用contains而不是equals，因此下面的请求仍然会正确映射：


``` stata
curl -H "Accept:application/json,text/html"
  http://localhost:8080/spring-rest/ex/foos
```

从Spring 3.1开始，@RequestMapping注释现在具有生产和消耗属性，特别是为此目的：

``` java
@RequestMapping(
  value = "/ex/foos", 
  method = RequestMethod.GET, 
  produces = "application/json"
)
@ResponseBody
public String getFoosAsJsonFromREST() {
    return "Get some Foos with Header New";
}
```
此外，具有headers属性的旧类型的映射将自动从Spring 3.1开始转换为新的生成机制，因此结果将相同。

相同的方式consumed ：

``` stata
curl -H "Accept:application/json"
  http://localhost:8080/spring-rest/ex/foos
```

此外，还生成支持多个值：

``` java
@RequestMapping(
  value = "/ex/foos", 
  method = GET,
  produces = { "application/json", "application/xml" }
)
```

请记住，这些 - 旧的方式和指定accept标头的新方法基本上是一样的映射，所以Spring不会允许它们在一起使用 - 将这两个方法都激活会导致：


``` java
Caused by: java.lang.IllegalStateException: Ambiguous mapping found. 
Cannot map 'fooController' bean method 
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromREST()
to 
{ [/ex/foos],
  methods=[GET],params=[],headers=[],
  consumes=[],produces=[application/json],custom=[]
}: 
There is already 'fooController' bean method
java.lang.String 
org.baeldung.spring.web.controller
  .FooController.getFoosAsJsonFromBrowser() 
mapped.
```


关于新产品和消费机制的最后一个注释 - 这些行为与大多数其他注释的行为不同：当在类型级别指定时，方法级注释不补充，而是覆盖类型级别信息。


## **4. RequestMapping with Path Variables**


映射URI的一部分可以通过@PathVariable注释绑定到变量。

### **4.1. Single @PathVariable**


单个路径变量的简单示例：

``` java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(
  @PathVariable("id") long id) {
    return "Get a specific Foo with id=" + id;
}
```

如果方法参数的名称与路径变量的名称完全匹配，则可以使用无值的@PathVariable来简化此操作：

``` java
@RequestMapping(value = "/ex/foos/{id}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariable(
  @PathVariable String id) {
    return "Get a specific Foo with id=" + id;
}
```

请注意，@PathVariable受益于自动类型转换，因此我们也可以将该ID声明为：

``` java
@PathVariable long id
```

### **4.2. Multiple @PathVariable**

更复杂的URI可能需要将URI的多个部分映射到多个值：

``` java
@RequestMapping(value = "/ex/foos/{fooid}/bar/{barid}", method = GET)
@ResponseBody
public String getFoosBySimplePathWithPathVariables
  (@PathVariable long fooid, @PathVariable long barid) {
    return "Get a specific Bar with id=" + barid + 
      " from a Foo with id=" + fooid;
}
```

### **4.3. @PathVariable with RegEx**

映射@PathVariable时也可以使用正则表达式;例如，我们将把映射限制为仅接受id的数值：

``` java
@RequestMapping(value = "/ex/bars/{numericId:[\\d]+}", method = GET)
@ResponseBody
public String getBarsBySimplePathWithPathVariable(
  @PathVariable long numericId) {
    return "Get a specific Bar with id=" + numericId;
}
```

这将意味着以下URI将匹配：


``` x86asm
http://localhost:8080/spring-rest/ex/bars/1
```

但这不会匹配：

``` x86asm
http://localhost:8080/spring-rest/ex/bars/abc
```


## **5. RequestMapping with Request Parameters**


@RequestMapping允许使用@RequestParam注释轻松映射URL参数。 我们现在正在将请求映射到URI，如：

``` java


http://localhost:8080/spring-rest/ex/bars?id=100
@RequestMapping(value = "/ex/bars", method = GET)
@ResponseBody
public String getBarBySimplePathWithRequestParam(
  @RequestParam("id") long id) {
    return "Get a specific Bar with id=" + id;
}
```

然后我们使用控制器方法签名中的@RequestParam（“id”）注释来提取id参数的值

使用id参数发送请求，我们将在curl中使用参数支持：

> 	curl -i -d id=100 http://localhost:8080/spring-rest/ex/bars

在这个例子中，参数是直接绑定的，而不是先被声明。
对于更高级的场景，@RequestMapping可以可选地定义参数，这也是缩小请求映射的另一种方法：

``` java
@RequestMapping(value = "/ex/bars", params = "id", method = GET)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParam(
  @RequestParam("id") long id) {
    return "Get a specific Bar with id=" + id;
}
```


允许更灵活的映射 - 可以设置多个参数值，并不需要全部使用它们：

``` java
@RequestMapping(
  value = "/ex/bars", 
  params = { "id", "second" }, 
  method = GET)
@ResponseBody
public String getBarBySimplePathWithExplicitRequestParams(
  @RequestParam("id") long id) {
    return "Narrow Get a specific Bar with id=" + id;
}
```

我们可以这样请求URI：

> http://localhost:8080/spring-rest/ex/bars?id=100&second=something

将始终映射到最佳匹配 - 这是较精细的匹配，它定义了id和second参数。



## **6. RequestMapping Corner Cases**


### **6.1. @RequestMapping – multiple paths mapped to the same controller method**


虽然单个控制器方法通常使用一个@RequestMapping路径值，但这只是一个很好的做法，而不是一个很难和快速的规则 - 有些情况可能需要将多个请求映射到同一个方法。对于这种情况，@RequestMapping的value属性确实接受多个映射，而不仅仅是一个映射：

``` java
@RequestMapping(
  value = { "/ex/advanced/bars", "/ex/advanced/foos" }, 
  method = GET)
@ResponseBody
public String getFoosOrBarsByPath() {
    return "Advanced - Get some Foos or Bars";
}
```


现在，这两个curl命令都应该使用相同的方法：


``` stylus
curl -i http://localhost:8080/spring-rest/ex/advanced/foos
curl -i http://localhost:8080/spring-rest/ex/advanced/bars
```


### **6.2. @RequestMapping – multiple HTTP request methods to the same controller method**


使用不同HTTP动词的多个请求可以映射到相同的控制器方法：


``` java
@RequestMapping(
  value = "/ex/foos/multiple", 
  method = { RequestMethod.PUT, RequestMethod.POST }
)
@ResponseBody
public String putAndPostFoos() {
    return "Advanced - PUT and POST within single method";
}
```

使用curl，这两个现在将访问同样的方法：


``` stylus
curl -i -X POST http://localhost:8080/spring-rest/ex/foos/multiple
curl -i -X PUT http://localhost:8080/spring-rest/ex/foos/multiple

```

### **6.3. @RequestMapping – a fallback for all requests**

要使用特定的HTTP方法为所有请求实现一个简单的回调 - 例如，对于GET：

``` java
@RequestMapping(value = "*", method = RequestMethod.GET)
@ResponseBody
public String getFallback() {
    return "Fallback for GET Requests";
}
```

甚至对于所有请求：


``` java
@RequestMapping(
  value = "*", 
  method = { RequestMethod.GET, RequestMethod.POST ... })
@ResponseBody
public String allFallback() {
    return "Fallback for All Requests";
}
```

## **7. New Request Mapping Shortcuts**

在Spring Framework 4.3中引入了基于HTTP方法的@RequestMapping注释：

- **@GetMapping**
- **@PostMapping**
- **@PutMapping**
- **@DeleteMapping**
- **@PatchMapping**

这些新的注释大大提高了可读性，并减少了代码的冗长度(verbosity )。我们通过创建一个支持CRUD操作的RESTful API来查看这些新的注释：

``` java
@GetMapping("/{id}")
public ResponseEntity<?> getBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id, "Bazz"+id), HttpStatus.OK);
}
 
@PostMapping
public ResponseEntity<?> newBazz(@RequestParam("name") String name){
    return new ResponseEntity<>(new Bazz("5", name), HttpStatus.OK);
}
 
@PutMapping("/{id}")
public ResponseEntity<?> updateBazz(
  @PathVariable String id,
  @RequestParam("name") String name) {
    return new ResponseEntity<>(new Bazz(id, name), HttpStatus.OK);
}
 
@DeleteMapping("/{id}")
public ResponseEntity<?> deleteBazz(@PathVariable String id){
    return new ResponseEntity<>(new Bazz(id), HttpStatus.OK);
}
```

## **8. Spring Configuration**

Spring MVC配置很简单 - 考虑到我们的FooController在以下包中定义：


``` java
package org.baeldung.spring.web.controller;
 
@Controller
public class FooController { ... }
```

我们只需要一个@Configuration类来启用完整的MVC支持，并为控制器配置类路径扫描：

``` java
@Configuration
@EnableWebMvc
@ComponentScan({ "org.baeldung.spring.web.controller" })
public class MvcConfig {
    //
}
```

## **9. Conclusion**

本文重点介绍Spring中的@RequestMapping注释 - 讨论一个简单的用例，HTTP头的映射，URI的部分绑定与@PathVariable绑定，并使用URI参数和@RequestParam注释。




