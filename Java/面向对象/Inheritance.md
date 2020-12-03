### Inheritance in Java

Inheritance is one of the four fundamental OOP concepts. The other three being-

- [Encapsulation](https://www.netjstech.com/2015/04/encapsulation-in-java.html)
- [Polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html)
- [Abstraction](https://www.netjstech.com/2015/04/abstraction-in-java.html)

**Table of contents**

1. [Inheritance Concept](https://www.netjstech.com/2015/04/inheritance-in-java.html#Inheritance)
2. [Inheritance in Java](https://www.netjstech.com/2015/04/inheritance-in-java.html#Inheritancejava)
3. [Inheritance in Java - What is inherited](https://www.netjstech.com/2015/04/inheritance-in-java.html#Whatisinherited)
4. [Inheritance in Java - What is not inherited](https://www.netjstech.com/2015/04/inheritance-in-java.html#Whatisnotinherited)
5. [Java example code showing inheritance using extends](https://www.netjstech.com/2015/04/inheritance-in-java.html#inheritanceexp)
6. [Types of Inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html#inheritancetypes)
7. [Multiple Inheritance in Java](https://www.netjstech.com/2015/04/inheritance-in-java.html#Multiinheritance)



### Inheritance Concept

Inheritance is a mechanism, by which one class acquires, all the properties and behaviors of another class. The class whose members are inherited is called the **Super class** (or base class), and the class that inherits those members is called the **Sub class** (or derived class).

The relationship between the Super and inherited subclasses is known as **IS-A** relationship.

**As Example-** If Shape is a super class and there are subclasses Circle and Rectangle derived from the Shape super class then Circle IS-A Shape and Rectangle IS-A shape.

### Inheritance in Java

Inheritance in Java is done using extends keyword.

- extends

  – A

   

  class in Java

   

  inherits from another class using extends keyword. In Java, you extend an already existing class or

   

  abstract class

  . Syntax of inheritance in Java using extends keyword is as follows-

  ```
  class Sub_class extends Super_class{
      ...
      ...
  }
   
  ```

### Inheritance in Java - What is inherited

In Java when a class is extended, sub-class inherits all the **public**, **protected** and **default (Only if the sub-class is located in the same package as the super class)** methods and fields of the super class.

### Inheritance in Java - What is not inherited

- **Private** fields and methods of the [super](https://www.netjstech.com/2015/04/super-in-java.html) class are not inherited by the sub-class and can’t be accessed directly by the subclass.
- [Constructors](https://www.netjstech.com/2015/04/constructor-in-java.html) of the super-class are not inherited. There is a concept of [constructor chaining in Java](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html) which determines in what order constructors are called in case of inheritance.

### Java example code showing inheritance using extends

```
//Super class
class A{
 public int a = 10;
 private int b = 15;
 public void displayFromA(){
  System.out.println("Value of a " + a);
 }
}

// sub class 
class B extends A{
 int c = 20;
 public void displayFromB(){
  System.out.println("Value of field a " + a);
  // This line will give compiler error as b 
  // is private and not visible
  System.out.println("Value of field b " + b);
  // Calling inherited method directly as if 
  // it is a method of this class, because it is inherited 
  // from super class
  displayFromA();
  // ok, field of this class
  System.out.println("Value of field c " + c);
 }
}

public class InheritanceDemo {
 public static void main(String[] args) {
  B b = new B();
  // calling the method of class B using class B object
  b.displayFromB();
  // calling the method of class A using class B object
  b.displayFromA();

 }
}
```

**Output**

```
Value of field a 10
Value of a 10
Value of field c 20
Value of a 10
```

It can be seen how methods and fields of super class A are visible in class B, private field b is not visible.

### Types of Inheritance

Going by OOPS concepts there are 5 types of Inheritance-

**Single Inheritance**

In this type of inheritance a sub class is derived from a single Super class.

[![Single Inheritance in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:20:25-Single%252BInheritance.jpg)](https://2.bp.blogspot.com/-DMJgeo2_Z1I/VS0ztW54epI/AAAAAAAAAC8/Ombi1wB--zE/s1600/Single%2BInheritance.jpg)

**Multi-level inheritance**

In this type of inheritance, a subclass is created from another sub class thus having levels of hierarchy.

[![Multi-level inheritance in java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:20:25-multi-level%252Binheritance.jpg)](https://1.bp.blogspot.com/-xrCDz6d_Zl0/VS00QP8NaGI/AAAAAAAAADE/ZtDm8-k3FgY/s1600/multi-level%2Binheritance.jpg)

**Multiple Inheritance**

In this type of inheritance, a sub class is created using more than one super class. **Note** – [Java does not support multiple inheritance](https://www.netjstech.com/2017/06/why-no-multiple-inheritance-in-java-diamond-problem.html).

[![Multiple Inheritance](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:20:25-multipleinheritance.jpg)](https://3.bp.blogspot.com/--5I792btfMA/VS02FTKin0I/AAAAAAAAADg/6xFKG8cGFZs/s1600/multipleinheritance.jpg)

**Hierarchical Inheritance**

In this type of inheritance more than one sub classes are created from the same super class.

[![Hierarchical Inheritance](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:20:26-hierarchical-inheritance.jpg)](https://4.bp.blogspot.com/-h8VfBVAC1ws/VS03DUNdtLI/AAAAAAAAADs/xvbJAFOjF7k/s1600/hierarchical-inheritance.jpg)

**Hybrid inheritance**

This type of inheritance is the combination of more than one inheritance type as described above. Hence, it may be a combination of Multilevel and Multiple inheritance or Hierarchical and Multilevel inheritance or Hierarchical and Multiple inheritance.

[![Hybrid inheritance](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:20:26-hybrid-inheritance.jpg)](https://3.bp.blogspot.com/-gT-gvHO-ZkA/VS037CY7RmI/AAAAAAAAAD0/FcloyZ5VvA0/s1600/hybrid-inheritance.jpg)

### Multiple Inheritance in Java

Java doesn’t support multiple inheritance which means a class can extend at most one super class only. Though, a class can implement more than one interface.

- Refer [Why no multiple inheritance in Java](https://www.netjstech.com/2017/06/why-no-multiple-inheritance-in-java-diamond-problem.html) to see why multiple inheritance is not supported in Java.

**Points to note-**

- In Java inheritance is done using extends keyword.
- Constructors of the super class are not inherited by the sub class.
- Private fields and methods of the superclass are not inherited by the subclass and can’t be directly accessed.
- Members with default access in the super class are inherited by the subclass only if they both reside in the same package.
- Using [final](https://www.netjstech.com/2015/04/final-in-java.html) keyword in Java, a class can restrict that no class can extend it.
- If both super class and sub class have a method with same signature then subclass method is said to be overriding the super class method.
- Multiple inheritance is not allowed in Java, a class can extend at most one super class. Though a class can implement more than one interface and that's one way to get multiple inheritance in Java.

That's all for this topic **Inheritance in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!