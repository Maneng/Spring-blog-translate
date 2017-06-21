
## **1. Overview**

最重要的Spring-MVC注释之一是@ModelAttribute注释。

@ModelAttribute是将方法参数或方法返回值绑定到命名模型属性的注释，然后将其公开到Web视图。

在下面的例子中，我们将通过一个共同的概念来证明注释的可用性和功能：从公司员工提交的表单。

## **2. The @ModelAttribute in Depth**

如介绍性段落所示，@ModelAttribute可以用作方法参数或方法级别。

### **2.1 At the Method Level**

当在方法级别使用注释时，它表示该方法的目的是添加一个或多个模型属性。这样的方法支持与@RequestMapping方法相同的参数类型，但不能直接映射到请求。

让我们来看看一个快速的例子，开始了解它的工作原理：

``` java
@ModelAttribute
public void addAttributes(Model model) {
    model.addAttribute("msg", "Welcome to the Netherlands!");
}
```

在这个例子中，我们展示了一个方法，它将一个名为msg的属性添加到控制器类中定义的所有模型中。

当然，我们会在文章中稍后再看到这一点。


一般来说，在调用任何请求处理程序方法之前，Spring-MVC将始终先调用该方法。也就是说，@ModelAttribute方法在使用@RequestMapping注释的控制器方法被调用之前被调用。序列之后的逻辑是，必须在控制器方法中开始任何处理之前创建模型对象。

您也可以将相应的类注释为@ControllerAdvice也很重要。因此，您可以在Model中添加将被标识为全局的值。这实际上意味着对于每个请求，存在默认值，对于响应部分中的每个方法。

### **2.2 As a Method Argument**

当用作方法参数时，它表示应该从模型检索参数。当不存在时，应首先实例化，然后添加到模型中，一旦出现在模型中，参数字段应从具有匹配名称的所有请求参数中填充。


在员工模型属性之后的代码片段中填充了从提交到addEmployee端点的表单中的数据。在调用提交方法之前，Spring MVC在幕后执行此操作：

``` java
@RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
public String submit(@ModelAttribute("employee") Employee employee) {
    // Code that uses the employee object
 
    return "employeeView";
}
```

稍后在本文中，我们将看到如何使用employee对象来填充employeeView模板的完整示例。

因此，它将表单数据与bean绑定。用@RequestMapping注释的控制器可以使用@ModelAttribute注释自定义类参数。

这通常被称为Spring-MVC中的数据绑定，这是一种常见的机制，可以使您不必单独解析每个表单域。

## **3. Form Example**

在本节中，我们将提供概述部分中提到的示例：提供用户（在我们具体的例子中的公司的雇员）输入一些个人信息（特别是名称和身份证件）的非常基本的形式。提交完成后，没有任何错误，用户期望看到先前提交的数据，显示在另一个屏幕上。

### **3.1 The View**

我们首先创建一个带有id和name字段的简单表单：


``` html
<form:form method="POST" action="/spring-mvc-java/addEmployee"
  modelAttribute="employee">
    <form:label path="name">Name</form:label>
    <form:input path="name" />
     
    <form:label path="id">Id</form:label>
    <form:input path="id" />
     
    <input type="submit" value="Submit" />
</form:form>
```


### **3.2 The Controller**

这是控制器类，上面提到的视图的逻辑正在实现：

``` java
@Controller
@ControllerAdvice
public class EmployeeController {
 
    private Map<Long, Employee> employeeMap = new HashMap<>();
 
    @RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
    public String submit(
      @ModelAttribute("employee") Employee employee,
      BindingResult result, ModelMap model) {
        if (result.hasErrors()) {
            return "error";
        }
        model.addAttribute("name", employee.getName());
        model.addAttribute("id", employee.getId());
 
        employeeMap.put(employee.getId(), employee);
 
        return "employeeView";
    }
 
    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("msg", "Welcome to the Netherlands!");
    }
}
```


在submit（）方法中，我们有一个Employee对象绑定到我们的View。你能看到这个注释的力量吗？您可以简单地将表单域映射到对象模型。在该方法中，我们从表单中获取值并将其设置为ModelMap。

最后我们返回employeeView，这意味着相应的JSP文件将被称为View代理。

此外，还有一个addAttributes（）方法。其目的是增加将在全球范围内识别的模型中的价值。也就是说，默认值将作为对每个控制器方法的每个请求的响应返回。我们还必须将特定类注释为@ControllerAdvice。

### **3.3 The Model**

如前所述，Model对象非常简单，包含“前端”属性所需的所有内容。现在，我们来看一个例子：

``` java
@XmlRootElement
public class Employee {
 
    private long id;
    private String name;
 
    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }
 
    // standard getters and setters removed
}
```

### **3.4 Wrap Up**

@ControllerAdvice协助控制器，特别是@ModelAttribute方法适用于所有@RequestMapping方法。当然，在其余的@RequestMapping方法之前，我们的addAttributes（）方法将是最先运行的方法。

保持这一点，并且在运行submit（）和addAttributes（）之后，我们可以在Controller类中返回的View中引用它们，通过提及它们在美元化的花括号中的给定名称，例如$ {name}。


### **3.5 Results View**

现在我们来打印我们从表单中收到的内容：

``` html
<h3>${msg}</h3>
Name : ${name}
ID : ${id}
```
## **4. Conclusion**

在本教程中，我们调查了@ModelAttribute注释的用法，用于方法参数和方法级使用情况。