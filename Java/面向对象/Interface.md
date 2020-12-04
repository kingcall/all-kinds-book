### Interface in Java With Examples

**Interfaces** help in achieving full [abstraction in Java](https://www.netjstech.com/2015/04/abstraction-in-java.html), as using interface in Java, you can specify what a [class](https://www.netjstech.com/2015/04/class-in-java.html) should do, but how class does it is not specified.

Interfaces in Java looks syntactically similar to classes, but they differ in many ways-

- Interfaces don't have instance variables, which means interfaces don't have a state.
- In interfaces methods are declared with out any body. They end with a semicolon.
- An interface in Java **can't be instantiated**, which also means that **interfaces can't have constructors**.
- An interface is implemented by a class, not extended.
- An interface itself can extend multiple interfaces.

**Please note that** Java 8 onward, it is possible to add a [default implementation to a method](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) in an interface as well as [interface static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) and even [Private Methods in Java Interface](https://www.netjstech.com/2018/10/private-methods-in-java-interface.html) Java 9 onward, here we'll discuss interfaces in its normal form.



### General form of interface in Java

```
access_modifier interface name {
 type final_var1 = value;
 type final_var2 = value;
 ----
 ----
 return_type method_name1(param_list);
 return_type method_name2(param_list);
 ----
 ----
}
```

If no [access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) is specified, that **means default access** where interface is available to other members of the [package](https://www.netjstech.com/2016/07/package-in-java.html) in which it is declared. When interface is **declared as public**, it can be used by any program.

For top level interfaces only these two access modifiers (default and public) are permitted, there are **nested interfaces** which can be declared as public, private or protected. We'll discuss nested interfaces later in the post.

All the variables declared inside an interface in Java **are implicitly public, final and static**, which means they can't be changed by the implementing class. They should **also be initialized**.

All the methods with in an interface in Java are **implicitly abstract methods** and should be implemented by a class that is implementing the interface. Methods in an interface are **also implicitly public**.

Note that Java 9 onward you can also add private methods to a Java interface. Please refer [Private Methods in Java Interface](https://www.netjstech.com/2018/10/private-methods-in-java-interface.html) to know more about adding private methods in a Java interface.

### Java Interface example

```
public interface MyInterface {
 int i = 10;
 Number method1();
 void method2(String Id);
}
```

Here is an interface **MyInterface** which has one variable i, which is public static final implicitly. There are 2 methods, one has no return type, one returns Number.



### Implementing an Interface in Java

In Java, class uses **implements** keyword to implement an interface.

```
public class MyClass implements MyInterface { 
 public Integer method1() {
  System.out.println("in method 1" + i);  
  return null;
 }

 public void method2(String Id) {
  System.out.println("in method 2");  
 }
 public static void main(String[] args) {
  
 }
}
```

When implementing methods defined in interfaces there are several important points -

- A class can implement more than one interface, in that case interfaces are separated with a comma.

  ```
   class class_name implements interface 1, interface 2{
   }
   
  ```

- The methods of an interface that are implemented by a class must be declared public i.e. visibility can't be reduced.

- The signature of the interface method must remain same while implementing it. In case of return type subclass of the return type as defined in interface can also be used. As in the above program method1 has the return type Number where as in the implemented method in the class has the return type Integer. It is permissible as Integer is the sub-class of Number.

- The initialized variable in the interface **is constant** and it is not allowed to change that value in the implementing class. As, in the above program there is a variable i with value 10. If we try to change that value to 20 in the implementing class it will throw compiler error "The final field MyInterface.i cannot be assigned".

- Any number of classes can implement an interface and each class is free to provide their own implementation. That's how using interfaces, **Java fully utilizes "one interface, multiple methods"** aspect of polymorphism.

- A class can extend only one class, but a class can implement many interfaces. Which means [multiple inheritance is not allowed in Java](https://www.netjstech.com/2017/06/why-no-multiple-inheritance-in-java-diamond-problem.html)

- An interface can extend another interface, similar to the way that a class can extend another class.

### Extending an interface in Java

Just like class, if an interface in Java is inheriting from another interface it **uses extends keyword**.
An interface, unlike class, can extend more than one interface. If a class implements an interface that inherits another interface, it must provide implementation for all the methods that are there in the inheritance chain of interface.

**Extending an interface Java Example**

```
// Interface
public interface MyInterface {
 int i = 10;
 Number method1();
 void method2(String Id);
}
// extending interface
interface B extends MyInterface{
 void method3();
}

// class implements all methods of MyInterface and B
public class MyClass implements B {
 
 public Integer method1() {
  System.out.println("in method 1" + i);  
  return null;
 }

 public void method2(String Id) {
  System.out.println("in method 2");  
 }

 public void method3() {
  System.out.println("in method 3");  
 }
 public static void main(String[] args) {
  
 }
}
```

It can be seen that the class Myclass has to implement all the methods in the inheritance chain of the interface.

### Partial implementation of interface by a class

If a class implements an interface but does not implement all the methods of that interface then that class must be declared as [abstract](https://www.netjstech.com/2015/04/abstract-class-in-java.html).

```
public interface MyInterface {
 void method1();
 String method2(String Id);
}
```

[![implemnting java interface methods](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/16:57:02-abstract%252Bclass-1.png)](https://1.bp.blogspot.com/-KIzZfR0PVnY/VUDv0YP0TzI/AAAAAAAAAGQ/qBX_J1PA6GU/s1600/abstract%2Bclass-1.png)

**Compiler error that class must implement methods declared in MyInterface interface.**

But we can declare the class as abstract in that case

```
public abstract class AbstractClassDemo implements MyInterface {
 public static void main(String[] args) {
  System.out.println();

 }
}
```

### Nested Interfaces in Java

An interface or a class can have another interface. Such an interface is known as **nested interface** or a **member interface** in Java.
A nested interface can be declared as public, private or protected. When a nested interface is used outside, it must be used as a fully qualified name i.e. must be qualified by the name of the class or interface of which it is a member.

**Java nested interface Example**

```
// Class with nested interface
class A{
 public interface TestInterface{
  void displayValue(String value);
 }
}

// class implementing the nested interface
class B implements A.TestInterface{
 public void displayValue(String value) {
  System.out.println("Value is " + value);
 }
}

public class MyClass{ 
 public static void main(String[] args) {
  // reference of class B assigned to nested interface
  A.TestInterface obRef = new B();
  obRef.displayValue("hello");
 }
}
```

**Output**

```
Value is hello
```

### Interface and run time polymorphism

As we already know that there can't be an object of an interface, but interface can be used to create object references. As [run time polymorphism in Java](https://www.netjstech.com/2015/04/polymorphism-in-java.html) is implemented through the use of super class reference thus interface can be used to provide super class reference which holds references of sub-classes at run time and provide the appropriate functionality.

**Run time polymorphism using interface Java Example**

Let's assume that in an application there is a need to handle payment done through several modes like; cash, cheque, credit card etc. and based on the mode of the payment the functionality may be different.
This can be achieved through an interface where the interface defines a method payment and then several classes implement that interface and provide the functionality for the payment method according to the business needs. That's how using interfaces, Java fully utilizes "one interface, multiple methods" aspect of polymorphism.

```
public interface PaymentInt {
 public void payment(double amount);
}
// Cash Payment implementation of Payment interface
class CashPayment implements PaymentInt{
 // method implementation according to cash payment functionality
 public void payment(double amount) {
  System.out.println("Cash Payment of amount " + amount);
 }
}

//Cheque Payment implementation of Payment interface
class ChequePayment implements PaymentInt{
 // method implementation according to cheque payment functionality
 public void payment(double amount) {
  System.out.println("Cheque Payment of amount " + amount);  
 }
}

//CreditCard Payment implementation of Payment interface
class CreditCardPayment implements PaymentInt{
 // method implementation according to credit card payment functionality
 public void payment(double amount) {
  System.out.println("CreditCard Payment of amount " + amount);
 }
}

public class PaymentDemo {
 public static void main(String[] args) {
  // Payment interface reference holding the CashPayment obj
  PaymentInt paymentInt = new CashPayment();
  paymentInt.payment(134.67);
  // Payment interface reference holding the CreditCardPayment obj
  paymentInt = new CreditCardPayment();
  paymentInt.payment(2347.89);
  // Payment interface reference holding the ChequePayment obj
  paymentInt = new ChequePayment();
  paymentInt.payment(1567.45);
 }
}
```

**Output**

```
Cash Payment of amount 134.67
CreditCard Payment of amount 2347.89
Cheque Payment of amount 1567.45
```

It can be seen how at run time reference is changed and the appropriate payment method is called.

**Points to note-**

- Interfaces help in achieving full abstraction in Java.
- For top level interfaces only default and public access modifiers are permitted.
- All the variables in interface are implicitly public, static and final.
- All the methods in an interface are implicitly public and abstract.
- The methods of an interface that are implemented by a class must be declared public.
- If a class implements an interface but does not implement all the methods of that interface then that class must be declared as abstract.
- A class can extend only one class, but implement many interfaces.
- An interface, unlike class, can extend more than one interface.
- An interface or a class can have another interface. Such an interface is known as nested interface or a member interface.
- Any number of classes can implement an interface and each class is free to provide their own implementation. That's how using interfaces, Java fully utilizes "one interface, multiple methods" aspect of polymorphism.
- With Java 8, it is possible to add a default implementation to a method in an interface.

That's all for this topic **Interface in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!