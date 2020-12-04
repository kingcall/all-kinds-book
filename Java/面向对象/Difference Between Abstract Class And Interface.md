### Difference Between Abstract Class And Interface in Java

Difference between abstract class and interface in Java is one of the most popular [java interview questions](https://www.netjstech.com/2015/08/core-java-basics-interview-questions.html) and many times it is a kind of "*breaking the ice*" question when the interview just starts. But that way Abstract class Vs Interface becomes a very important question as it is often said "*first impression is the last impression*" so let's try to see the differences between abstract class and interface in Java in this post.

First of all it is very important to know what an abstract class is and what an interface is. So please go through these two posts [abstract class in Java](https://www.netjstech.com/2015/04/abstract-class-in-java.html) and [interface in Java](https://www.netjstech.com/2015/05/interface-in-java.html) to familiarize yourself with abstract class and interface.

There are also **some similarities** between abstract class and interface in Java like both can't be instantiated, both have abstract methods where extending/implementing class has to provide the implementation for such methods.

### Abstract class Vs Interface in Java

|                      | Abstract Class                                               | Interface                                                    |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Methods              | Abstract class can have both abstract methods (method with no body) and non-abstract methods (methods with implementation). | Interface can have abstract methods only. **Note**: From Java 8 interfaces can have [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) and [private methods](https://www.netjstech.com/2018/10/private-methods-in-java-interface.html) Java 9 onward. So on this account both abstract classes and interfaces in Java are becoming quite similar as both can have methods with implementation. |
| Access Modifiers     | Abstract class methods can have public, protected, private and default modifier apart from abstract methods. | In interface methods are by default public abstract only. From Java 9 private methods can also be added to a Java interface. |
| Variables            | Abstract class fields can be non-static or non-final.        | In interface all the fields are by default public, static, final. |
| Implementation       | Abstract class may have some methods with implementation and some methods as abstract. | In interface all the methods are by default abstract, where only method signature is provided. **Note**: From Java 8 interfaces can have [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) where default implementation can be provided with in the interface and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) that can be accessed using the Interface name. Apart from that interfaces can have private methods too Java 9 onward. |
| Constructor          | Abstract classes have a constructor, it may be user supplied or default in case no constructor is written by a user. | Interfaces can't have a constructor.                         |
| Multiple Inheritance | Abstract class can extend at most one class and implement one or more interfaces. | Interface can only extend one or more interfaces.            |
| Extends/Implements   | Abstract class are extended by the sub-classes. Sub-classes need to provide implementation for all the abstract methods of the extended abstract class or be declared as abstract itself. | Interface is implemented by a class and the implementing class needs to provide implementation for all the abstract methods declared in an interface. If a class does not implement all the abstract methods of an interface then that class must be declared as abstract. |
| Easy to evolve       | Abstract class was considered easy to evolve as abstract classes could add new methods and provide default implementation to those methods. | Interface was not considered easy to evolve as, in the case of adding new method to an interface, all the implementing classes had to be changed to provide implementation for the new method. With Java 8 even interfaces can have default methods so that issue has been addressed. |

### Which one should you use, abstract classes or interfaces?

As we know abstract classes have to be extended where as interfaces need to be implemented by a class. That itself suggests when to use what, in case you need to extend some functionality further you need to go with abstract class. When you need to start from generalized structure and move towards more specialized structure by extending the generalized class and providing specialized implementations, abstract class is a better choice.

In case of interfaces it is expected that unrelated classes would implement that interface and each class is free to provide their own implementation. That's how using interfaces, Java fully utilizes "one interface, multiple methods" aspect of [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html).

That's all for this topic **Difference Between Abstract Class And Interface in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!

