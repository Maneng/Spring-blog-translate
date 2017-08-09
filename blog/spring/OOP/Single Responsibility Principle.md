# Single Responsibility Principle

## 单一责任原则 面向对象的术语

在面向对象的编程中（Java，除其他语言之外，遵循这一范式），您将经常听到诸如鲁棒性，内聚性，耦合等术语。凝聚力是衡量一个模块内的代码段数量（一个类的方法） ，包中的类...）属于一起。凝聚力越高越好，因为高凝聚力意味着更容易的维护和调试，更大的代码功能和可重用性。术语凝聚力有时与耦合的概念形成对比，并且通常，模块的松散耦合与高内聚性有关。

另一个广泛使用的术语是鲁棒性，其可以被定义为计算机系统或算法处理错误和故障的能力（可能由诸如程序员的错误或格式不正确的用户输入等各种因素引起）。强大的系统可以优雅地处理这些不必要的情况。软件工程师可以通过各种方法来实现鲁棒性，例如测试不同类型输入的代码，但是一般来说，为了实现稳健性（和高度凝聚力），程序员遵循一套规则和原则来更好地组织面向对象的程序。一个这样的原则是单一的责任原则。



## Single Responsibility Principle

单一责任原则围绕声称某个代码模块（通常是一个类）应该仅对软件提供的功能的一部分负责。在软件工程书籍中，这有时也被定义如下：模块只能有一个理由来改变。这意味着在程序中执行了一个关切的划分，每个关注的方法都应该被一个类完全封装。现在很明显，这种方法有助于高度凝聚力 - 因为与相同关注点相关的方法（功能的相同部分）将成为同一类的成员和鲁棒性 - 因为这样可以减少错误的可能性。此外，如果发生错误，程序员将更有可能找到原因，最后解决问题。

单一责任原则建立在面向对象编程的基本概念之一，即所谓的分裂与征服原则 - 通过解决其多个子问题来解决问题。这种方法阻止了“神对象”的创建 - “知道太多或太多”的对象。

你写的类不应该是瑞士军刀。他们应该做一件事，一件事情好。
### (Bad) Example

我们来考虑Java中的这个经典例子 - “可以自己打印的对象”。

```java
class Text {
    String text;
    String author;
    int length;
    String getText() { ... }
    void setText(String s) { ... }
    String getAuthor() { ... }
    void setAuthor(String s) { ... }
    int getLength() { ... }
    void setLength(int k) { ... }
    /*methods that change the text*/
    void allLettersToUpperCase() { ... }
    void findSubTextAndDelete(String s) { ... }
    /*method for formatting output*/
    void printText() { ... }
}
```

乍一看，这个类可能正确写入。然而，它与单一责任原则相矛盾，因为它有多个原因要改变：我们有改变文本本身的方法，并为用户打印文本。如果调用了这些方法中的任何一个，那么类将会改变。这也不好，因为它将课程的逻辑与演示文稿相结合。
### Better Example

解决这个问题的一种方法是写另一个类，其唯一关心的是打印文本。这样，我们将分离类的功能和“美容”部分。
```java



class Text {
    String text;
    String author;
    int length;
    String getText() { ... }
    void setText(String s) { ... }
    String getAuthor() { ... }
    void setAuthor(String s) { ... }
    int getLength() { ... }
    void setLength(int k) { ... }
    /*methods that change the text*/
    void allLettersToUpperCase() { ... }
    void findSubTextAndDelete(String s) { ... }
}
class Printer {
    Text text;
    Printer(Text t) {
       this.text = t;
    }
    void printText() { ... }
}

```


## Summary

在第二个例子中，我们分两个类之间编辑文本和打印文本的职责。您可以注意到，如果发生错误，调试将更容易，因为识别出错的地方不会那么困难。此外，由于您修改了较小部分的代码，所以不小心引入软件错误的风险较小。
即使这个例子不是很明显（因为它很小），这种方法可以让你看到“更大的图片”，而不是在代码中丢失自己;它使程序更容易升级和扩展，没有类太广泛，代码变得混乱。
### Single Responsibility Principle in Spring
当您使用Spring组件和编码来更加舒适地支持Spring中的控制和依赖注入反转时，您将发现自己的类将自然遵循单一责任原则。在常规Spring应用程序中我经常看到的典型违反单一责任原则是控制器操作中丰富的代码。我看到Spring控制器获取JDBC连接以调用数据库。这明显违反了单一责任原则。控制器对象没有与数据库进行业务交互。控制器也没有实现其他业务逻辑的业务。在实践中，您的控制器方法应该非常简单和轻便。数据库调用和其他业务逻辑属于服务层。