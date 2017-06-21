

# **1. Overview**

通常，当我们需要验证用户输入时，Spring MVC提供了标准的预定义验证器。

但是，当我们需要验证一个更具体的类型输入时，我们有可能创建自己的定制验证逻辑。

在本文中，我们将这样做 - 我们将创建一个自定义验证器来验证具有电话号码字段的表单，然后显示多个字段的自定义验证器.

# **2. Setup**

要从获得API，请将依赖关系添加到您的pom.xml文件中：

``` xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.4.1.Final</version>
</dependency>
```

# **3. Custom Validation**

创建自定义验证器需要我们自己的注释，并在我们的模型中使用它来强制验证规则。

所以，我们来创建我们的自定义验证器 - 它检查电话号码。电话号码必须是数字超过8位，但不能超过11位数字

# **4. The New Annotation**

让我们创建一个新的@interface来定义我们的注释：

``` java
@Documented
@Constraint(validatedBy = ContactNumberValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ContactNumberConstraint {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```


使用@Constraint注释，我们定义了要验证我们的字段的类，message（）是在用户界面中显示的错误消息，附加代码是符合Spring标准的大多数样板代码。

# **5. Creating a Validator**

现在让我们创建一个验证器类来强制验证规则：

``` java
public class ContactNumberValidator implements
  ConstraintValidator<ContactNumberConstraint, String> {
 
    @Override
    public void initialize(ContactNumberConstraint contactNumber) {
    }
 
    @Override
    public boolean isValid(String contactField,
      ConstraintValidatorContext cxt) {
        return contactField != null && contactField.matches("[0-9]+")
          && (contactField.length() > 8) && (contactField.length() < 14);
    }
 
}
```
验证类实现ConstraintValidator接口，必须实现isValid方法;我们在这种方法中定义了我们的验证规则。

当然，我们将在这里提供一个简单的验证规则，以显示验证器的工作原理。

ConstraintValidator定义用于验证给定对象的给定约束的逻辑。实施必须遵守以下限制：
- 该对象必须解析为非参数类型
- 对象的通用参数必须是无界通配符类型

# **6. Applying Validation Annotation**

在我们的例子中，我们创建了一个简单的类，其中包含一个字段来应用验证规则。在这里，我们正在设置要验证的注释字段：

	

``` java
@ContactNumberConstraint
private String phone;
```

我们定义了一个字符串字段，并用我们的自定义注释@ContactNumberConstraint对它进行了注释。在我们的控制器中，我们创建了我们的映射，并处理了错误：

``` java
@Controller
public class ValidatedPhoneController {
  
    @GetMapping("/validatePhone")
    public String loadFormPage(Model m) {
        m.addAttribute("validatedPhone", new ValidatedPhone());
        return "phoneHome";
    }
     
    @PostMapping("/addValidatePhone")
    public String submitForm(@Valid ValidatedPhone validatedPhone,
      BindingResult result, Model m) {
        if(result.hasErrors()) {
            return "phoneHome";
        }
        m.addAttribute("message", "Successfully saved phone: "
          + validatedPhone.toString());
        return "phoneHome";
    }   
}
```


我们定义了具有单个JSP页面的简单控制器，并使用submitForm方法来强制验证我们的电话号码。

# **7. The View**

我们的视图是一个基本的JSP页面，其窗体具有单个字段。当用户提交表单时，该字段将被我们的自定义验证器验证，并重定向到同一页面，并显示验证成功或失败的消息：

``` html
<form:form
  action="/${pageContext.request.contextPath}/addValidatePhone"
  modelAttribute="validatedPhone">
    <label for="phoneInput">Phone: </label>
    <form:input path="phone" id="phoneInput" />
    <form:errors path="phone" cssClass="error" />
    <input type="submit" value="Submit" />
</form:form>
```


# **8. Tests**

现在让我们测试我们的控制器，并检查它是否给我们适当的响应和视图：


``` java
@Test
public void givenPhonePageUri_whenMockMvc_thenReturnsPhonePage(){
    this.mockMvc.
      perform(get("/validatePhone")).andExpect(view().name("phoneHome"));
}
```

另外，我们根据用户输入测试我们的字段是否被验证：


``` java
@Test
public void
  givenPhoneURIWithPostAndFormData_whenMockMVC_thenVerifyErrorResponse() {
  
    this.mockMvc.perform(MockMvcRequestBuilders.post("/addValidatePhone").
      accept(MediaType.TEXT_HTML).
      param("phoneInput", "123")).
      andExpect(model().attributeHasFieldErrorCode(
          "validatedPhone","phone","ContactNumberConstraint")).
      andExpect(view().name("phoneHome")).
      andExpect(status().isOk()).
      andDo(print());
}
```

在测试中，我们为用户提供“123”的输入，并且 - 正如我们预期的那样 - 一切正常，我们在客户端看到错误。



# **9. Custom Class Level Validation**

也可以在类级别定义自定义验证注释，以验证该类的多个属性。

这种情况的常见用例是验证类中的两个字段是否具有匹配值。


## **9.1. Creating the Annotation**

我们添加一个名为FieldsValueMatch的新注释，可以稍后在类中应用。注释将具有两个参数field和fieldMatch，它们表示要比较的字段的名称：

``` java
@Constraint(validatedBy = FieldsValueMatchValidator.class)
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface FieldsValueMatch {
 
    String message() default "Fields values don't match!";
 
    String field();
 
    String fieldMatch();
 
    @Target({ ElementType.TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @interface List {
        FieldsValueMatch[] value();
    }
}
```

我们可以看到我们的自定义注释还包含一个List子界面，用于在类上定义多个FieldsValueMatch注释。

## **9.2. Creating the Validator**

接下来，我们需要添加将包含实际验证逻辑的FieldsValueMatchValidator类：

``` java
public class FieldsValueMatchValidator 
  implements ConstraintValidator<FieldsValueMatch, Object> {
 
    private String field;
    private String fieldMatch;
 
    public void initialize(FieldsValueMatch constraintAnnotation) {
        this.field = constraintAnnotation.field();
        this.fieldMatch = constraintAnnotation.fieldMatch();
    }
 
    public boolean isValid(Object value, 
      ConstraintValidatorContext context) {
 
        Object fieldValue = new BeanWrapperImpl(value)
          .getPropertyValue(field);
        Object fieldMatchValue = new BeanWrapperImpl(value)
          .getPropertyValue(fieldMatch);
         
        if (fieldValue != null) {
            return fieldValue.equals(fieldMatchValue);
        } else {
            return fieldMatchValue == null;
        }
    }
}
```


isValid（）方法检索两个字段的值，并检查它们是否相等。


## **9.3. Applying the Annotation**

我们创建一个NewUserForm模型类，用于用户注册所需的数据，它具有两个电子邮件和密码属性，以及两个verifyEmail和verifyPassword属性，以重新输入两个值。

由于我们有两个字段来检查其对应的匹配字段，所以我们在NewUserForm类上添加两个@FieldsValueMatch注释，一个用于电子邮件值，一个用于密码值：

``` java
@FieldsValueMatch.List({ 
    @FieldsValueMatch(
      field = "password", 
      fieldMatch = "verifyPassword", 
      message = "Passwords do not match!"
    ), 
    @FieldsValueMatch(
      field = "email", 
      fieldMatch = "verifyEmail", 
      message = "Email addresses do not match!"
    )
})
public class NewUserForm {
    private String email;
    private String verifyEmail;
    private String password;
    private String verifyPassword;
 
    // standard constructor, getters, setters
}
```


为了在Spring MVC中验证模型，让我们创建一个带有/ user POST映射的控制器，该控件接收一个用@Valid注释的NewUserForm对象，并验证是否有任何验证错误：


``` java
@Controller
public class NewUserController {
 
    @GetMapping("/user")
    public String loadFormPage(Model model) {
        model.addAttribute("newUserForm", new NewUserForm());
        return "userHome";
    }
 
    @PostMapping("/user")
    public String submitForm(@Valid NewUserForm newUserForm, 
      BindingResult result, Model model) {
        if (result.hasErrors()) {
            return "userHome";
        }
        model.addAttribute("message", "Valid form");
        return "userHome";
    }
}
```


## **9.4. Testing the Annotatio**


要验证我们的自定义类级别注释，让我们编写一个JUnit测试，它将匹配信息发送到/ user端点，然后验证响应是否包含错误：


``` java
public class ClassValidationMvcTest {
  private MockMvc mockMvc;
     
    @Before
    public void setup(){
        this.mockMvc = MockMvcBuilders
          .standaloneSetup(new NewUserController()).build();
    }
     
    @Test
    public void givenMatchingEmailPassword_whenPostNewUserForm_thenOk() 
      throws Exception {
        this.mockMvc.perform(MockMvcRequestBuilders
          .post("/user")
          .accept(MediaType.TEXT_HTML).
          .param("email", "john@yahoo.com")
          .param("verifyEmail", "john@yahoo.com")
          .param("password", "pass")
          .param("verifyPassword", "pass"))
          .andExpect(model().errorCount(0))
          .andExpect(status().isOk());
    }
}
```


接下来，我们还添加一个JUnit测试，它将不匹配的信息发送到/ user端点，并声明结果将包含两个错误：


``` java
@Test
public void givenNotMatchingEmailPassword_whenPostNewUserForm_thenOk() 
  throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders
      .post("/user")
      .accept(MediaType.TEXT_HTML)
      .param("email", "john@yahoo.com")
      .param("verifyEmail", "john@yahoo.commmm")
      .param("password", "pass")
      .param("verifyPassword", "passsss"))
      .andExpect(model().errorCount(2))
      .andExpect(status().isOk());
    }
```


## **10. Summary**

在这篇快速的文章中，我们展示了如何创建自定义验证器来验证一个字段或类，并将它们连接到Spring MVC中。