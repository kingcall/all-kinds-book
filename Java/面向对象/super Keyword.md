### super Keyword in Java With Examples

The **super** keyword in java is essentially a reference variable that can be used to refer to class' immediate parent class.

### Usage of super in Java

super keyword in Java can be used for the following-

- Invoke the constructor of the super class. See [example](https://www.netjstech.com/2015/04/super-in-java.html#superinvokeconstructor).
- Accessing the variables and methods of parent class. See [example](https://www.netjstech.com/2015/04/super-in-java.html#supermemberaccess).

First let's see how code will look like if super() is not used.

Let's say there is a super class Shape with instance variables length and breadth. Another class Cuboids extends it and add another variable height to it. With these 2 classes and not using **super()**, constructors of the class will look like-

```
public class Shape {
 int length;
 int breadth;
 Shape(){
  
 }
 // Constructor
 Shape(int length, int breadth){
  this.length = length;
  this.breadth = breadth;
 }
}

class Cuboids extends Shape{
 int height;
 //Constructor
 Cuboids(int length, int breadth, int height){
  this.length = length;
  this.breadth = breadth;
  this.height = height;
 }
}
```

It can be noticed there are 2 problems with this approach -

- Duplication of code as the same initialization code in the constructor is used twice. Once in Cuboids class and once in Shape class.
- Second and most important problem is super class instance variables can not be marked as private because they have to be accessed in child class, thus violating the OOP principle of [Encapsulation](https://www.netjstech.com/2015/04/encapsulation-in-java.html).

So **super()** comes to the rescue and it can be used by a child class to refer to its immediate super class. Let's see how super keyword can be used in Java.

### Using super to invoke the constructor of the super class

If you want to initialize the variables that are residing in the immediate parent class then you can call the constructor of the parent class from the [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) of the subclass using **super() in Java**.

**Note that** in that case **super()** should be the **first statement** inside the subclass' constructor. This will ensure that if you call any methods on the parent class in your constructor, the parent class has already been set up correctly.

**Java example code using super**

```
public class Shape {
 private int length;
 private int breadth;
 Shape(){
  
 }
 Shape(int length, int breadth){
  this.length = length;
  this.breadth = breadth;
 }
}

class Cuboids extends Shape{
 private int height;
 Cuboids(int length, int breadth, int height){
  // Calling super class constructor
  super(length, breadth);
  this.height = height;
 }
}
```

Here it can be noticed that the instance variables are **private now** and super() is used to initialize the variables residing in the super class thus avoiding duplication of code and ensuring proper encapsulation.

**Note:** If a constructor does not explicitly invoke a superclass constructor, the Java compiler automatically inserts a call to the no-argument constructor of the superclass. If the super class does not have a no-argument constructor, you will get a compile-time error.

That will happen because; If a constructor is explicitly defined for a class, then the Java compiler will not insert the default no-argument constructor into the class. You can see it by making a slight change in the above example.

In the above example, if I comment the default constructor and also comment the super statement then the code will look like -

```
public class Shape {
  private int length;
  private int breadth;
  /*Shape(){
   
  }*/
  Shape(int length, int breadth){
   this.length = length;
   this.breadth = breadth;
  }
}

class Cuboids extends Shape{
  private int height;
  Cuboids(int length, int breadth, int height){
   // Calling super class constructor
   /*super(length, breadth);*/
   this.height = height;
  }
}
```

This code will give compile-time error. "*Implicit super constructor Shape() is undefined*".

### Using super to access Super class Members

If method in a child class overrides one of its superclass' methods, method of the super class can be invoked through the use of the **keyword super**. super can also be used to refer to a hidden field, that is if there is a variable of the same name in the super class and the child class then super can be used to refer to the super class variable.

**Java example showing use of super to access field**

```
public class Car {
 int speed = 100;
 
}

class FastCar extends Car{
 int speed = 200;
 FastCar(int a , int b){
  super.speed = 100;
  speed = b;
 }
}
```

Here, in the constructor of the FastCar class **super.speed** is used to access the instance variable of the same name in the super class.

**Example showing use of super to access parent class method**

```
public class Car {
 void displayMsg(){
  System.out.println("In Parent class Car");
 }
}

class FastCar extends Car{
 void displayMsg(){
  System.out.println("In child class FastCar");
  // calling super class method
  super.displayMsg();
 }
 public static void main(String[] args){
  FastCar fc = new FastCar();
  fc.displayMsg();
 }
}
public class Test {

 public static void main(String[] args){
  FastCar fc = new FastCar();
  fc.displayMsg();
 }
}
```

**Output**

```
In child class FastCar
In Parent class Car
```

**Points to note-**

- **super** keyword in java is a reference variable to refer to class' immediate parent class.
- super can be used to invoke the constructor of the immediate parent class, that's help in avoiding duplication of code, also helps in preserving the encapsulation.
- If a constructor does not explicitly invoke a superclass constructor, the Java compiler automatically inserts a call to the no-argument constructor of the superclass.
- super can also be used to access super class members.
- If a variable in child class is shadowing a super class variable, **super** can be used to access super class variable. Same way if a parent class method is overridden by the child class method then the parent class method can be called using super.

That's all for this topic **super Keyword in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!

[>>>Return to Java Basics Tutorial Page](https://www.netjstech.com/2015/04/java-basics.html)