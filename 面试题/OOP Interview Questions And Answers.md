In this post some of the Java OOP interview questions and answers are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is encapsulation?**

   Encapsulation means keeping together the implementation (code) and the data it manipulates (variables). In Java a class contains the member variables (instance variables) and the code in methods. In a properly encapsulated Java class method defines how member variables can be used.
   Read more about encapsulation [here](https://www.netjstech.com/2015/04/encapsulation-in-java.html).

2. ------

3. **What is abstraction?**

   Abstraction means hiding the complexity and only showing the essential features of the object. So in a way, Abstraction means abstracting/hiding the real working and we, as a user, knowing only how to use it.

   Real world example would be a vehicle which we drive with out caring or knowing what all is going underneath.

   Read more about abstraction

    

   here

   

4. ------

5. **Difference between abstraction and encapsulation?**

   Abstraction is more about hiding the implementation details. In Java abstraction is achieved through abstract classes and interfaces.

   

   Encapsulation is about wrapping the implementation (code) and the data it manipulates (variables) with in a same class. A Java class, where all instance variables are private and only the methods with in the class can manipulate those variables, is an example of encapsulated class.
   Read more about difference between Encapsulation and Abstraction in Java [here.](https://www.netjstech.com/2016/08/difference-between-encapsulation-and-abstraction-java.html)

6. ------

7. **What is Polymorphism?**

   Polymorphism, a Greek word, where poly means many and morph means change, thus it refers to the ability of an object taking many forms. The concept of polymorphism is often expressed as "One interface, multiple methods".
   Where one general class may have the generic method and the derived classes (classes which extend the general class) may add their own specific method with implementation. At the time of execution "One interface" i.e. the general class will take "many forms" i.e. references of the derived classes.
   Read more about polymorphism [here](https://www.netjstech.com/2015/04/polymorphism-in-java.html).

8. ------

9. **What is inheritance?**

   Inheritance is a mechanism, by which one class acquires, all the properties and behaviors of another class. The class whose members are inherited is called the Super class (or base class), and the class that inherits those members is called the Sub class (or derived class).
   The relationship between the Super and inherited subclasses is known as IS-A relationship.
   Read more about inheritance [here](https://www.netjstech.com/2015/04/inheritance-in-java.html).

10. ------

11. **What Is a Class?**

    In object-oriented terms class provides a blue print for the individual objects created from that class. Once a class is defined it can be used to create objects of that type.
    Read more about class [here](https://www.netjstech.com/2015/04/class-in-java.html).

12. ------

13. **What Is an Object?**

    In object-oriented terms object is an instance of the class, which gets its state and related behavior from the class. When a new class is created essentially a new data type is created. This type can be used to declare objects of that type.
    An object stores its state in fields and exposes its behavior through methods.
    Read more about object [here](https://www.netjstech.com/2015/04/object-in-java.html).

14. ------

15. **What is method overloading?**

    When two or more methods with in the same class or with in the parent-child relationship classes have the same name, but the parameters are different in types or number the methods are said to be overloaded. This process of overloaded methods is known as method overloading.
    Read more about method overloading [here](https://www.netjstech.com/2015/04/method-overloading-in-java.html).

16. ------

17. **What is method overriding?**

    In a parent-child relation between classes, if a method in subclass has the same signature as the parent class method then the method is said to be overridden by the subclass and this process is called method overriding.
    Read more about method overriding [here](https://www.netjstech.com/2015/04/method-overriding-in-java.html).

18. ------

19. **What is interface in Java?**

    Interfaces help in achieving full abstraction in Java, as using interface, you can specify what a class should do, but how class does it is not specified.
    Interfaces look syntactically similar to classes, but they differ in many ways -

    - Interfaces don't have instance variables.
    - In interfaces methods are declared with out any body. They end with a semicolon.
    - Interface can't be instantiated.
    - Interfaces don't have constructors.
    - An interface is implemented by a class not extended.
    - An interface can extend multiple interfaces.

    

    Read more about interfaces

     

    here

    .

20. ------

21. **Can an Interface be final?**

    No, interface can't be final. The whole idea of having an interface is to inherit it and implement it. Having it as final means it can't be subclassed.

    Read more about interfaces

     

    here

    .

22. ------

23. **What is an abstract class?**

    An abstract class is a class that is declared using the abstract keyword. An abstract class may contain methods without any implementation, called abstract methods.

    Read more about abstract class

     

    here

    .

24. ------

25. **What are the differences between abstract class and interface?**

    

    |                      | Abstract Class                                               | Interface                                                    |
    | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | Methods              | Abstract class can have both abstract and non-abstract methods. | Interface can have abstract methods only. Note: From Java 8 interfaces can have [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html). |
    | Access Modifiers     | Abstract class methods can have public, protected, private and default modifier apart from abstarct methods. | In interface methods are by default public abstract only.    |
    | Variables            | Abstract class fields can be non-static or non-final.        | In interface all the fields are by default public, static, final. |
    | Implementation       | Abstract class may have some methods with implementation and some methods as abstract. | In interface all the methods are by default abstract, where only method signature is provided. Note: From Java 8 interfaces can have [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html). |
    | Constructor          | Abstract class have a constructor, it may be user supplied or default in case no constructor is written by a user. | Interface can't have a constructor.                          |
    | Multiple Inheritance | Abstract class can extend at most one class and implement one or more interfaces. | Interface can only extend one or more interfaces.            |
    | Abstraction          | Abstract class can provide both partial or full abstraction. | Interface provides full abstraction as no implementation is provided for any of the method. |
    | Extends/Implements   | Abstract class are extended by the sub-classes. Sub-classes need to provide implementation for all the abstract methods of the extended abstract class or be declared as abstract itself. | Interface is implemented by a class and the implementing class needs to provide implementation for all the methods declared in an interface. If a class does not implement all the methods of interface then that class must be declared as abstract. |
    | Easy to evolve       | Abstract class was considered easy to evolve as abstract classes could add new methods and provide default implementation to those methods. | Interface was not considered easy to evolve as, in the case of adding new method to an interface, all the implementing classes had to be changed to provide implementation for the new method. With Java 8 even interfaces can have default methods so that issue has been addresses. |

    

26. ------

27. **What is an Assocation?**

    An association is a structural relationship, in object oriented modelling, that specifies how objects are related to one another. This structural relationship can be shown in-

    - Class diagram associations
    - Use case diagram associations

    An association may exist between objects of different types or between the objects of the same class. An association which connects two classes is called a

     

    binary association

    . If it connects more than two classes (not a very common scenario) it is called

     

    n-ary association

    . There are two types of associations

    - Aggregation
    - Composition

    Read more about Association

     

    here

    .

    

28. ------

29. **What is an Aggregation?**

    Sometimes you do need to define a “whole/part” relationship between classes where one class (whole) consists of smaller classes (part). This kind of relationship is called aggregation. It is also known as a “has-a” relationship as object of the whole has objects of the part. For example Company has a Person.

    Aggregation doesn't represent a strong "whole/part" relationship and both the "whole" and "part" entities can survive without each other too.

    Read more about Aggregation

     

    here

    .

    

30. ------

31. **What is a Composition?**

    There is a variation of aggregation called composition which has strong ownership, thus the scope of the whole and part are related. In composition if there are two classes, class A (whole) and class B (part) then destruction of class A object will mean class B object will also cease to exist. Which also means class B object may be a part of class A object only.
    Read more about Composition [here](https://www.netjstech.com/2016/08/association-aggregation-and-composition-differences-in-java.html).