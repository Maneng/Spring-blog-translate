# Java Bean Validation入门
**目录**  

- [**1. Overview**](#1-overview)
- [**2. JSR 303 and JSR 349**](#2-jsr-303-and-jsr-349)
- [**3. Dependencies**](#3-dependencies)
  - [**3.1. Validation API**](#31-validation-api)
  - [**3.2. Validation API Reference Implementation**](#32-validation-api-reference-implementation)
  - [**3.3. Expression Language Dependencies**](#33-expression-language-dependencies)
- [**4. Using Validation Annotations**](#4-using-validation-annotations)
- [**5. Programmatic Validation**](#5-programmatic-validation)
  - [**5.1. Defining the Bean**](#51-defining-the-bean)
  - [**5.2. Validate the Bean**](#52-validate-the-bean)
- [**6. Conclusion**](#6-conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## **1. Overview**

在这篇快速的文章中，我们将介绍使用标准框架（JSR 303和JSR 349）来验证Java bean的基础知识。

在大多数应用程序中验证用户输入当然是一个超常见的要求，Java Bean验证框架已成为处理这种逻辑的事实上的标准。

## **2. JSR 303 and JSR 349**

JSR 303是用于bean验证的Java API的一个规范，它是JavaEE和JavaSE的一部分，它确保bean的属性符合特定标准，使用诸如@NotNull，@Min和@Max之类的注释。

JSR 349扩展了JSR 303，具有诸如约束冲突消息中的动态表达式评估，消息插入等功能。

有关规范的完整信息，请阅读JSR 303或JSR 349 JSR。

## **3. Dependencies**

我们将在这里使用Maven示例来显示确切的所需依赖关系，但是当然这些jar可以通过多种方式添加到项目中。


### **3.1. Validation API**

根据规范JSR 303和349，验证api依赖项包含标准验证API：

``` vbscript-html
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```

### **3.2. Validation API Reference Implementation**

Hibernate Validator是验证API的参考实现。为了使用它，我们必须添加以下依赖关系。

``` vbscript-html
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.1.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator-annotation-processor</artifactId>
    <version>5.2.1.Final</version>
</dependency>
```
这里的一个快速说明是，hibernate验证器与Hibernate的持久性方面完全分离，并将其作为依赖项添加，我们不将这些持久性方面添加到项目中。

### **3.3. Expression Language Dependencies**

JSR 349提供变量插值，允许表达式在违例消息中。为了解析这些表达式，我们必须在表达式语言API和该API的实现上添加依赖关系。 GlassFish提供了参考实现。

``` vbscript-html
<dependency>
    <groupId>javax.el</groupId>
    <artifactId>javax.el-api</artifactId>
    <version>2.2.4</version>
</dependency>
 
<dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>javax.el</artifactId>
    <version>2.2.4</version>
</dependency>
```

如果未添加这些JAR，则会在运行时收到错误消息，如下所示：


``` sql
HV000183: Unable to load ‘javax.el.ExpressionFactory’. Check that you have the EL dependencies on the classpath, or use ParameterMessageInterpolator instead
```

## **4. Using Validation Annotations**

我们将使用User bean作为这里的主要示例，并致力于为其添加一些简单的验证：

``` java
import javax.validation.constraints.AssertTrue;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
 
public class User {
 
    @NotNull(message = "Name cannot be null")
    private String name;
 
    @AssertTrue
    private boolean working;
 
    @Size(min = 10, max = 200, message = "About Me must be between 10 and 200 characters")
    private String aboutMe;
 
    @Min(value = 18, message = "Age should not be less than 18")
    @Max(value = 150, message = "Age should not be greater than 150")
    private int age;
 
    // standard setters and getters 
}
```


示例中使用的所有注释都是标准JSR注释：

- @NotNull – Validates that the annotated property value is not null 验证注释的属性值不为null
- @AssertTrue – Validates that the annotated property value is true 验证注释属性值为true
- @Size – Validates that the annotated property value has a size between the attributes min and max; can be applied to String, Collection, Map, and array properties 验证注释属性值的大小在属性min和max之间;可以应用于String，Collection，Map和数组属性
- @Min – Validates that the annotated property has a value no smaller than the value attribute 验证注释属性的值不小于value属性
- @Max – Validates that the annotated property has a value no larger than the value attribute 验证注释属性的值不大于value属性

些注释接受附加属性，但是message属性对它们都是共同的。这是当相应属性的值验证失败时通常会显示的消息。


## **5. Programmatic Validation**

一些框架（如Spring）有简单的方法可以通过使用注释触发验证过程。这主要是因为我们不必与编程验证API进行交互。

我们现在去手动路线，实际上是以编程方式设置的：

``` sml
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
```
为了验证一个bean，我们必须先使用一个ValidatorFactory来构造一个Validator对象。

### **5.1. Defining the Bean**

我们不会设置这个无效用户 - 使用空值名称：

``` pf
User user = new User();
user.setWorking(true);
user.setAboutMe("Its all about me!");
user.setAge(50);
```
### **5.2. Validate the Bean**

现在我们有一个验证器，我们可以通过将它传递给validate方法来验证我们的bean。任何违反User对象中定义的约束的行为都将返回为Set。

> Set<ConstraintViolation<User>> violations = validator.validate(user);

通过迭代违规，我们可以通过使用getMessage方法获取所有违规消息。

``` java
for (ConstraintViolation<User> violation : violations) {
    log.error(violation.getMessage()); 
}
```

在我们的示例（ifNameIsNull_nameValidationFails）中，该集合将包含一个单一的ConstraintViolation，其消息是“Name not not null”。

## **6. Conclusion**

本教程的重点是简单的通过标准的Java Validation API，并使用javax.validation注解和API说明了bean验证的基础知识