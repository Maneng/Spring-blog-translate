# Open Closed Principle

## Open Closed Principle

随着应用程序的发展，需要进行更改。添加新功能或在应用程序中更新现有功能时，需要进行更改。通常在这两种情况下，您都需要修改现有的代码，并且存在破坏应用程序功能的风险。对于良好的应用程序设计和代码编写部分，您应避免在需求更改时更改现有代码。相反，您应该通过添加新代码来扩展现有功能以满足新的要求。您可以按照开闭原则实现。

开放原则代表了五个SOLID软件工程原理的“O”，用于编写更易于阅读，可维护，易于升级和修改的精心设计的代码。贝特朗·迈尔（Bertrand Meyer）创造了“开放封闭原则”（Open Closed Principle）一词，该原则首先出现在他的书“面向对象软件构建”一书中，1988年发行。这是Java最初发布前的大约八年。

这个原则指出：“软件实体（类，模块，功能等）应该是开放的扩展，但关闭修改”。让我们把这个语句的两个关键短语放在一起：

- 1.“开放扩展”：这意味着一个软件模块的行为，说一个类可以扩展，使其以新的和不同的方式行事。重要的是要注意，术语“扩展”不仅限于使用Java扩展关键字的继承。如前所述，Java当时不存在。这意味着一个模块应该提供扩展点来改变它的行为。一种方法是在运行时使用多态来调用对象的扩展行为。
- 2.“关闭修改”：这意味着这种模块的源代码保持不变。

最初这些短语可能会让人很难理解 - 我们如何改变模块的行为而不进行代码的更改？ 在Java中的答案是使用抽象。您可以通过具体的子类创建来修复原来不足的功能并且通过具体子类表示无限可能行为的抽象（Java接口和抽象类）。


在我们编写符合开放原则的代码之前，我们来看看违反开放原则的后果。

## Open Closed Principle Violation (Bad Example)

考虑在批准健康保险索赔之前验证健康保险索赔的保险制度。我们可以遵循互补的单一责任原则，通过创建两个不同的类来建模这个要求。负责验证声明的HealthInsuranceSurveyor和负责批准声明的ClaimApprovalManager。

### HealthInsuranceSurveyor.java

```java
package guru.springframework.blog.openclosedprinciple;
public class HealthInsuranceSurveyor{
    public boolean isValidClaim(){
        System.out.println("HealthInsuranceSurveyor: Validating health insurance claim...");
        /*Logic to validate health insurance claims*/
        return true;
    }
}
```

### ClaimApprovalManager.java

```java
package guru.springframework.blog.openclosedprinciple;
public class ClaimApprovalManager {
    public void processHealthClaim (HealthInsuranceSurveyor surveyor)
    {
        if(surveyor.isValidClaim()){
            System.out.println("ClaimApprovalManager: Valid claim. Currently processing claim for approval....");
        }
    }
}
```


HealthInsuranceSurveyor和ClaimsApprovalManager类都能正常工作，保险系统的设计似乎是完美的，直到出现处理车辆保险索赔的新要求。我们现在需要添加一个新的VehicleInsuranceSurveyor类，这不应该会造成任何问题。但是，我们还需要修改ClaimApprovalManager类来处理车辆保险索赔。修改的ClaimApprovalManager将是这样：

### Modified ClaimApprovalManager.java

```java

package guru.springframework.blog.openclosedprinciple;
public class ClaimApprovalManager {
    public void processHealthClaim (HealthInsuranceSurveyor surveyor)
    {
        if(surveyor.isValidClaim()){
            System.out.println("ClaimApprovalManager: Valid claim. Currently processing claim for approval....");
        }
    }
    public void processVehicleClaim (VehicleInsuranceSurveyor surveyor)
    {
        if(surveyor.isValidClaim()){
            System.out.println("ClaimApprovalManager: Valid claim. Currently processing claim for approval....");
        }
    }
}
```

在上面的示例中，我们通过添加一个新的processVehicleClaim（）方法来修改ClaimApprovalManager类，以包含新的功能（车辆保险的索赔批准）。

显然，这显然违反了开放原则。我们需要修改类以添加对新功能的支持。事实上，我们在第一次我们写了ClaimApprovalManager类时违反了Open Closed原则。这可能在当前的示例中看起来是无害的，但考虑到企业应用程序中需要跟上快速变化的业务需求的后果。对于每个更改，您需要修改，测试和部署整个应用程序。这不仅使应用程序扩展很昂贵，而且易于出现软件错误。


## Coding to the Open Closed Principle

保险索赔示例的理想方法是设计ClaimApprovalManager类，方法如下：

- 开放以支持更多类型的保险索赔。
- 每当添加新类型的声明的支持时，关闭任何修改。

为了实现这一点，让我们通过创建一个抽象类来表示不同的声明验证行为来引入一层抽象。我们将命名为InsuranceSurveyor。

### InsuranceSurveyor.java

```java
package guru.springframework.blog.openclosedprinciple;
public abstract class InsuranceSurveyor {
    public abstract boolean isValidClaim();
}
```


接下来，我们将为每种类型的声明验证写入特定的类。

### HealthInsuranceSurveyor.java

```java
package guru.springframework.blog.openclosedprinciple;
public class HealthInsuranceSurveyor extends InsuranceSurveyor{
    public boolean isValidClaim(){
        System.out.println("HealthInsuranceSurveyor: Validating health insurance claim...");
        /*Logic to validate health insurance claims*/
        return true;
    }
}
```

### VehicleInsuranceSurveyor.java

```java
package guru.springframework.blog.openclosedprinciple;
public class VehicleInsuranceSurveyor extends InsuranceSurveyor{
    public boolean isValidClaim(){
       System.out.println("VehicleInsuranceSurveyor: Validating vehicle insurance claim...");
        /*Logic to validate vehicle insurance claims*/
        return true;
    }
}

```


在上面的例子中，我们编写了HealthInsuranceSurveyor和VehicleInsuranceSurveyor类，它们扩展了抽象的InsuranceSurveyor类。这两个类都提供了不同的isValidClaim（）方法的实现。我们现在将编写ClaimApprovalManager类以遵循开/关原则。


### ClaimApprovalManager.java

```java

package guru.springframework.blog.openclosedprinciple;
public class ClaimApprovalManager {
    public void processClaim(InsuranceSurveyor surveyor){
        if(surveyor.isValidClaim()){
            System.out.println("ClaimApprovalManager: Valid claim. Currently processing claim for approval....");
        }
    }
}
```


在上面的例子中，我们编写了一个processClaim（）方法来接受一个I​​nsuranceSurveyor类型，而不是指定一个具体的类型。这样，InsuranceSurveyor实现的任何进一步补充都不会影响ClaimApprovalManager类。我们的保险系统现在可以支持更多类型的保险索赔，并且每当添加新的索赔类型时，都会关闭任何修改。为了测试我们的例子，我们来写这个单元测试。


### ClaimApprovalManagerTest.java

```java

package guru.springframework.blog.openclosedprinciple;
import org.junit.Test;
import static org.junit.Assert.*;
public class ClaimApprovalManagerTest {
    @Test
    public void testProcessClaim() throws Exception {
      HealthInsuranceSurveyor healthInsuranceSurveyor=new HealthInsuranceSurveyor();
      ClaimApprovalManager claim1=new ClaimApprovalManager();
      claim1.processClaim(healthInsuranceSurveyor);
        VehicleInsuranceSurveyor vehicleInsuranceSurveyor=new VehicleInsuranceSurveyor();
        ClaimApprovalManager claim2=new ClaimApprovalManager();
        claim2.processClaim(vehicleInsuranceSurveyor);
    }
}
```

输出为：

```
.   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.2.3.RELEASE)
Running guru.springframework.blog.openclosedprinciple.ClaimApprovalManagerTest
HealthInsuranceSurveyor: Validating health insurance claim...
ClaimApprovalManager: Valid claim. Currently processing claim for approval....
VehicleInsuranceSurveyor: Validating vehicle insurance claim...
ClaimApprovalManager: Valid claim. Currently processing claim for approval....
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.001 sec - in guru.springframework.blog.openclosedprinciple.ClaimApprovalManagerTest
```


### Summary

大多数时候，软件实体的真正关闭实际上是不可能的，因为总是有机会改变会违反关闭。例如，在我们的保险范例中，处理特定类型的索赔的业务规则的变化将需要修改ClaimApprovalManager类。因此，在企业应用程序开发过程中，即使您不一定总是在每个方面都编写符合Open Closed原则的代码，因此随着应用程序的发展，采取步骤将是有益的。


