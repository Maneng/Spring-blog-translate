# Interface Segregation Principle

## Interface Segregation Principle

接口形成了Java编程语言的核心部分，它们被广泛应用于企业应用程序中，以实现抽象，并支持类型的多个继承 - 类实现多个接口的能力。从编码的角度来看，编写接口很简单。您可以使用interface关键字来创建一个接口并在其中声明方法。其他类可以使用该接口与implements关键字，然后提供已声明方法的实现。作为Java程序员，您必须编写大量的接口，但关键问题是您是否在保留设计原则的同时编写它们？编写接口时要遵循的设计原则是接口隔离原则。

接口隔离原则代表面向对象编程的五个SOLID原理的“I”，用于编写更易于阅读，可维护，易于升级和修改的精心设计的代码。敏捷软件开发：原理，模式和实践，他在2002年的书中提到，这一原则首先被罗伯特·马丁（Robert C. Martin）用来咨询施乐。这个原则指出，“客户不应该被迫依赖于他们不使用的方法”。这里，术语“客户端”是指接口的实现类。

接口隔离原理说的是，您的接口不应该随着实现类不需要的方法而膨胀。对于这样的接口，也称为“胖接口”，即使对于他们不需要的方法，实现类也不必要地被迫提供实现（虚拟/空）。此外，实现类在接口变化时可能会发生变化。方法签名的方法或更改的添加需要修改所有实现类，即使其中一些不使用该方法。

接口隔离原则主张将“胖接口”分为更小和高度凝聚力的接口，称为“角色接口”。每个“角色接口”都声明一个或多个特定行为的方法。因此，客户端而不是实现“胖接口”，只能实现方法与它们相关的那些“角色接口”。


## 接口隔离原则违规（错误示例）

考虑构建不同类型玩具的应用程序的要求。每个玩具都会有一个价格和颜色。一些玩具，如玩具车或玩具火车，还可以移动，而玩具飞机等玩具也可以移动和飞行。这是界定玩具行为的接口。


### Toy.java

```java

public interface Toy {
    void setPrice(double price);
    void setColor(String color);
    void move();
    void fly();
}
```

代表玩具飞机的课程可以实现玩具接口，并提供所有接口方法的实现。但是，想象一个代表玩具屋的类。这是ToyHouse班的外观。

### ToyHouse.java

```java
public class ToyHouse implements Toy {
    double price;
    String color;
    @Override
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public void setColor(String color) {
        this.color=color;
    }
    @Override
    public void move(){}
    @Override
    public void fly(){}    
}
```

如代码中所示，ToyHouse需要提供move（）和fly（）方法的实现，尽管它不需要它们。这违反了接口隔离原则。这种违规会影响代码的可读性，并使程序员混淆。想象一下，您正在编写ToyHouse类，IDE的智能感知功能会弹出fly（）方法以进行自动完成。不完全是玩具屋想要的行为，是吗？

违反接口隔离原则也会导致违反开闭原则。例如，考虑到玩具接口被修改为包括一个walk（）方法来容纳玩具机器人。因此，即使玩具不走路，您现在也需要修改所有现有的玩具实施类，以包括一个步行方法。事实上，玩具实施阶段永远不会关闭修改，这将导致易于维护的难度和昂贵的脆弱应用程序。

## 遵循接口隔离原则
通过遵循接口隔离原则，您可以解决玩具构建应用程序的主要问题 - 玩具接口强制客户端（实现类）依赖于它们不使用的方法

解决方案是将玩具接口分成多个角色接口，以实现特定的行为。让我们分离玩具接口，使我们的应用程序现在有三个接口：玩具，可移动和可飞行。

### Toy.java

```java

package guru.springframework.blog.interfacesegregationprinciple;
public interface Toy {
     void setPrice(double price);
     void setColor(String color);
}
```

### Movable.java
```java

package guru.springframework.blog.interfacesegregationprinciple;
public interface Movable {
    void move();
}
```


### Flyable.java
```java

package guru.springframework.blog.interfacesegregationprinciple;
public interface Flyable {
    void fly();
}
```


在上面的例子中，我们首先使用setPrice（）和setColor（）方法编写了Toy接口。由于所有玩具都将具有价格和颜色，所有玩具实施类都可以实现此接口。然后，我们写了Movable和Flyable接口来表示玩具中的移动和飞行行为。我们来写实现类。


### ToyHouse.java


```java
package guru.springframework.blog.interfacesegregationprinciple;
public class ToyHouse implements Toy {
    double price;
    String color;
    @Override
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public void setColor(String color) {
        this.color=color;
    }
    @Override
    public String toString(){
        return "ToyHouse: Toy house- Price: "+price+" Color: "+color;
    }
}
```

### ToyCar.java

```java
package guru.springframework.blog.interfacesegregationprinciple;
public class ToyCar implements Toy, Movable {
    double price;
    String color;
    @Override
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public void setColor(String color) {
     this.color=color;
    }
    @Override
    public void move(){
        System.out.println("ToyCar: Start moving car.");
    }
    @Override
    public String toString(){
        return "ToyCar: Moveable Toy car- Price: "+price+" Color: "+color;
    }
}
```

### ToyPlane.java

```java
package guru.springframework.blog.interfacesegregationprinciple;
public class ToyPlane implements Toy, Movable, Flyable {
    double price;
    String color;
    @Override
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public void setColor(String color) {
        this.color=color;
    }
    @Override
    public void move(){
        System.out.println("ToyPlane: Start moving plane.");
    }
    @Override
    public void fly(){
        System.out.println("ToyPlane: Start flying plane.");
    }
    @Override
    public String toString(){
        return "ToyPlane: Moveable and flyable toy plane- Price: "+price+" Color: "+color;
    }
}

```

你所见，实现类现在只实现他们感兴趣的接口。我们的类没有不必要的代码杂乱，更易于读取，并且由于接口方法的改变而更容易修改。

接下来，我们来写一个类来创建实现类的对象

### ToyBuilder.java

```java
package guru.springframework.blog.interfacesegregationprinciple;
public class ToyBuilder {
    public static ToyHouse buildToyHouse(){
        ToyHouse toyHouse=new ToyHouse();
        toyHouse.setPrice(15.00);
        toyHouse.setColor("green");
        return toyHouse;
        }
    public static ToyCar buildToyCar(){
        ToyCar toyCar=new ToyCar();
        toyCar.setPrice(25.00);
        toyCar.setColor("red");
        toyCar.move();
        return toyCar;
    }
    public static ToyPlane buildToyPlane(){
        ToyPlane toyPlane=new ToyPlane();
        toyPlane.setPrice(125.00);
        toyPlane.setColor("white");
        toyPlane.move();
        toyPlane.fly();
        return toyPlane;
    }
}
```
在上面的代码示例中，我们用ToyBuilder类编写了三个静态方法来创建ToyHouse，ToyCar和ToyPlane类的对象。最后，我们来写这个单元测试来测试我们的例子。


### ToyBuilderTest.java


```java
package guru.springframework.blog.interfacesegregationprinciple;
import org.junit.Test;
public class ToyBuilderTest {
    @Test
    public void testBuildToyHouse() throws Exception {
    ToyHouse toyHouse=ToyBuilder.buildToyHouse();
    System.out.println(toyHouse);
    }
    @Test
    public void testBuildToyCar() throws Exception {
    ToyCar toyCar=ToyBuilder.buildToyCar();;
        System.out.println(toyCar);
    }
    @Test
    public void testBuildToyPlane() throws Exception {
    ToyPlane toyPlane=ToyBuilder.buildToyPlane();
        System.out.println(toyPlane);
    }
}
```



## Summary of Interface Segregation Principle

接口隔离原则和单一责任原则都具有相同的目标：确保小型，集中和高度凝聚力的软件组件。不同之处在于单一责任原则涉及类，而接口隔离原则涉及接口。接口隔离原理易于理解，易于理解。但是，识别不同的接口有时可能是一个挑战，因为需要谨慎考虑以避免接口的扩散。因此，在编写接口的同时，考虑实现类具有不同行为集合的可能性，如果是这样，则将接口隔离成多个接口，每个接口具有特定的角色。


## Spring框架中的接口隔离原理

在我使用Spring框架进行编程时，我已经写了关于依赖注入编程的这个博客的次数。使用Spring框架进行企业应用程序开发时，接口隔离原则变得尤为重要。

随着您正在构建的应用程序的大小和范围增加，您将需要可插拔组件。即使只是单元测试你的类，接口隔离原则也有一个作用。如果您正在测试一个您为依赖注入编写的类，就像我之前写过的那样，写入接口是理想的。通过设计类以对接口使用依赖注入，实现指定接口的任何类都可以注入到类中。在测试课程时，您可能希望注入一个模拟对象，以满足您的单元测试的需求。但是，当您编写的类正在生产中运行时，Spring Framework将会将该接口的真正全功能实现注入到类中。

接口隔离原则和依赖注入是使用Spring框架开发企业级应用程序时掌握的两个非常强大的概念。






### Conclusion

面向对象的语言（如Java）非常强大，为开发人员提供了极大的灵活性。你可以误用或滥用任何语言。在多态性帖子中，我解释了“Is-A”测试。如果你正在编写扩展类的对象，但是'Is-A'测试失败，你可能违反了Liskov替换原则。
