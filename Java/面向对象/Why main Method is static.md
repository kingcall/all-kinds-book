### Why main Method is static in Java

When we start learning java and write our first "Hello World" program, there are two things that stand out.

- [File name and class name should be same in Java](https://www.netjstech.com/2015/04/why-file-name-and-class-name-same-in-java.html).
- Main method signature- The main method signature must be public static void main(String[] args)



Here we'll talk about the **main method** and the necessity to mark main method as [static in Java](https://www.netjstech.com/2015/04/static-in-java.html). Coming to main method signature it is easy to see that main method is-

- **Public**- [Access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) is public so that main method is visible to every other class, same package or other. If it is not public JVM classes will not be able to access it.
- **Void**- As it does not return any value.
- **String[] args**- arguments to this method. The main method takes an array of Strings as parameter, that array is called 'args'.



But why main method is static in Java is a question that needs some explanation.

In the post [Why file name and class name should be same in Java](https://www.netjstech.com/2015/04/why-file-name-and-class-name-same-in-java.html) it has already beed discussed that at run time class name should be same, as that's how [JVM](https://www.netjstech.com/2015/05/what-are-jvm-jre-and-jdk-in-java.html) knows which class to load and where is the entry point (main method). When you start execution of a Java program, JVM looks for the main method in the provided Java class as that is the entry point for the execution.

But the question here is, how will JVM access that main method with out creating an instance of the class, **answer is having main method as static**.

Here note that any static method in Java is associated with the class not with any object of the class. Static method can be called without creating any object of the class.

### What if main method is not declared as static

If Java main method is not declared as static then instance of the class has to be created which may cause ambiguity, **consider the example** -

```
public class A {
 private int i;
  A(int i){
  this.i = i;
 }
 public static void main(String args[]){
  A a = new A(5);
 }
}
```

Here in the class there is a [constructor](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html) with one argument i. Assume that the main method is not static, which means JVM has to directly create the instance of class A to execute the main method. But in order to create an [object](https://www.netjstech.com/2015/04/object-in-java.html) of the class constructor has to be invoked which brings up the question what should be passed as i?

JVM wouldn't know with what values your object has to be initialized. So you have a catch22 situation here in order to get an object of the class this line **A a = new A(5);** has to be executed (Which is with in the main method) and to execute this line instance of the class is needed if there is no static method in the class.

To avoid these types of ambiguities it doesn't make sense for the JVM to have to create an object of the class before the entry point (main method) is called. That's why main method is static in Java.

**Points to note-**

- main method in Java must be declared public, static and void if any of these are missing; java program will compile but at run time error will be thrown.
- At run time interpreter is given the class file which has main method, thus main method is entry point for any java program.
- Declaring main method as static in Java ensures that JVM can invoke the entry point (main method) with out creating any instance of the class.
- With [varargs in Java 5](https://www.netjstech.com/2015/05/varargs-in-java.html) onward it is possible to write the main method as **public static void main(String ... args)**.

That's all for this topic **Why main Method is static in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!