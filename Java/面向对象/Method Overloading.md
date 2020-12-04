### Method Overloading in Java

When two or more methods with in the same class or with in the parent-child relationship classes, have the same name but the parameters are different in types or number, the methods are said to be overloaded. This process of overloaded methods is known as **method overloading** in Java.

Method overloading is one of the ways through which java supports [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html). In fact method overloading is an example of static polymorphism or compile time polymorphism.

When an overloaded method is called, which of the overloaded method is to be called is determined through-

- Types of the parameters- For example test(int a, int b) and test(float a, int b). In these overloaded methods name is same but the types of the method parameters differ.
- Number of the parameters- For example test(int a, int b) and test(int a, int b, int c). In these overloaded methods name is same but the number of parameters is different.

**Please note that** return type of the methods may be different but **that alone is not sufficient** to determine the method that has to be called.

**Table of contents**

1. [Examples of method overloading in Java](https://www.netjstech.com/2015/04/method-overloading-in-java.html#methodoverloadingexp)
2. [Method overloading in Java and automatic type conversion](https://www.netjstech.com/2015/04/method-overloading-in-java.html#methodoverloadingconv)
3. [Overloaded method in case of Inheritance](https://www.netjstech.com/2015/04/method-overloading-in-java.html#methodoverloadingInheritance)
4. [Benefit of Method Overloading](https://www.netjstech.com/2015/04/method-overloading-in-java.html#methodoverloadingBenefit)



### Examples of method overloading in Java

1- Method overloading example with methods having different types of parameters. In the Java example there are two overloaded methods, one having two parameters and both are int where as other method has one int and one double parameter.

```
public class OverloadingExample {
 // overloaded Method
 void overloadedMethod(int i, int j){
  System.out.println("In overloadedMethod with both int parameters- " + i);
 }
 
 // overloaded Method
 void overloadedMethod(int i, double j){
  System.out.println("In overloadedMethod with int and double parameters " + i + " " + j);
 }
 
 
 public static void main(String args[]){ 
  OverloadingExample obj = new OverloadingExample();
  obj.overloadedMethod(5, 7);
  obj.overloadedMethod(5, 103.78);
 }
}
```

**Output**

```
In overloadedMethod with both int parameters- 5
In overloadedMethod with int and double parameters 5 103.78
```

2- Method overloading example with methods having different number of parameters. In the Java example there are two overloaded methods, one having a single int parameter where as other method has one int and one String parameter.

```
public class OverloadingExample {
 // overloaded Method
 void overloadedMethod(int i){
  System.out.println("In overloadedMethod with int parameter- " + i);
 }
 
 // overloaded Method
 void overloadedMethod(int i, String s){
  System.out.println("In overloadedMethod with int and string parameters- " + i + " " + s);
 }
 
 
 public static void main(String args[]){ 
  OverloadingExample obj = new OverloadingExample();
  obj.overloadedMethod(5);
  obj.overloadedMethod(5, "Test");
 }
}
```

**Output**

```
In overloadedMethod with int parameter- 5
In overloadedMethod with int and string parameters- 5 Test
```

### Method overloading in Java and automatic type conversion

In java [automatic type promotion](https://www.netjstech.com/2015/04/java-automatic-numeric-type-promotion.html) may happen and it does have a role in how overloaded methods are called, let's see it with an example to have clarity.

```
public class OverloadingExample {
 // overloaded Method with 1 int param
 void overloadedMethod(int i){
  System.out.println("In overloadedMethod with one int parameter " + i);
 }
 
 // overloaded Method with 2 double params
 void overloadedMethod(double i, double j){
  System.out.println("In overloadedMethod with 2 double parameters " + i + " " + j);
 }
 
 
 public static void main(String args[]){ 
  OverloadingExample obj = new OverloadingExample();
  obj.overloadedMethod(5);
  obj.overloadedMethod(5.7, 103.78);
  obj.overloadedMethod(5, 10);
 }
}
```

Here notice the third call to the method
**obj.overloadedMethod(5, 10);**

It has 2 int params, but there is no overloaded method with 2 int params, in this case automatic type conversion happens and java promotes these 2 int parameters to double and calls **overloadedMethod(double i, double j)** instead.

### Overloaded method in case of Inheritance

In the case of parent-child relationship i.e. [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html) if both classes have the method with **same name and same number and type of parameters then it is considered as [method overriding](https://www.netjstech.com/2015/04/method-overriding-in-java.html)**, otherwise it is **method overloading**.

```
class Parent {
 private int i;
 // Constructor
 Parent(int i){
  this.i = i;
 }
 // Method with no param
 public void dispayData(){
  System.out.println("Value of i " + i);
 } 
}

class Child extends Parent{
 private int j;
 // Constructor
 Child(int i, int j){
  // Calling parent class constructor
  super(i);
  this.j = j;
 }
 // Overloaded Method with String param
 public void dispayData(String showMsg){
  System.out.println(showMsg + "Value of j " + j);
 } 
}

class OverloadDemo{
 public static void main(String args[]){ 
  Child childObj = new Child(5, 10);
  // This call will invoke the child class method
  childObj.dispayData("in Child class displayData ");
  // This call will invoke the parent class method
  childObj.dispayData();
 }
}
```

**Output of the program would be**

```
in Child class displayData Value of j 10
Value of i 5
```

### Benefit of Method Overloading

If we don't have method overloading then the methods which have same functionality but parameters type or number differs have to have different names, thus **reducing readability of the program**.

**As example** - If we have a method to add two numbers but types may differ then there will be different methods for different types like **addInt(int a, int b)**, **addLong(long a, long b)** and so on with out overloaded methods.

With method overloading we can have the same name for the overloaded methods thus increasing the readability of the program.

That's all for this topic **Method Overloading in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!