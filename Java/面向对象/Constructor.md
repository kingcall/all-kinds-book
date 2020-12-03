### Constructor in Java

In Java there is a special method provided to **initialize objects** when they are created. This special method which helps in automatic initialization is called **Constructor** in Java.

### Java Constructor

Constructor in Java has the same name as the class in which it is created and defined just like a method, that is, constructor's syntax is similar to a method.

- Refer [Initializer block in Java](https://www.netjstech.com/2016/02/initializer-block-in-java.html) to see another way to initialize instance variables.

Constructor in Java is called automatically when the object is created using the new operator, before the [new operator](https://www.netjstech.com/2017/02/object-creation-using-new-operator-java.html) completes. Constructors don't have a return type, *even void is not allowed*. Constructors have an implicit return type which is the class type (current instance) itself.

**Table of contents**

1. [Java Constructor Example](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaConstructorExp)
2. [Types of Constructors in Java](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaConstructorTypes)
3. [Default Constructor in Java](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaDefaultConstructor)
4. [No-arg Constructor in Java](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaNoargConstructor)
5. [Parameterized Constructor in Java](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaParameterizedConstructor)
6. [Constructor Chaining in Java](https://www.netjstech.com/2015/04/constructor-in-java.html#JavaConstructorChaining)



### Java Constructor Example

```
public class ConstrExample {
 int i;
 String name;
 // Constructor
 public ConstrExample() {
  System.out.println("Creating an object");
  System.out.println("i - " + i + " name - " + name);
 }

 public static void main(String[] args) {
  ConstrExample constrExample = new ConstrExample();
 }
}
```

Here Constructor is this part

```
public ConstrExample() {
 System.out.println("Creating an object");
 System.out.println("i - " + i + " name - " + name);
}
```

And executing this program will give the output as-

```
Creating an object 
i - 0 name - null
```

**Note here that**-

- Constructor name is same as the class name.
- In the above code constructor's [access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) is public, but constructor can have private, protected or default access modifier.
- As already stated constructors in Java don't have a return type. If you give even void as a return type then compiler would not treat it as a constructor but as a regular method. In that case Java compiler will add a default constructor.
- Constructor in Java can't be [final](https://www.netjstech.com/2015/04/final-in-java.html).

### Types of Constructors in Java

Java constructors can be classified into three types-

1. Default constructor
2. No-arg constructor
3. Parameterized constructor

### Default Constructor in Java

When a constructor is not explicitly defined in a class, then Java inserts a default no-arg constructor for a class. If a constructor is explicitly defined for a class, then the Java compiler will not insert the default no-argument constructor into the class.

**Java Default Constructor example**

In the constructor example shown above we used a no-arg constructor, if we don't write that no-arg constructor explicitly even then Java will insert a default no-arg constructor.

```
public class ConstrExample {
 int i;
 String name;
  
 public static void main(String[] args) {
  ConstrExample constrExample = new ConstrExample();  
 }
}
```

As you can see in the program there is no constructor. If you see the structure of the generated .class file using [javap command](https://www.netjstech.com/2016/10/how-to-run-javap-programmatically-from-java.html) then you can also see an entry for default constructor added by Java.

```
javap F:\Anshu\NetJs\NetJSExp\bin\org\netjs\examples\ConstrExample.class

Compiled from "ConstrExample.java"

public class org.netjs.examples.ConstrExample {
  int i;
  java.lang.String name;
  // Constructor
  public org.netjs.examples.ConstrExample();
  public static void main(java.lang.String[]);
}
```

### No-arg Constructor in Java

Constructor with out any parameters is known as no-arg constructor. While default constructor is implicitly inserted and has no body, no-arg constructor is explicitly written in the class and it can have code too.

In your class if you do have any constructor then default constructor is not inserted automatically. In that case if you want to have an option to create object without passing any argument then you have to have a no-arg constructor in your class. See example for this scenario in Parameterized constructor section.

**Java No-arg Constructor example**

```
public class ConstrExample {
 int i;
 String name;
 public ConstrExample() {
  System.out.println("Creating an object");
  this.name = "Hello";
  this.i = 10;  
 }
  
 public static void main(String[] args) {
  ConstrExample constrExample = new ConstrExample();
  System.out.println("i = " + constrExample.i);
  System.out.println("name = " + constrExample.name);
 }
}
```

**Output**

```
Creating an object
i = 10
name = Hello
```

### Parameterized Constructor in Java

If we want our object's fields to be initialized with specific values, we can do it by adding parameters to the constructor.
A class can have multiple constructors too as long as constructor signatures are different.

**Java Parameterized Constructor Example**

```
public class ConstrExample {
 int i;
 String year;
 // Parameterized Constructor
 ConstrExample(int i, String year) {
  System.out.println("Creating a parameterized object");
  this.i = i;
  this.year = year;
  System.out.println("i - " + i + " year - " + year);
 }
 //no-arg constructor
 ConstrExample() {
  System.out.println("Creating a object");  
  System.out.println("i - " + i + " year - " + year);
 }
  
 public static void main(String[] args) {
  ConstrExample constrExample1 = new ConstrExample(10, "2015");
  ConstrExample constrExample2 = new ConstrExample(); 
 }
}
```

And executing this program will give the output as-

```
Creating a parameterized object
i - 10 year - 2015
Creating a object
i - 0 year - null
```

**Note here that**-

- There are 2 constructors here one with parameters another no-arg constructor, this is known as [constructor overloading](https://www.netjstech.com/2015/04/constructor-overloading-in-java.html).
- 2 Objects are created one with parameters which will invoke the parameterized constructor and another without any parameters which will invoke the no-arg contructor.

### Constructor Chaining in Java

When you have a hierarchy of classes it is important to know what is the order in which the constructors for the classes are executed. That order is known as [constructor chaining in Java](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html).

**As example-** If class A is superclass and there is Class B which is subclass of A. What is the order in which constructors of Class A and Class B are executed?
Answer is, the order followed is from **superclass to subclass**.

Subclass can call a constructor in the superclass inside one of the subclass constructors using **super()**. In that case super() must be the first statement in a subclass constructor. If super() is not used then the default no-arg constructor of each superclass will be executed. **Note that** the order remains same (from superclass to subclass ) whether or not super() is used.

**Java Constructor Chaining example**

```
class A{
 A(){
  System.out.println("In class A's constructor");
 }
}

class B extends A{
 B(){
  System.out.println("In class B's constructor");
 }
}

class C extends B{
 C(){
  System.out.println("In class C's constructor");
 }
}

public class ConstrChaining {

public static void main(String[] args) {
  C c = new C();

 }
}
```

And executing this program will give the output as-

```
In class A's constructor
In class B's constructor
In class C's constructor
```

**Points to note-**

- Constructor in Java has the same name as the class in which it is created.
- Constructor is called automatically when the object is created, in order to initialize an object.
- Constructors don't have a return type, even void is not allowed.
- Constructor can have any access modifier public, private, protected or default.
- When a constructor is not explicitly defined for a class, then Java creates a default no-arg constructor for a class, that is called default constructor in Java.
- Within the same class there may be multiple constructors where the constructors differ in number and/or types of parameters, that process is known as [Constructor overloading](https://www.netjstech.com/2015/04/constructor-overloading-in-java.html).
- In case of inheritance, constructors are called in order of super class to sub class, that process is known as Constructor chaining.

That's all for this topic **Constructor in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!