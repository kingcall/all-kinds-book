### Interface Static Methods in Java

Java 8 has added support for [interface default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) as well as **interface static methods** which is one of the important change in Java 8 along with the addition of [lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) and [stream API](https://www.netjstech.com/2016/11/stream-api-in-java-8.html). In this post we'll see how to use interface static methods in Java and what are the rules for using them

**Table of contents**

1. [Static method in Java interface](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html#Staticmethod)
2. [Java interface static method example](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html#StaticmethodExp)
3. [Interface static methods are not inherited](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html#Staticmethodinherit)
4. [Hiding interface static method](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html#StaticmethodHiding)
5. [Advantages of Java interface static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html#StaticmethodAdvantage)



### Static method in Java interface

Like [static methods](https://www.netjstech.com/2015/04/static-in-java.html) in a [class](https://www.netjstech.com/2015/04/class-in-java.html), now we can write static method in interfaces too. Static methods in an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) can be called independently of any [object](https://www.netjstech.com/2015/04/object-in-java.html) just like how static methods are called in a class.

**General form of calling the interface static method**

Static methods in an interface are called by using the interface name preceding the method name.

```
InterfaceName.staticMethodName;
```



### Java interface static method example

```
public interface MyInterface {
 int method1();
 // default method, providing default implementation
 default String displayGreeting(){
    return "Hello from MyInterface";
 }
 //static method
 static int getDefaultAmount(){
    return 0;
 }
}
public class MyClass{ 
  public static void main(String[] args) { 
    int num = MyInterface.getDefaultAmount();
    System.out.println("num " + num);
  } 
}
```

**Output**

```
num 0
```

In the example code interface **MyInterface** has one static method **getDefaultAmount()**. Note that **MyClass** is not even implementing the interface MyInterface still static method of the **MyInterface** can be called from MyClass using the call **MyInterface.getDefaultAmount()**;. This is because, no implementation or reference of the interface is required to call the static method.

### Interface static methods are not inherited

Static interface methods are not inherited by-

- Implementing classes
- Extending interfaces

Here interface B is extending mentioned interface MyInterface, but it can not access the static method of interface MyInterface.

```
interface B extends MyInterface{ 
  default String displayGreeting(){
    B.getDefaultAmount(); // Compiler Error 
    return "Hello from MyInterface";
  }
}
```

Same way even if **MyClass** implements **MyInterface** still it can not access static method of MyInterface, either by using the class name or by using the object reference.

```
public class MyClass implements MyInterface{
  // provides implementation for the non-default method
  // of the interface
  @Override
  public int method1() {
    return 10;
  }
  //Overriding the default method of MyInterface
  public String displayGreeting(){
    return MyInterface.super.displayGreeting();
  }
 
  public static void main(String[] args) {
    MyInterface myInt = new MyClass();

    int num = MyInterface.getDefaultAmount();
    System.out.println("num " + num);
    MyClass.getDefaultAmount(); // Compiler error
    myInt.getDefaultAmount();// Compiler error
  } 
}
```

### Hiding interface static method

Though implementing class can't provide implementation for the static methods of an interface but the implementing class can hide the interface static method in Java by providing method with same signature in the implementing class.

```
public interface MyInterface {
  int method1();
  // default method, providing default implementation
  default String displayGreeting(){
     return "Hello from MyInterface";
  }
  static int getDefaultAmount(){
     return 0;
  }
}
```

**Implementing class**

```
public class MyClass implements MyInterface{
  // provides implementation for the non-default method
  // of the interface
  @Override
  public int method1() {
    return 10;
  }
  //Overriding the default method of MyInterface
  public String displayGreeting(){
    return MyInterface.super.displayGreeting();
  }
  // static method
  public static int getDefaultAmount(){
    return 5;
  }
 
  public static void main(String[] args) {
    MyInterface myInt = new MyClass();

    int num = MyInterface.getDefaultAmount();
    System.out.println("num - Interface " + num);
    System.out.println("num - Class " + MyClass.getDefaultAmount());  
  } 
}
```

**Output**

```
num - Interface 0
num - Class 5
```

Here getDefaultAmount() method is provided in the MyClass class also which hides the interface static method.

### Advantages of Java interface static methods

- Interface static methods can be used for providing utility methods.
- With Interface static methods we can secure an implementation by having it in static method as implementing classes can't override them. Though we can have a method with same signature in implementing class but that will hide the method won't override it.

That's all for this topic **Interface Static Methods in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!