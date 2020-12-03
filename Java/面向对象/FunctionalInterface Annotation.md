### FunctionalInterface Annotation in Java

In the post [Functional Interfaces in Java](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html) we have already seen that functional interfaces are those interfaces that have only one abstract method. Java 8 also introduced an annotation **@FunctionalInterface** to be used with functional interfaces. Annotating an interface with @FunctionalInterface in Java indicates that an interface type declaration is intended to be a functional interface.

It is **not mandatory to mark functional interface** with @FunctionalInterface annotation, it is more of a best practice to do that and also gives a surety that no other abstract method will be added **accidentally to the functional interface**. Because it will result in compiler error if any other abstract method is added to a functional interface which is annotated with @FunctionalInterface annotation.

Let's see it with some examples what is permitted and what is not with @FunctionalInterface annotation in Java.
First there is an example of a valid functional interface that is annotated with @FunctionalInterface-

```
@FunctionalInterface
public interface IMyFuncInterface {
  public void getValue();
}
```

When you annotate an interface with @FunctionalInterface, if more than one abstract method is defined it will result in compile time error.

```
@FunctionalInterface
public interface IMyFuncInterface {
  public void getValue();
  // Second abstract method so compiler error
  public void setValue();
}
```

Note that in Java 8 [default methods](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) and [static methods](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) are also added in interface which means interface can have a method with default implementation and static methods in Java 8. In a functional interface there may be one or more default methods/static methods but there should be only one abstract method. It is ok to have a functional interface like following.

```
@FunctionalInterface
public interface IMyFuncInterface {
  int func(int num1, int num2);
  // default method
  default int getValue(){
    return 0;
  }    
}
```

A functional interface can specify **Object class public methods** too in addition to the abstract method. That interface will still be a **valid functional interface**. The public Object methods are considered implicit members of a functional interface as they are automatically implemented by an instance of functional interface.

**As example**- This is a valid functional interface

```
@FunctionalInterface
interface IFuncInt {
  int func(int num1, int num2);
  // default method
  default int getValue(){
    return 0;
  }
  public String toString();
  public boolean equals(Object o);
}
```

That's all for this topic **@FunctionalInterface Annotation in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!