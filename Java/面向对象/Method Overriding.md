### Method Overriding in Java

In a parent-child relation between classes, if a method in subclass has the same signature as the parent class method then the method is said to be overridden by the subclass and this process is called **method overriding** in Java.

### Rules for method overriding in Java

1. Overridden method must have the same name and same number and type of parameters in both parent and child classes.
2. In the case of parent-child relationship if two classes have the method with same name and signature then only it is considered as method overriding, otherwise it is [method overloading](https://www.netjstech.com/2015/04/method-overloading-in-java.html).
3. The visibility of [access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) can't be reduced in the child class method. For example if the method is protected in the parent class then the overridden child class method can have the modifier as public (visibility can be increased) but not as private (reduced visibility).
4. In case of method overriding, return type of the child class method should be of the same type or the sub type of the parent class' method return type. Read more about it in this post- [Covariant Return Type in Java](https://www.netjstech.com/2015/05/covariant-return-type-in-java.html)
5. Java exception handling has certain rules in place for method overriding.
   If parent class method doesn't throw any exception then the child class method can throw only unchecked (runtime) exceptions not checked exceptions.
   If parent class method throws an exception then the child class method can throw the same exception, subtype exception of the exception declared in the superclass method or no exception at all.
   Read more about rules regarding method overriding with respect to exception handling in this post- [Java Exception Handling And Method Overriding](https://www.netjstech.com/2015/05/exception-handling-and-method-overriding-java.html)

### Method overriding in Java Example

In the example there is a super class **Parent** and a sub-class **Child**. In the super class there is a method **dispayData()** which is overridden in the sub-class to provide sub-class specific implementation.

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
 // Overridden Method
 public void dispayData(){
  System.out.println("Value of j " + j);
 } 
}

public class OverrideDemo {

 /**
  * @param args
  */
 public static void main(String[] args) {
  Child child = new Child(5, 7);
  child.dispayData();  
 }
}
```

**Output**

```
Value of j 7
```

It can be noticed here that when **displayData()** is invoked on an object of class Child, the overridden displayData() method of the child class is called.

### Benefits of method overriding in Java

**Method overriding** allows java to support run-time [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html) which in turn helps in writing more robust code and code reuse.

**Method overriding** also helps in hierarchical ordering where we can move from general to specific.

If we use the same example to **demonstrate run time polymorphism** here.

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
 // Overridden Method
 public void dispayData(){
  System.out.println("Value of j " + j);
 } 
}

public class OverrideDemo {

 /**
  * @param args
  */
 public static void main(String[] args) {
  Child child = new Child(5, 7);
  Parent parent = new Parent(8);
  // calling parent displayData method
  parent.dispayData();
  // parent holding the reference of child
  parent = child;
  // child class displayData method will be called
  parent.dispayData();  
 }

}
```

**Output**

```
Value of i 8
Value of j 7
```

It can be noticed here that initially parent object calls displayData() method of parent class, later at run time the reference is changed and the parent is assigned the reference of child class. Now, when the displayData() method is called on the parent object, it calls the overridden method of the class Child.

**Points to note-**

- If method in the sub-class has the same name and signature as the parent class method then only it is called method overriding otherwise it is method overloading.
- Method overriding allows java to support [run-time polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html).
- Method overriding helps in hierarchical ordering where we can move from general to specific. In super class a method can have a general implementation where as the sub classes provide the more specific implementation by overriding the parent class method.
- If a method is declared as [final](https://www.netjstech.com/2015/04/final-in-java.html) in parent class then it can not be overridden by sub classes.
- If a method is declared as [abstract](https://www.netjstech.com/2015/04/abstract-class-in-java.html) in the parent class then the sub class has to provide implementation for those inherited abstract methods

That's all for this topic **Method Overriding in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!