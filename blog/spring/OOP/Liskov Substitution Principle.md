# Liskov Substitution Principle

## Liskov Substitution Principle

iskov替代原则是面向对象编程（单一责任，开闭原则，Liskov替换，界面隔离和依赖性反转）的SOLID原则之一。我们已经写了单一的责任原则，结合的这五个原则用于使面向对象的代码更易于阅读，更易于维护，更易于升级和修改。

Liskov替换原则说明如下：“在计算机程序中，如果S是T的子类型，则可以用类型S的对象替换类型T的对象（即，类型S的对象可以替代类型T的对象）而不改变该程序的任何所需的属性（正确性，执行任务等）“。简单地说，面向对象程序中某个类的任何对象都可以被一个子类的对象所替代。为了更好地理解这个原则，我们将进行一些小的挖掘，以简要地提醒自己关于继承及其属性的概念，以及子类型，一种多态的形式。


## 继承，多态，子类型

继承是一个相当简单易懂的概念 - 当一个对象或一个类基于另一个对象或类时。当类从另一个类“继承”时，这意味着继承的类（也称为子类或子类）包含超类（父类）的所有特性，但也可以包含新属性。让我们用一个常见的例子来说明这一点：如果你有一个类Watch，你可以继承该类来获得一个类PocketWatch。怀表仍然是手表，它只是一些额外的功能。


另一个例子是一个叫做“女人”的子类，叫做“母亲”。一个母亲还是一个女人，另外还有一个孩子。这使我们到下一个我们应该解释的术语，这被称为多态性：对象可以在某种情况下以某种方式行事，而在另一种情况下可以以另一种方式行事。在面向对象编程中，这称为上下文相关行为。使用最后一个例子：一个母亲，当和她的孩子一起散步或参加一个学校的父母的会议，将表现为一个母亲。但是当她和她的朋友在一起时，在工作中或者只是在做错事，她会像一个女人一样行事。 （可以看出，这个差异不是那么严格。）

子类型是一个与多态不相同的概念。然而，两者之间的密切联系和融合在一起，如C ++，Java和C＃，它们之间的区别实际上是不存在的。但是，为了完整起见，我们仍然会给出子类型的正式定义，但是在本文中将不再详细讨论细节。 “在编程语言理论中，子类型（也是子类型多态性或包含多态性）是一种类型多态性的形式，其中子类型是通过某些替代性概念与另一数据类型（超类型）相关的数据类型，这意味着程序元素通常子程序或函数，用于对超类型的元素进行操作也可以对子类型的元素进行操作。如果S是T的子类型，则子类型关系通常被写为S <T，表示类型S的任何术语可以在期望类型T的术语的上下文中被安全地使用。

这给我们带来了文章的原始主题--Liskov替代原则。


## Liskov替代原则示例

1988年由Barbara Liskov撰写的Liskov替代原则指出，引用基类的函数必须能够在不知道的情况下使用派生（子）类的对象。对于程序员来说，重要的是注意到，与其他“四人帮”原则不同，其破坏可能会导致恶劣的工作代码，违反这一原则很可能导致错误或难以维护代码。

### 让我们用Java来说明一下：


```java

class TrasportationDevice
{
   String name;
   String getName() { ... }
   void setName(String n) { ... }
   double speed;
   double getSpeed() { ... }
   void setSpeed(double d) { ... }
   
   Engine engine;
   Engine getEngine() { ... }
   void setEngine(Engine e) { ... }
   void startEngine() { ... }
}
class Car extends TransportationDevice
{
   @Override
   void startEngine() { ... }
}
```

这里没有问题吧？一辆汽车绝对是一个运输设备，在这里我们可以看到它覆盖了其超类的startEngine（）方法。


### 我们再添加一个运输设备：


```java

class Bicycle extends TransportationDevice
{
   @Override
   void startEngine() /*problem!*/
}
```

一切都不按计划进行！是的，自行车是运输设备，但是它没有引擎，因此startEngine方法无法实现。

这些是违反Liskov替代原则导致的问题，通常是可以通过一种不做任何事情的方法来识别，甚至不能实现。

这些问题的解决方案是一个正确的继承层次结构，在我们的例子中，我们将通过区分具有和不具有引擎的传输设备的类来解决问题。即使自行车是运输装置，它也没有发动机。在这个例子中，我们对运输设备的定义是错误的。它不应该有一个引擎。


我们可以重构我们的TransportationDevice类，如下所示：

```java

class TrasportationDevice
{
   String name;
   String getName() { ... }
   void setName(String n) { ... }
 
   double speed;
   double getSpeed() { ... }
   void setSpeed(double d) { ... }
}
```

现在我们可以将TransportationDevice扩展到非机动设备。

```java
class DevicesWithoutEngines extends TransportationDevice
{  
   void startMoving() { ... }
}
```
并将TransportationDevice扩展到电动装置。这是更适合添加的Engine对象。

```java
class DevicesWithEngines extends TransportationDevice
{  
   Engine engine;
   Engine getEngine() { ... }
   void setEngine(Engine e) { ... }
 
   void startEngine() { ... }
}
```

因此，我们的汽车类更加专业化，同时遵守Liskov替代原则。


```java
class Car extends DevicesWithEngines
{
   @Override
   void startEngine() { ... }
}
```

而我们的自行车类也符合Liskov替代原则。

```java

class Bicycle extends DevicesWithoutEngines
{
   @Override
   void startMoving() { ... }
}
```

### Conclusion

面向对象的语言（如Java）非常强大，为开发人员提供了极大的灵活性。你可以误用或滥用任何语言。在多态性帖子中，我解释了“Is-A”测试。如果你正在编写扩展类的对象，但是'Is-A'测试失败，你可能违反了Liskov替换原则。
