### Class in Java

In object-oriented terms class provides a blue print for the individual objects created from that class. Once a class is defined it can be used to create objects of that type, thus it can be said that class defines new data type.

In this tutorial we'll see how to create a class in Java, how to add a constructor and methods to a class and what all access modifiers can be used with a Java class.

### Form of a class in Java

A class has certain properties like data (variables) and the code that uses that data in the code logic. Since class provides a template for an [object](https://www.netjstech.com/2015/04/object-in-java.html), thus many individual objects of this class type can be created which will have the basic class properties.

### Java class structure

```
class classname {
  type instance_variable1;
  type instance_variable2;
  …..
  ......

  type methodname1(arguments){
    method code here.....
  }

  type methodname2(arguments){
    method code here.....
  }
}
```

The variables defined with in a class are called [instance variables](https://www.netjstech.com/2017/03/kinds-of-variables-in-java.html). The code that operates on these variables is defined with in methods. Together these variables and methods provide the structure of the class.

### Java class example

Here is a simple example of defining a class in Java. This step by step example shows how to write a method in a class, how to define a constructor in the class and how to use it to initialize fields, how to create an object of the class.

```
class Rectangle{
  double length;
  double width;
}

class RectangleDemo {
  public static void main(String[] args) {
    Rectangle rectangle = new Rectangle();
    rectangle.length = 10;
    rectangle.width = 5;
    double area = rectangle.length * rectangle.width;
    System.out.println("Area of the rectangle is " + area);    
  }
}
```

In this example I have created a class Rectangle which has two instance variables length and width. Then using this template Rectangle (class) [new object has been created](https://www.netjstech.com/2017/02/object-creation-using-new-operator-java.html).

```
Rectangle rectangle = new Rectangle();
```

Then values are assigned to the object's instance variables and the area is computed.

We can provide a method in the Rectangle class itself to compute the area, and make the instance variables private to have better [encapsulation](https://www.netjstech.com/2015/04/encapsulation-in-java.html), **the new class will look like this**-

```
class Rectangle{
  private double length;
  private double width;
  // area method to compute area 
  public void area(double ln, double wd){
    length = ln;
    width = wd;
    double area = length * width;
    System.out.println("Area of the rectangle is " + area);
  }
}

class RectangleDemo {
  public static void main(String[] args) {
    Rectangle rectangle = new Rectangle();
    //rectangle.length = 10;
    //rectangle.width = 5;
    //double area = rectangle.length * rectangle.width;
    //System.out.println("Area of the rectangle is " + area); 
    rectangle.area(5, 10);
  }
}
```

But you must be wondering why this double work of passing the variables and then in the method assigning the values to the instance variables. Yes you can use [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) to do this.

So, we can make the Java class even better by introducing constructor for the class, the initialization of the instance variables will be done through the constructor in that case. With these changes **the class will look like this** -

```
class Rectangle{
 double length;
 double width;
 //Constructor
 public Rectangle(double length, double width) {
  this.length = length;
  this.width = width;
 }
 // Method to compute area
 public void area(){
  double area = length * width;
  System.out.println("Area of the rectangle is " + area);
 }
}

 class RectangleDemo { 
 public static void main(String[] args) {
  Rectangle rectangle = new Rectangle(10, 5);  
  rectangle.area();
 }
}
```

### Modifiers for a class in Java

Modifiers for a class may be classified into two categories-

- Access modifiers
- Non-Access modifiers

### Access modifiers for a Java class

Classes in Java can only have the **default (package)** and **public** [access modifiers](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) assigned to them.

- **Public**- class may be declared with the modifier public, in which case that class is visible to all classes everywhere.
- **Default**- If a class in Java has no modifier (the default, also known as package-private), it is visible only within its own package.

**Note** that an [inner class](https://www.netjstech.com/2017/05/nested-class-inner-class-java.html) can be **declared as private** but top level class can take only the above mentioned 2 modifiers.

### Non Access modifiers for a Java class

Non access modifiers for the class in Java are-

- [final](https://www.netjstech.com/2015/04/final-in-java.html)
- [abstract](https://www.netjstech.com/2015/04/abstract-class-in-java.html)
- [strictfp](https://www.netjstech.com/2015/05/strictfp-in-java.html)

**Points to note-**

- class provides a blue print for the individual objects created from that class.
- The variables defined with in a class in Java are called instance variables.
- The code that operates on these variables is enclosed with in methods.
- Together these variables and methods provide the structure of the class.
- Access modifiers used with the class are default (package)and public
- An inner class can be declared as private.
- Non access modifiers for the class are [final](https://www.netjstech.com/2015/04/final-in-java.html), [abstract](https://www.netjstech.com/2015/04/abstract-class-in-java.html) and [strictfp](https://www.netjstech.com/2015/05/strictfp-in-java.html)

That's all for this topic **Class in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!