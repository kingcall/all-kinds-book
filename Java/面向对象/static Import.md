### static Import in Java With Examples

In order to access any [static](https://www.netjstech.com/2015/04/static-in-java.html) member (static field or method) of the class, it is necessary to qualify references with the class they came from.

ClassName.static_method()

With **static import** feature of Java 5, members defined in a class as public static can be used without qualifying it with the class name, in any Java class which does a static import. So, static import in Java allows unqualified access to the static member. This shortens the syntax required to use a static member.

### Syntax of static import in Java

Any Java program can import the members individually or en masse-

```
// accessing specific static variable of a class
import static package_name.class_name.static_variable;

// accessing specific static method of a class
import static package_name.class_name.static_method;

// accessing all the static members of a class en masse
import static package_name.class_name.*;
```

### Java static import Example

```
public class MyClass {
 public static final int COUNT = 1;
 public static int incrementCounter(int num){
  return num + COUNT;
 }
}
import static org.netjs.examples.MyClass.*;
public class StaticImportDemo {
 public static void main(String[] args) {
  // Accessing the static field of MyClass with out 
  //qualifying it with class name
  System.out.println("Count field of MyClass " + COUNT);
  // invoking the static method of MyClass with out 
  //qualifying it with class name
  System.out.println("Calling static method of MyClass " + incrementCounter(5));
 }
}
```

It can be seen that static import is done here to get all the static members of the MyClass,

**import static org.netjs.examples.MyClass.\*;**

After that the static field and methods of MyClass can be used with out qualifying them with Class.

### Benefits of static import in Java

- when you require frequent access to static members from one or two classes it is a good idea to use static import in order to avoid qualifying with class name every time.
- To avoid constant interface antipattern. Read about it [here](http://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) and [here](http://en.wikipedia.org/wiki/Constant_interface)

### Disadvantages of static import in Java

- Overuse of static import feature may result in unreadable and unmaintainable program. As it will be very difficult to recognize which class the particular static member comes from.

- Static import may also result in ambiguity. Consider a scenario where static members having the same name are imported from 2 different classes. As compiler will not be able to determine which one to use in absence of class name qualification, it will result in an error.

  For example in the following code all the members of both Integer and Long classes are static imported, while trying to access **MAX_VALUE** field compiler gives "*The field MAX_VALUE is ambiguous*" error as field MAX_VALUE is present in both INTEGER AND LONG classes. By doing a static import of all the members of both these classes we brought those members into the global namespace, thus the name collision.

  [![static import in java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:04:56-static%252Bimport.png)](https://4.bp.blogspot.com/-SR8yEa5PYnI/VUikEKqP1FI/AAAAAAAAAG4/-Eqw056ZRdg/s1600/static%2Bimport.png)

**Points to note-**

- **import static** is used to import static members of a class, you can import a **specific member** of class or **import en masse**.
- Used sparingly and in case of importing static members of one or two classes, static import in Java can increase readability of a program by removing the boilerplate of repetition of class names.
- Overuse of static import may result in unreadable code as it will be difficult to recognize from which class the particular static member comes from.
- There may be a **name collision** error in case 2 classes have the static variable of the same name and both of them are imported individually.

That's all for this topic **static Import in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!