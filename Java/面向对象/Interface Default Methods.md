### Interface Default Methods in Java

Java 8 has added support for **default methods** as well as **static methods** to interfaces. In this post we'll talk about **interface default methods in Java**.

- Refer [interface static methods in Java 8](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) to know more about interface static methods in Java.

**Table of contents**

1. [Need for interface default methods in Java](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodsRequire)
2. [Default method in Java interface](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodJava)
3. [Java Interface Default Method Example](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodExp)
4. [Interface default methods and multiple inheritance issues](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodinheritance)
5. [Use of Super with interface default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodWithSuper)
6. [Difference between Interface with default method and abstract class in Java](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html#DefaultMethodVsAbstract)



### Need for interface default methods in Java

There was a problem with interfaces in Java that they were **not open to extension**, which means if there was a need to add new method to an interface it would have **broken the existing implementations** of that interface. Thus it was imperative that all the classes implementing that interface had to provide implementation for the newly added method, even if the method was not needed. Thus *Java interfaces were not easy to evolve*.

**One example** that comes to mind is **Java MapReduce API for Hadoop**, which was changed in 0.20.0 release to favour abstract classes over interfaces, since they are easier to evolve. Which means, a new method can be added to [abstract class](https://www.netjstech.com/2015/04/abstract-class-in-java.html) (with default implementation), with out breaking old implementations of the class.

### Default method in Java interface

Java 8 onward it is possible to add default method in Java interfaces too, thus making them easier to evolve. With the addition of default method to an interface, addition of new method, even to an interface will not break the pre-existing code.

Interface default method should be used for backward compatibility. Whenever there is a need to add new methods to an existing interface, default methods can be used so that existing implementing classes don't break and not forced to provide implementation for the newly added methods.

### Java Interface Default Method Example

An interface default method in Java is defined the same way a method will be defined in a class. One difference is that in interface, default method is preceded by the keyword **default**.

```
public interface MyInterface {
  int method1();
  // default method, providing default implementation
  default String displayGreeting(){
    return "Hello from MyInterface";
  }
}
public class MyClass implements MyInterface{
  // provides implementation for the non-default method
  // of the interface
  @Override
  public int method1() {
    return 10;
  }
  public static void main(String[] args) {
    MyInterface myInt = new MyClass();
    System.out.println("Value " +  myInt.method1());
    // Calls the default method provided by interface itself
    System.out.println("Greeting " + myInt.displayGreeting());
  }
}
```

**Output**

```
Value 10
Greeting Hello from MyInterface
```

It can be seen that in MyInterface interface there is a default method **displayGreeting()**. Here implementing class is not overriding and providing its own implementation of the default method thus the default is used.

Implementing class **can provide its own implementation** of a default method by [overriding](https://www.netjstech.com/2015/04/method-overriding-in-java.html) it. If we use the same interface and class as used above and override the **displayGreeting** method to change the implementation.

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
    return "Hello from MyClass";
  }
 
  public static void main(String[] args) {
    MyInterface myInt = new MyClass();
    System.out.println("Value " +  myInt.method1());
    // Calls the default method provided by interface itself
    System.out.println("Greeting " + myInt.displayGreeting());
  }
}
```

**Output**

```
Value 10
Greeting Hello from MyClass
```

It can be seen how output has changed and now the **displayGreeting()** method of the implementing class is used.

### Interface default methods and multiple inheritance issues

With the inclusion of default methods, interfaces may have [multiple inheritance issues](https://www.netjstech.com/2017/06/why-no-multiple-inheritance-in-java-diamond-problem.html). Though an interface can't hold state information (interface can't have instance variables), so state information can't be inherited, but behavior in form of default method may be inherited which may cause problems. Let's see it with **an example**-

Let's assume there are two interfaces **A** and **B** and both have default method **displayGreeting()**. There is a class **MyClass** which implements both these interfaces A and B.

**Now consider the scenarios**-

- What implementation of default method displayGreeting() will be called when MyClass is implementing both interfaces A and B and **not overriding** the displayGreeting() method.
- What implementation of displayGreeting() will be called when MyClass is implementing both interfaces A and B **and overriding** the displayGreeting method and providing its own implementation.
- If interface A is inherited by interface B, what will happen in that case?

To handle these kinds of scenarios, Java defines a **set of rules for resolving default method conflicts**.

- If implementing class overrides the default method and provides its own functionality for the default method then the method of the class takes priority over the interface default methods.
  **As Example -** If MyClass provides its own implementation of displayGreeting(), then the overridden method will be called not the default method in interface A or B.
- When class implements both interfaces and both have the same default method, also the class is not overriding that method then the error will be thrown.
  *"Duplicate default methods named displayGreeting inherited from the interfaces"*
- In case when an interface extends another interface and both have the same default method, the inheriting interface default method will take precedence. Thus, if interface B extends interface A then the default method of interface B will take precedence.

### Use of Super with interface default methods

As stated above when class implements 2 interfaces and both have the same default method then the class has to provide an implementation of its own for the default method otherwise error will be thrown. From the implementing class if you want to call the default method of interface A or interface B, then from the implementation in the class you can call the default method of the specific interface using the [super keyword](https://www.netjstech.com/2015/04/super-in-java.html).

Syntax for calling the default method of the specific interface using super is as follows-

```
InterfaceName.super.methodName();
```

For example, if you want to call the default method of interface A.

```
public interface A {
  int method1();
  // default method, providing default implementation
  default String displayGreeting(){
    return "Hello from interface A";
  }
}

interface B{
  // default method, providing default implementation
  default String displayGreeting(){
    return "Hello from Interface B";
  }
}
public class MyClass implements A, B{
  // provides implementation for the non-default method
  // of the interface
  @Override
  public int method1() {
     return 10;
  }
  //Overriding the default method of MyInterface
  public String displayGreeting(){
     return A.super.displayGreeting();
  }
 
  public static void main(String[] args) {
     A myInt = new MyClass();
     System.out.println("Value " +  myInt.method1());
     // Calls the default method provided by interface itself
     System.out.println("Greeting " + myInt.displayGreeting());
  }
}
```

**Output**

```
Value 10
Greeting Hello from interface A
```

It can be seen from **displayGreeting()** method of the class, using super, displayGreeting() method of interface A is called here.

### Difference between Interface with default method and abstract class in Java

With interfaces also providing default methods the lines between the [abstract class](https://www.netjstech.com/2015/04/abstract-class-in-java.html) and [interface](https://www.netjstech.com/2015/05/interface-in-java.html) blurs a bit in Java. But there are still certain differences between them.

- Abstract class can have constructor, instance variables but interface can't have any of them.
- interfaces with default methods also cannot hold state information.

**Points to note-**

- With Java 8 interface default methods had been added in order to make interfaces easy to evolve.
- With interface default method, classes are free to use the default method of the interface or override them to provide specific implementation.
- With interface default methods there may be multiple inheritance issue if same default method is provided in more than one interfaces and a class is implementing those interfaces.
- super can be used from the implementing class to invoke the interface default method. It's general form is InterfaceName.super.methodName();
- Interface default methods don't change the basic traits of interface, interface still can't hold state information, it is still not possible to create an instance of an interface by itself.

That's all for this topic **Interface Default Methods in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!