# Dependency Inversion Principle

## Dependency Inversion Principle

作为Java程序员，您可能听说过代码耦合，并且被告知要避免紧密耦合的代码。编写“好代码”无效是应用程序中存在紧耦合代码的主要原因。例如，使用新的运算符创建类的对象会导致类紧密耦合到另一个类。这种耦合似乎是无害的，不会扰乱小型计划。但是，当您进入企业应用程序开发时，紧密耦合的代码可能会导致严重的不利后果。

当一个类明确地知道另一个类的设计和实现时，一个类的改变会增加打破另一个类的风险。这种变化可能会在整个应用程序中产生波纹效应，从而使应用程序变得脆弱。为了避免这种问题，你应该写出松散耦合的“好的代码”，并且为了支持这一点，你可以转向依赖性反转原则。

依赖性反转原则代表面向对象编程的五个实体原理的最后一个“D”。罗伯特·马丁（Robert C. Martin）首先假定了依赖性反转原则，并在1996年发表。原则规定

> “A.高级模块不应该依赖于低级模块。两者都应该取决于抽象。B. 摘要不应该依赖细节。细节应该取决于抽象。“

传统的应用架构遵循自上而下的设计方法，其中高级问题被分解成较小的部分。换句话说，根据这些较小的部分来描述高级设计。因此，直接写入的高级模块取决于较小的（低级）模块。


依赖性倒置原则说的是高层模块不用应该依赖低层模块,都应该依赖于抽象。让我们的通过Java来看看它。

![a](https://udemy-images.s3.amazonaws.com/redactor/raw/2017-07-22_06-47-15-af276c77e0688e3d7c33c360d8c74b37.png)


在上图中，没有依赖性反转原则，包A中的对象A引用包B中的对象B.伴随依赖性反转原理，接口A作为包A中的抽象引入。对象A现在引用接口A和对象B继承自接口A.原则是：


1.对象A和对象B现在都取决于接口A，即抽象。
2.它将从对象A到对象B存在的依赖关系转换为依赖于抽象（接口A）的对象B.

在我们编写遵循依赖性反转原则的代码之前，我们来研究一个典型的违反原则的问题。

## 依赖倒置原则冲突（为实施例）

考虑打开或关闭灯泡的电开关的示例。我们可以通过创建两个类来建模这个要求：ElectricPowerSwitch和LightBulb。我们先写LightBulb类。

### LightBulb.java

```java

public class LightBulb {
    public void turnOn() {
        System.out.println("LightBulb: Bulb turned on...");
    }
    public void turnOff() {
        System.out.println("LightBulb: Bulb turned off...");
    }
}
```



在上面的LightBulb类中，我们写了turnOn（）和turnOff（）方法打开和关闭一个灯泡。 接下来，我们将编写ElectricPowerSwitch类。


### ElectricPowerSwitch.java

```java
public class ElectricPowerSwitch {
    public LightBulb lightBulb;
    public boolean on;
    public ElectricPowerSwitch(LightBulb lightBulb) {
        this.lightBulb = lightBulb;
        this.on = false;
    }
    public boolean isOn() {
        return this.on;
    }
    public void press(){
        boolean checkOn = isOn();
        if (checkOn) {
            lightBulb.turnOff();
            this.on = false;
        } else {
            lightBulb.turnOn();
            this.on = true;
        }
    }
}
```


上面的例子中，我们写了一个引用LightBulb的FieldPowerPowerSwitch类。在构造函数中，我们创建了一个LightBulb对象并将其分配给该字段。然后我们写了一个isOn（）方法，它将ElectricPowerSwitch的状态返回为一个布尔值。在press（）方法中，根据状态，我们调用turnOn（）和turnOff（）方法。

我们的开关现在可以用来打开和关闭灯泡。但我们所做的错误是显而易见的。我们的高级别的PowerPowerSwitch类直接依赖于低级别的LightBulb类。如果您在代码中看到，LightBulb类在ElectricPowerSwitch中是硬编码的。但是，开关不应该与灯泡相连。应该能够打开和关闭其他设备和设备，比如风扇，交流电，或整个游乐园的闪电系统。现在，想像一下，在每次添加新的设备或设备时，我们将在ElectricPowerSwitch类中进行修改。我们可以得出结论，我们的设计是有缺陷的，我们需要通过遵循依赖性反转原则重新审视它。


## 遵循依赖性反转原则

要在我们的示例中遵循依赖性反转原则，我们将需要一个PowerPowerSwitch和LightBulb类将依赖的抽象。但是，在创建之前，让我们为交换机创建一个接口。

```java
package guru.springframework.blog.dependencyinversionprinciple.highlevel;
public interface Switch {
    boolean isOn();
    void press();
}
```

我们用isOn（）和press（）方法为交换机编写了一个接口。如果需要，这个接口将使我们灵活地插入其他类型的交换机，比如说遥控开关。接下来，我们将以接口的形式编写抽象，我们将其称为可切换。

### Switchable.java

```java
package guru.springframework.blog.dependencyinversionprinciple.highlevel;
public interface Switchable {
    void turnOn();
    void turnOff();
}
```

在上面的例子中，我们使用turnOn（）和turnoff（）方法编写了可切换接口。从现在起，应用程序中的任何可切换设备都可以实现此接口并提供自己的功能。我们的ElectricPowerSwitch类也将依赖于此接口，如下所示：


### ElectricPowerSwitch.java
```java


package guru.springframework.blog.dependencyinversionprinciple.highlevel;
public class ElectricPowerSwitch implements Switch {
    public Switchable client;
    public boolean on;
    public ElectricPowerSwitch(Switchable client) {
        this.client = client;
        this.on = false;
    }
    public boolean isOn() {
        return this.on;
    }
   public void press(){
       boolean checkOn = isOn();
       if (checkOn) {
           client.turnOff();
           this.on = false;
       } else {
             client.turnOn();
             this.on = true;
       }
   }
}

```

在ElectricPowerSwitch类中，我们实现了Switch接口，并引用了可切换接口，而不是字段中的任何具体类。然后，我们在接口上调用turnOn（）和turnoff（）方法，在运行时将对传递给构造函数的对象进行调用。现在，我们可以添加低级可切换类，而不用担心修改ElectricPowerSwitch类。我们将添加两个类：LightBulb和Fan。


### LightBulb.java

```java
package guru.springframework.blog.dependencyinversionprinciple.lowlevel;
import guru.springframework.blog.dependencyinversionprinciple.highlevel.Switchable;
public class LightBulb implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("LightBulb: Bulb turned on...");
    }
    @Override
    public void turnOff() {
        System.out.println("LightBulb: Bulb turned off...");
    }
}

```

### Fan.java

```java
package guru.springframework.blog.dependencyinversionprinciple.lowlevel;
import guru.springframework.blog.dependencyinversionprinciple.highlevel.Switchable;
public class Fan implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("Fan: Fan turned on...");
    }
    @Override
    public void turnOff() {
        System.out.println("Fan: Fan turned off...");
    }
}

```


在我们写的LightBulb和Fan类中，我们实现了可切换接口，以提供自己的打开和关闭功能。在编写java类时，如果您错过了我们如何将其安排在包中，请注意，我们将可切换接口与低级电气设备类保持在不同的包中。尽管如此，除了一个import语句，除了一个import语句之外，这并没有任何区别，所以我们已经明确地表达了我们的意图 - 我们希望低级别依赖（反向）我们的抽象。如果我们稍后决定将高级包作为其他应用程序可用于其设备的公共API发布，这也将帮助我们。为了测试我们的例子，我们来写这个单元测试。


### ElectricPowerSwitchTest.java

```java

package guru.springframework.blog.dependencyinversionprinciple.highlevel;
import guru.springframework.blog.dependencyinversionprinciple.lowlevel.Fan;
import guru.springframework.blog.dependencyinversionprinciple.lowlevel.LightBulb;
import org.junit.Test;
public class ElectricPowerSwitchTest {
    @Test
    public void testPress() throws Exception {
     Switchable switchableBulb=new LightBulb();
     Switch bulbPowerSwitch=new ElectricPowerSwitch(switchableBulb);
     bulbPowerSwitch.press();
     bulbPowerSwitch.press();
    Switchable switchableFan=new Fan();
    Switch fanPowerSwitch=new ElectricPowerSwitch(switchableFan);
    fanPowerSwitch.press();
    fanPowerSwitch.press();
    }
}
```


## Summary of the Dependency Inversion Principle

罗伯特·马丁将“依赖性反转原则”视为公开封闭原则和Liskov替代原则的一流组合，发现自己的名字足够重要。在使用依赖性反转原则时，会附带编写额外代码的开销，它提供的优势超过了额外的努力。因此，从现在起开始编写代码时，请考虑依赖关系破坏代码的可能性，如果是这样，可以添加抽象以使您的代码对变更有弹性。

## 依赖性反转原则和Spring框架

您可能认为依赖性反转原则与依赖注入有关，因为它适用于Spring Framework，您将是正确的。鲍勃·马丁叔叔在马丁·福勒（Martin Fowler）创造了“依赖注入”（Dependency Injection）术语的基础上，提出了依赖性反转概念。这两个概念是高度相关的。依赖性反转更侧重于代码的结构，其重点是保持代码松散耦合。另一方面，依赖注入是代码功能如何工作的方式。在使用Spring Framework进行编程时，Spring正在使用依赖注入来组合应用程序。依赖性反转是什么使您的代码解耦，因此Spring可以在运行时使用依赖注入。


