### Abstraction in Java

Abstraction is one of the four fundamental OOP concepts. The other three being-

- [Inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html)
- [Polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html)
- [Encapsulation](https://www.netjstech.com/2015/04/encapsulation-in-java.html)

### Abstraction Concept

Abstraction means hiding the complexity and only showing the essential features of the [object](https://www.netjstech.com/2015/04/object-in-java.html). So in a way, abstraction means abstracting/hiding the real working (implementation details) and we, as a user, knowing only how to use it.

Real world example of abstraction would be-

- A vehicle which we drive with out caring or knowing what all is going underneath.
- A TV set where we enjoy programs with out knowing the inner details of how TV works.





**An example of abstraction in Java would be**- [Java Database Connectivity (JDBC) API](https://www.netjstech.com/2017/12/java-jdbc-overview-jdbc-tutorial.html) which provides universal data access from the Java programming language. Using the JDBC API, we can access virtually any data source without knowing how the driver for that particular data source is implemented. All we have is an API with a given set of methods.



### Abstraction in Java

Abstraction in Java is achieved through-

- [Interface](https://www.netjstech.com/2015/05/interface-in-java.html)- Which defines an expected behavior, with out providing any details about the behavior.
- [Abstract Class](https://www.netjstech.com/2015/04/abstract-class-in-java.html)- Which provides incomplete functionality and leave it on the extending class to fill the gaps.

Following image shows the concept of abstraction. User interacts to the real implementation through API methods. How is API actually implemented is abstracted from the user.

[![abstraction in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:22:29-Abstraction.png)](https://4.bp.blogspot.com/-_4lL8SmtMDA/XCHIKFjKkGI/AAAAAAAABFY/ezIQ7D8rmQsMjx0SVvt4P1Nxqe7OrlHCQCPcBGAYYCw/s1600/Abstraction.png)

### Abstraction in Java through Interface example

```
public interface MyInterface {
 void display(String msg);
 String getValue(String str);
}
```

As it can be seen there are 2 methods in the interface MyInterface; **display** and **getValue**. Here **display** method has one parameter of type [String](https://www.netjstech.com/2016/07/string-in-java.html) and no value is returned. Whereas, **getValue()** method has one String parameter and returns a String too.

Let’s say we have two classes **MyClassImpl** and **MyClassImpl1** implementing the interface MyInterface.

**MyClassImpl class**

```
public class MyClassImpl implements MyInterface {

 @Override
 public void display(String msg) {
  System.out.println("Message is " + msg);
 }

 @Override
 public String getValue(String str) {
  // TODO Auto-generated method stub
  return "Hello " + str;
 }
}
```

Here the **display method** is implemented to display the passed parameter. Method **getValue** prefixes Hello to the passed parameter.

**MyClassImpl1 class**

```
public class MyClassImpl1 implements MyInterface {

 @Override
 public void display(String msg) {
  System.out.println("Message in Uppercase " + msg.toUpperCase());
 }

 @Override
 public String getValue(String str) {
  return str.toUpperCase();
 }
}
```

Here the **display method** is implemented to display the passed parameter in upper case. Method **getValue** returns the passed parameter after converting it to upper case.

**TestInt Class**

```
public class TestInt {

 public static void main(String[] args) {
  MyInterface obj;
  // Holding reference of MyClssImpl
  obj = new MyClassImpl();
  callClass(obj);
  // Holding reference of MyClssImpl1
  obj = new MyClassImpl1();
  callClass(obj);

 }
 
 private static void callClass(MyInterface obj){
  obj.display("Calling class");
  String str = obj.getValue("abstraction test");
  System.out.println("Value - " + str);
 }
}
```

**Output**

```
Message is Calling class
Value - Hello abstraction test
Message in Uppercase CALLING CLASS
Value - ABSTRACTION TEST
```

In **TestInt** class there is an object of type **MyInterface** when it has reference of type **MyClassImpl** it is calling methods of that class, when it has reference of type **MyClassImpl1** it is calling methods of that class. But as a user you are abstracted from the different implementations.

**Points to note**

- Abstraction is one of the four fundamental OOPS concept. Other being, inheritance, polymorphism and encapsulation.
- Abstraction is a way to hide the real implementation and user just knows how to use it.
- Abstraction in Java is achieved through [interfaces](https://www.netjstech.com/2015/05/interface-in-java.html) and [abstract classes](https://www.netjstech.com/2015/04/abstract-class-in-java.html).

That's all for this topic **Abstraction in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!