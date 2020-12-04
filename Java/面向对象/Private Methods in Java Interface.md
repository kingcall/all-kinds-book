### Private Methods in Java Interface

Interfaces in Java were living the life of having public static final fields and public abstract methods till Java 7 but got a makeover with some new features added to them later namely-

- Capability to add **default methods** to interface from Java 8 onward.
- Capability to add **static methods** to interface from Java 8 onward.
- Capability to add **private methods** to interface from Java 9 onward.

In this post we’ll see how to add private methods to a [Java interface](https://www.netjstech.com/2015/05/interface-in-java.html), what is the reason to include private methods to an interface.

### Private methods in Java interface

Private methods in Java interfaces are defined using **private modifier** the same way it is done for a Java class.

```
private methodName(argument_List){
 ..
 ..
}
```

Private methods in Java interface can be both [static](https://www.netjstech.com/2015/04/static-in-java.html) and non-static.

### Reason to include private methods to an interface in Java

Java 8 included [interface default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [interface static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) to an interface with that inclusion it became possible to write method bodies with in an interface but a new problem was observed.

Let’s say you have 2 methods and both of them share some code and you want to write that code as a separate method which can be called from both the methods. In Java 8 that method with common code could be default or static.

Here note that both default and static methods with in a Java interface are public by default meaning the method having common code would also be implicitly public. Which means any class implementing the interface can access this method too. But that method has some code which makes sense with in the interface, calling it from any other class doesn’t make sense and it is also against the [encapsulation OOPS concept](https://www.netjstech.com/2015/04/encapsulation-in-java.html) because access is given to some method which is not at all required.

Let’s try to understand this scenario in Java 8 with an example-

```
public interface TestInterface {
 default void defMethod1(){
  sharedCode();
  System.out.println("In default method 1");
 }
 
 default void defMethod2(){
  sharedCode();
  System.out.println("In default method 2");
 }
 
 default void sharedCode(){
  System.out.println("In sharedCode, invoking it on its own"
    + " doesn't make much sense");
 }
}
```

As you can see in the Interface there are two default methods defMethod1() and defMethod2() in the interface. Common code executed by both the methods is kept in a separate default method to avoid duplication of the code.

But the problem with this interface is that any class implementing this interface could access sharedCode() method too as all the methods were public by default till Java 8.

```
public class TestClass implements TestInterface {
  public static void main(String[] args) { 
   TestClass obj = new TestClass();
   obj.sharedCode();
  }
}
```

**Output**

```
In sharedCode, invoking it on its own doesn't make much sense
```

### Interface private methods Java 9 onward

With the new feature of private methods in interfaces from Java 9 onward such methods (as shown in the above example) can be written as private methods which are not visible outside the interface. That way code redundancy can be avoided while keeping the access to the method restricted.

The example showed above can use [private access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) (Java 9 onward) with the sharedCode() method with in the interface to keep access to it restricted.

```
public interface TestInterface {
 default void defMethod1(){
  sharedCode();
  System.out.println("In default method 1");
 }
 
 default void defMethod2(){
  sharedCode();
  System.out.println("In default method 2");
 }
 
 private void sharedCode(){
  System.out.println("In sharedCode, invoking it on its own"
    + " doesn't make much sense");
 }
}
```

Now trying to access sharedCode() method from a class implementing this interface will result in compile time error “The method sharedCode() is undefined for the type TestClass”.

```
public class TestClass implements TestInterface {
  public static void main(String[] args) { 
   TestClass obj = new TestClass();
   obj.sharedCode(); // Compiler error
  }
}
```

Now you can invoke default method from the implementing class which in turn invokes the private method.

```
public class TestClass implements TestInterface {
  public static void main(String[] args) { 
   TestClass obj = new TestClass();
   obj.defMethod1();
  }
}
```

**Output**

```
In sharedCode, invoking it on its own doesn't make much sense
In default method 1
```

As you can see now private method can be accessed through methods of the interfaces only.

### Rules for private methods in Java interfaces

The usage of private methods in the Java interfaces is guided by the following rules-

1. Private methods in an interface can't be abstract they should have method body. Trying to define a private method as a regular public abstract method in an interface results in the error "This method requires a body instead of a semicolon"
2. If you want to invoke a private method from a static method in an interface then you should write a private static method. From a static context you cannot access a non-static method it will give you an error “Cannot make a static reference to the non-static method”.
3. A default method in an interface can invoke both static and non-static private methods with in an interface.

That's all for this topic **Private Methods in Java Interface**. If you have any doubt or any suggestions to make please drop a comment. Thanks!