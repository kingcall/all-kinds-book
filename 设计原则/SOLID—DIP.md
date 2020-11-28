[toc]

## 依赖反转原则（DIP）
- 高层模块（high-level modules）不要依赖低层模块（low-level）。高层模块和低层模块应该通过抽象（abstractions）来互相依赖。除此之外，抽象（abstractions）不要依赖具体实现细节（details），具体实现细节（details）依赖抽象（abstractions）。
- 在调用链上，调用者属于高层，被调用者属于低层。在平时的业务代码开发中，高层模块依赖底层模块是没有任何问题的。实际上，这条原则主要还是用来指导框架层面的设计，

### 正确理解
- 依赖倒置原则概念是高层次模块不依赖于低层次模块。看似在要求高层次模块，实际上是在规范低层次模块的设计。低层次模块提供的接口要足够的抽象、通用，在设计时需要考虑高层次模块的使用种类和场景。
- 明明是高层次模块要使用低层次模块，对低层次模块有依赖性。现在**反而低层次模块需要根据高层次模块来设计**，出现了「倒置」的显现
- 把原本的高层建筑依赖底层建筑“倒置”过来，变成底层建筑依赖高层建筑。高层建筑决定需要什么，底层去实现这样的需求，但是高层并不用管底层是怎么实现的。这样就不会出现前面的“牵一发动全身”的情况。
- 控制反转（Inversion of Control） 就是依赖倒置原则的一种代码设计的思路。具体采用的方法就是所谓的依赖注入（Dependency Injection）。

#### 优势
1. 低层次模块更加通用，适用性更广，能更好的适应业务需求的变化
2. 高层次模块没有依赖低层次模块的具体实现，方便低层次模块的替换

## 控制反转
- "控制反转"控制反转的英文翻译是 Inversion Of Control，缩写为 IOC。
- 框架提供了一个可扩展的代码骨架，用来组装对象、管理整个执行流程。程序员利用框架进行开发的时候，只需要往预留的扩展点上，添加跟自己业务相关的代码，就可以利用框架来驱动整个程序流程的执行。
- 这里的“控制”指的是对程序执行流程的控制，而“反转”指的是在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程可以通过框架来控制。流程的控制权从程序员“反转”到了框架。
- 实现控制反转的方法有很多，除了类似于模板设计模式的方法之外，还有依赖注入等方法，所以，控制反转并不是一种具体的实现技巧，而是一个比较笼统的设计思想，一般用来指导框架层面的设计

### 例子
```

public abstract class TestCase {
  public void run() {
    if (doTest()) {
      System.out.println("Test succeed.");
    } else {
      System.out.println("Test failed.");
    }
  }
  
  public abstract boolean doTest();
}

public class JunitApplication {
  private static final List<TestCase> testCases = new ArrayList<>();
  
  public static void register(TestCase testCase) {
    testCases.add(testCase);
  }
  
  public static final void main(String[] args) {
    for (TestCase case: testCases) {
      case.run();
    }
  }
```
- 我们只需要在框架预留的扩展点，也就是 TestCase 类中的 doTest() 抽象函数中，填充具体的测试代码就可以实现之前的功能了，完全不需要写负责执行流程的 main() 函数了
- 这个例子，就是典型的通过框架来实现“控制反转”的例子。框架提供了一个**可扩展的代码骨架**，用来组装对象、管理整个执行流程。程序员利用框架进行开发的时候，只需要往预留的扩展点上，添加跟自己业务相关的代码，就可以利用框架来驱动整个程序流程的执行。

## 依赖注入（DI）
- 依赖注入跟控制反转恰恰相反，它是一种具体的编码技巧。依赖注入的英文翻译是 Dependency Injection，缩写为 DI
- - 不通过 new() 的方式在类内部创建依赖类对象，而是**将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（或注入）给类使用**。
- 通过依赖注入的方式来将依赖的类对象传递进来，这样就提高了代码的扩展性，我们可以灵活地替换依赖的类,其实就是开闭原则
- 这个概念，有一个非常形象的说法，那就是：依赖注入是一个标价 25 美元，实际上只值 5 美分的概念。也就是说，这个概念听起来很“高大上”，实际上，理解、应用起来非常简单。
- 它是编写可测试性代码最有效的手段。

### 例子
```

// 非依赖注入实现方式
public class Notification {
  private MessageSender messageSender;
  
  public Notification() {
    this.messageSender = new MessageSender(); //此处有点像hardcode
  }
  
  public void sendMessage(String cellphone, String message) {
    //...省略校验逻辑等...
    this.messageSender.send(cellphone, message);
  }
}

public class MessageSender {
  public void send(String cellphone, String message) {
    //....
  }
}
// 使用Notification
Notification notification = new Notification();

// 依赖注入的实现方式
public class Notification {
  private MessageSender messageSender;
  
  // 通过构造函数将messageSender传递进来
  public Notification(MessageSender messageSender) {
    this.messageSender = messageSender;
  }
  
  public void sendMessage(String cellphone, String message) {
    //...省略校验逻辑等...
    this.messageSender.send(cellphone, message);
  }
}
//使用Notification
MessageSender messageSender = new MessageSender();
Notification notification = new Notification(messageSender);
```

### 依赖注入框架
- 在实际的软件开发中，一些项目可能会涉及几十、上百、甚至几百个类，类对象的创建和依赖注入会变得非常复杂
- 如果这部分工作都是靠程序员自己写代码来完成，容易出错且开发成本也比较高。
- 而对象创建和依赖注入的工作，本身跟具体的业务无关，我们完全可以抽象成框架来自动完成。
- 现成的依赖注入框架有很多，比如 Google Guice、Java Spring、Pico Container、Butterfly Container 等

### 对比基于接口编程
- 但是通常情况下**依赖注入**加上基于接口威力更大，接口把依赖抽象化，传入不同对接口的实现可以很好实现对扩展开放的原则,也就是说它们可以相辅相成

#### 相同点
- "基于接口而非实现编程"与"依赖注入"的联系是二者都是从外部传入依赖对象而不是在内部去new一个出来

#### 区别
- 区别是"基于接口而非实现编程"强调的是"接口"，强调依赖的对象是接口，而不是具体的实现类,而"依赖注入"不强调这个，类或接口都可以，只要是从外部传入不是在内部new出来都可以称为依赖注入。
- 依赖注入是一种具体编程技巧，关注的是对象创建和类之间关系，目的提高了代码的扩展性，我们可以灵活地替换依赖的类。.基于接口而非实现编程是一种设计原则，关注抽象和实现，上下游调用稳定性，目的是降低耦合性，提高扩展性。

#### 总结
- 基于接口而非实现编程，是一种指导编码的思想。依赖注入是它的一种具体应用