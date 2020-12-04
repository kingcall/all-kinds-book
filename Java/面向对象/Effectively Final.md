### Effectively Final in Java 8

In Java 8 with the addition of lambda expressions, a new concept effectively final variable has been added, which closely relates to [lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-and-variable-scope.html).

Prior to Java 8, [inner classes](https://www.netjstech.com/2017/05/nested-class-inner-class-java.html) had an important restriction that *inner classes could only use variable from enclosing scope if it's [final](https://www.netjstech.com/2015/04/final-in-java.html)*.

Let's see an example where we have an interface IFunc, with one method display(). That interface is implemented as an anonymous inner class in Test class. In the implementation of display() method it tries to access the variable i from the enclosing scope which is not declared as [final variable](https://www.netjstech.com/2015/04/final-in-java.html). Note that this code is compiled and executed using **Java 6**.

```
interface IFunc{
  void display();
}

public class Test {

 /**
  * @param args
  */
 public static void main(String[] args) {
  int i =7;
  // Anonymous inner class
  new IFunc() {
   @Override
   public void display() {
    System.out.println("Value is " + i); 
   }
  };
 }
}
```

There will be a compile time error-

*Cannot refer to a non-final variable i inside an inner class defined in a different method*

### Effectively final in Java 8

With Java 8 this restriction of declaring the variable in the enclosing scope as final has been eased a bit. If you are accessing a variable from the enclosing scope in an inner class (or lambda expression for that matter) you are not forced by compiler to declare that variable as final (though declaring the variable as final is still ok). This is known as effectively final in Java.

Note here that you still can't change the value of that variable. Doing so will result in "*local variable defined in an enclosing scope must be final or effectively final*" error. So, effectively final variable in Java is a field that is not final but the value of that field can't be changed after initialization.

Let's write the same code as above using [Lambda expression in Java 8](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html)-

```
@FunctionalInterface
interface  IFunc{
  void display();
}

public class InnerDemo {
  public static void main(String[] args) {
    int i = 7;
    // lambda expression that implements the display method 
    // of the IFunc functional interface 
    IFunc ifunc = ()-> System.out.println("Value of i is " + i);
    // Calling the display method
    ifunc.display();
  }   
}
```

**Output**

```
Value of i is 7
```

It can be seen if *you are not changing* the value of local variable i, it can be used with lambda expression and there is no need to declare i as final.

With inner class also you won't get error now. If you'll compile the same code (inner class implementation) which was not running in Java 6, it will compile with Java 8.

### Effectively final field error

As already stated you still can't change the value of the variable used from the enclosed scope, trying to do that will result in error.

Here it can be seen variable i from the enclosing scope is changed with in the lambda expression by trying to print i++. Thus compile time error is thrown that "*Local variable i defined in an enclosing scope must be final or effectively final*"

[![Effectively final in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:53:18-effectively%252Bfinal-New.png)](https://1.bp.blogspot.com/-tRq40A7Bvbg/XD3cNTMvecI/AAAAAAAABGw/HIDdpIeGSRQVDWU6OAqsaMIB3FhorYkJwCPcBGAYYCw/s1600/effectively%2Bfinal-New.png)

That's all for this topic **Effectively Final in Java 8**. If you have any doubt or any suggestions to make please drop a comment. Thanks!