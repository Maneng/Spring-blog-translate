
## **1. Overview**

本文介绍如何在Spring中配置HttpMessageConverter。
简单地说，消息转换器用于通过HTTP来对Java对象进行和从JSON，XML等进行编组和解组。

## **2. The Basics**

Web应用程序需要配置Spring MVC支持 - 一个方便且非常可定制的方法是使用@EnableWebMvc注释：

``` java
@EnableWebMvc
@Configuration
@ComponentScan({ "org.baeldung.web" })
public class WebConfig extends WebMvcConfigurerAdapter {
    ...
}
```

请注意，此类扩展了WebMvcConfigurerAdapter - 这将允许我们使用我们自己的Http转换器的默认列表。

### **2.2. The Default Message Converters**

默认情况下，以下HttpMessageConverters实例已预启用：
- ByteArrayHttpMessageConverter - 字节数组转换
- StringHttpMessageConverter - 转换字符串
- ResourceHttpMessageConverter - 转换任何类型的八位字节流的org.springframework.core.io.Resource
- SourceHttpMessageConverter - 转换javax.xml.transform.Source
- FormHttpMessageConverter - 将表单数据转换为/从MultiValueMap <String，String>转换。
- Jaxb2RootElementHttpMessageConverter - 将Java对象转换为/从XML转换（仅在类路径中存在JAXB2时添加）
- MappingJackson2HttpMessageConverter - 转换JSON（仅在类路径中存在Jackson 2时才添加）
- MappingJacksonHttpMessageConverter - 转换JSON（仅当Jackson 2存在于类路径时才添加）
- AtomFeedHttpMessageConverter - 转换Atom订阅源（仅当 Rome在类路径中存在时添加）
- RssChannelHttpMessageConverter - RSS源（只加了，如果Rome存在的类路径上）转换

##  **3. Client-Server Communication – JSON only**

### **3.1. High Level Content Negotiation**

每个HttpMessageConverter实现都有一个或多个关联的MIME类型。

当接收到新的请求时，Spring将使用“Accept”头来确定需要响应的媒体类型。

然后，它将尝试找到一个能够处理该特定媒体类型的注册转换器 - 它将使用它转换实体并发回响应。

该过程类似于接收包含JSON信息的请求 - 框架将使用“Content-Type”头来确定请求体的媒体类型。

然后，它将搜索可以将客户端发送的身体转换为Java对象的HttpMessageConverter。

让我们用一个简单的例子来阐明一下：

- 客户端向/ foos发送一个GET请求，将Accept标头设置为application / json，以获取所有Foo资源为Json
- Foo Spring控制器被命中并返回相应的Foo Java实体
- 然后，Spring然后使用Jackson消息转换器之一将实体编组到json

现在来看看这个工作原理的具体细节，以及我们如何利用@ResponseBody和@RequestBody注释。


### **3.2. @ResponseBody**

Controller上的@ResponseBody方法向Spring表示方法的返回值直接序列化到HTTP响应的主体。如上所述，客户端指定的“Accept”标头将用于选择适当的Http Converter来组织实体。

我们来看一个简单的例子：

``` java
@RequestMapping(method=RequestMethod.GET, value="/foos/{id}")
public @ResponseBody Foo findById(@PathVariable long id) {
    return fooService.get(id);
}
```


现在，客户端将在request-example curl命令中指定application / json中的“Accept”标头：


``` stylus
curl --header "Accept: application/json"
  http://localhost:8080/spring-rest/foos/1
```
The Foo class:

``` java
public class Foo {
    private long id;
    private String name;
}
```

和Http响应body：


``` json
{
    "id": 1,
    "name": "Paul",
}
```
###  **3.3. @RequestBody**

@RequestBody是用于Controller方法的参数 - 它向Spring指出，HTTP请求的正文反序列化到该特定的Java实体。如前所述，由客户端指定的“Content-Type”头将用于确定适当的转换器。 我们来看一个例子：


``` java
@RequestMapping(method=RequestMethod.PUT, value="/foos/{id}")
public @ResponseBody void updateFoo(
  @RequestBody Foo foo, @PathVariable String id) {
    fooService.update(foo);
}
```

现在，让我们用JSON对象来使用它 - 我们将“Content-Type”指定为application / json：


``` stata
curl -i -X PUT -H "Content-Type: application/json" 
-d '{"id":"83","name":"klik"}' http://localhost:8080/spring-rest/foos/1
```

我们得到200 OK - 一个成功的回应：

``` http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Fri, 10 Jan 2014 11:18:54 GMT
```
## **4. Custom Converters Configuration**

我们可以通过扩展WebMvcConfigurerAdapter类并覆盖configureMessageConverters方法来自定义消息转换器：


``` java
@EnableWebMvc
@Configuration
@ComponentScan({ "org.baeldung.web" })
public class WebConfig extends WebMvcConfigurerAdapter {
 
    @Override
    public void configureMessageConverters(
      List<HttpMessageConverter<?>> converters) {
     
        messageConverters.add(createXmlHttpMessageConverter());
        messageConverters.add(new MappingJackson2HttpMessageConverter());
 
        super.configureMessageConverters(converters);
    }
    private HttpMessageConverter<Object> createXmlHttpMessageConverter() {
        MarshallingHttpMessageConverter xmlConverter = 
          new MarshallingHttpMessageConverter();
 
        XStreamMarshaller xstreamMarshaller = new XStreamMarshaller();
        xmlConverter.setMarshaller(xstreamMarshaller);
        xmlConverter.setUnmarshaller(xstreamMarshaller);
 
        return xmlConverter;
    }
}
```


这里是相应的XML配置：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/context 
      http://www.springframework.org/schema/context/spring-context.xsd 
    http://www.springframework.org/schema/mvc 
      http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
 
    <context:component-scan base-package="org.baeldung.web" />
 
    <mvc:annotation-driven>
        <mvc:message-converters>
           <bean
             class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
              
           <bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
               <property name="marshaller" ref="xstreamMarshaller" />
               <property name="unmarshaller" ref="xstreamMarshaller" />
           </bean> 
        </mvc:message-converters>
    </mvc:annotation-driven>
     
    <bean id="xstreamMarshaller" class="org.springframework.oxm.xstream.XStreamMarshaller" />
 
</beans>
```
请注意，XStream库现在需要存在于类路径中。

还要注意，通过扩展这个支持类，我们正在丢失以前预先注册的默认消息转换器 - 我们只有我们定义的。

我们来看一下这个例子 - 我们正在创建一个新的转换器--MarshallingHttpMessageConverter，我们使用Spring XStream支持来配置它。因为我们使用底层编组框架的低级API（在这种情况下是XStream），因此我们可以非常灵活地进行配置，但是我们可以配置这些。

我们当然可以为Jackson做同样的事情 - 通过定义我们自己的MappingJackson2HttpMessageConverter，我们现在可以在此转换器上设置一个自定义的ObjectMapper，并根据需要进行配置。

在这种情况下，XStream是选定的编组器/解组器实现，但是可以使用其他类似CastorMarshaller的方法来引用Spring api文档，以获取可用编组器的完整列表。 在这一点上 - 在后端启用了XML - 我们可以使用XML表示形式使用API​​：

``` stylus
curl --header "Accept: application/xml"
  http://localhost:8080/spring-rest/foos/1
```

## **5. Using Spring’s RestTemplate with Http Message Converters**


除了服务器端，Http消息转换可以在客户端在Spring RestTemplate上进行配置。

我们将在适当的时候使用“Accept”和“Content-Type”标头来配置模板，我们将尝试使用完整的编组和解组Foo资源来消费REST API，包括JSON和XML 。

### **5.1. Retrieving the Resource with no Accept Header**

``` java
@Test
public void testGetFoo() {
    String URI = “http://localhost:8080/spring-rest/foos/{id}";
    RestTemplate restTemplate = new RestTemplate();
    Foo foo = restTemplate.getForObject(URI, Foo.class, "1");
    Assert.assertEquals(new Integer(1), foo.getId());
}
```
### **5.3. Retrieving a Resource with application/json Accept header**

``` java
@Test
public void givenConsumingJson_whenReadingTheFoo_thenCorrect() {
    String URI = BASE_URI + "foos/{id}";
 
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.setMessageConverters(getMessageConverters());
 
    HttpHeaders headers = new HttpHeaders();
    headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
    HttpEntity<String> entity = new HttpEntity<String>(headers);
 
    ResponseEntity<Foo> response = 
      restTemplate.exchange(URI, HttpMethod.GET, entity, Foo.class, "1");
    Foo resource = response.getBody();
 
    assertThat(resource, notNullValue());
}
private List<HttpMessageConverter<?>> getMessageConverters() {
    List<HttpMessageConverter<?>> converters = 
      new ArrayList<HttpMessageConverter<?>>();
    converters.add(new MappingJackson2HttpMessageConverter());
    return converters;
}
```
## **6. Conclusion**

在本教程中，我们研究了Spring MVC如何使我们能够指定和完全自定义Http消息转换器来自动将Java实体与XML或JSON进行编组/解组。这当然是一个简单的定义，消息转换机制可以做得更多，从上一个测试示例可以看出。

我们还研究了如何利用与RestTemplate客户端相同的强大机制 - 导致完全类型安全的消费API