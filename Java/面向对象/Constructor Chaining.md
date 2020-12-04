### Constructor Chaining in Java

When we have a hierarchy of classes (in case of [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html)) it is important to know the order in which constructors for the classes are executed. That order is known as **constructor chaining** in Java.

There is also concept of [constructor overloading in Java](https://www.netjstech.com/2015/04/constructor-overloading-in-java.html) where, with in the same class, there are multiple constructors with different signatures.

### How does Constructor Chaining work in Java

If class A is superclass and there is a child class Class B. In that case if a new instance of class B is created what is the order in which constructors of Class A and Class B are executed?

**Answer is**; the order followed is from superclass to subclass.

Subclass can call a constructor in the superclass inside one of the subclass constructors explicitly using **super()**. In that case super() must be the first statement in a subclass constructor. If super() is not used then the default no-arg constructor of each superclass will be executed implicitly. Note that the order remains same (from superclass to subclass ) whether or not super() is used.

### Constructor chaining Java example

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
  super();
  System.out.println("In class C's constructor");
 }
}
public class ConstrChaining {

 /**
  * @param args
  */
 public static void main(String[] args) {
  C c = new C();

 }
}
```

It can be noticed here that we have a [multilevel inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html) where super class is A which is extended by B and B is extended by C. When the instance of C is created it will trigger the chain of constructor invocations starting from the constructor of class A, then class B and at the last constructor of class C.
It can be seen from the **output** of the program

```
In class A's constructor
In class B's constructor
In class C's constructor
```

It can be seen from the program that I have explicitly called the constructor of B from the constructor of class C using super() where as in the constructor of class B there is no [super](https://www.netjstech.com/2015/04/super-in-java.html) still the constructor of class A is called because default no-arg constructor of super class will be executed implicitly in that case.

**Points to note**

1. Constructor chaining in Java refers to the order in which constructors will be called in case there is hierarchy of classes.
2. Constructor of the superclass can be called from Subclass' constructor explicitly using super().
3. If super() is used in a constructor then it has to be the first statement in the constructor.
4. If super() is not used then the default no-arg constructor of each superclass will be executed implicitly.

That's all for this topic **Constructor Chaining in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!