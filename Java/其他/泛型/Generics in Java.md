### Generics in Java

There have been many new feature additions in Java over the year, with introduction of [lambda expressions in Java 8](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) it has even moved toward functional programming. But long before the lambda expressions and [stream API](https://www.netjstech.com/2016/11/stream-api-in-java-8.html) there was one change which had a very big impact on the way you program and that was **generics** which was added in Java 5.

Generics in Java not only added a new way to declare classes, methods or interface but it also changed many of the classes in the API, Java collections API is an example. Initially it took some time getting used to the parameters like **T**, **K** or **V** with the classes or methods but now its quite a familiar thing and very much a part of your daily programming.

### Java Generics

Generics in Java enable you to specify type parameters when defining classes, [interfaces](https://www.netjstech.com/2015/05/interface-in-java.html) and methods. Here type parameter is the type of data (class or interface) which these classes, interfaces and methods will operate upon.

**As example**, when you say **List<T>** that means a list which will store elements of type T. Since you have specified a type parameter so this *List class is generic* and **T** is the *type parameter* which specifies the type of element stored in the [List](https://www.netjstech.com/2015/08/joining-merging-lists-in-java.html).

### Generic class format in Java

A [generic class](https://www.netjstech.com/2017/02/generic-class-interface-and-generic-method.html) is defined using the following format:

```
class name<T1, T2, ..., Tn> { 
  /* ... */ 
}
```

The type parameter section, delimited by angle brackets **(<>)**, follows the class name. It specifies the type parameters (also called type variables) T1, T2, ..., and Tn.

### Java Generics - Type Parameter Naming Conventions

By convention, generic type parameter names are single, upper case letters. That helps n distinguishing between type parameters and normal variables.

The most commonly used type parameter names are:

- E - Element (used extensively by the Java Collections Framework)
- K - Key
- N - Number
- T - Type
- V - Value
- S,U,V etc. - 2nd, 3rd, 4th types

### Benefits of Generics in Java

You might be thinking there is already an [Object class](https://www.netjstech.com/2017/06/object-class-in-java.html) which provides almost the same thing, as Object is super class of all the other classes so Object class can be used to make a class generic. Though that can be done but it may cause some unseen problems because there is no type safety.

Let’s say you have a List where you intend to store [strings](https://www.netjstech.com/2016/07/string-in-java.html) and you have not made it generic that means all its elements will be stored as objects of type Object-

```
List strList = new ArrayList();
strList.add("a");
strList.add("b");
strList.add(new Integer(4));
Iterator itr = strList.iterator();
while(itr.hasNext()){
 String str = (String)itr.next();
 System.out.println("" + str);
}
```

Since elements are stored as class Object type so there won’t be any compile time error if you store an integer too (as done in this line **strList.add(new Integer(4));**), even if you intended to store Strings.

Since the elements are stored as class Object type so you need to cast those elements to String while retrieving. At that time you will get a run time exception when the code will try to cast Integer to String.

So the code snippet shown above will throw run time [exception](https://www.netjstech.com/2015/05/how-to-create-custom-exception-class-java.html)-
*Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String*

That brings to the fore couple of benefits of using generics in Java-

- Stronger type checks at compile time

  \- In the above code if you use

   

  generics

   

  and specify the parameterized type of List as String you will get compile time error if an attempt is made to add element of any other type to the list. A Java compiler applies strong type checking to generic code and issues errors if the code violates type safety. Fixing compile-time errors is easier than fixing runtime errors, which can be difficult to find.

  **As example-**

  ```
  List<String> strList = new ArrayList<String>();
  strList.add("a");
  strList.add("b");
  strList.add(new Integer(4));
  ```

  With generics this code will throw following compile-time error if you try to add Integer to the list
  *The method add(String) in the type List<String> is not applicable for the arguments (Integer)*

- Casting is not required

  \- Another benefit you get with generics in Java is that cast is not required. Since you already specify the type with generics so explicit cast is not required.

  **As example-**

  ```
  List<String> strList = new ArrayList<String>();
  strList.add("a");
  strList.add("b");
  strList.add("c");
  
  Iterator<String> itr = strList.iterator();
  while(itr.hasNext()){
    String str = itr.next();
    System.out.println("" + str);
  }
  ```

  Here you see casting is not required when retrieving elements from the list.

- Enabling programmers to implement generic algorithms

  – In many algorithms, logic remains same, irrespective of the data they are used with. On the other hand, generics make it possible to write classes, methods and interface that work with different kind of data that too in a type safe manner. Now combine these two facts and the result you can write generic algorithms that work with any type of data.

  Let’s take a simple example to see how to create a generic class with parameterized type. While creating an object of the class you can specify the type, which in generics terminology is called generic type invocation, which replaces T with some concrete value.

  **Generic class**

  ```
  public class GenType<T> {
    T obj;
  
    public T getObj() {
      return obj;
    }
  
    public void setObj(T obj) {
      this.obj = obj;
    } 
  }
  ```

  **Using Generic class**

  ```
  public class GenericDemo {
    public static void main(String[] args) {
      // With Integer type
      GenType<Integer> genInt = new GenType<Integer>();
      genInt.setObj(21);
      int value = genInt.getObj();
      System.out.println("integer value " + value);
      
      // With String type
      GenType<String> genStr = new GenType<String>();
      genStr.setObj("Generic class test");
      String str = genStr.getObj();
      System.out.println("String value " + str);
      
      // With Double type
      GenType<Double> genDouble = new GenType<Double>();
      genDouble.setObj(34.56);
      double dblValue = genDouble.getObj();
      System.out.println("Double value " + dblValue);
    }
  }
  ```

  **Output**

  ```
  integer value 21
  String value Generic class test
  Double value 34.56
  ```

  Here you see that the same generic class is first invoked with Integer type, then String and then Double. And you get the added benefit of type safety and no chance of run time exception (which is a probability if you had used an Object class to make the class generic).

**Points to remember**

1. Generics in Java provide type safety as there are stronger type checks at compile-time.

2. Generics don’t work with primitive types, type arguments passed to the type parameters must be references.

3. You can have generic classes, generic methods and generic interfaces.

4. Type parameters are erased when generic classes are compiled.

5. A generic class with type parameters can’t have static members using the type parameter. As example -

   ```
   public class GenType<T> {
     static T obj;
     .....
     .....
   }
   ```

   This code will have compile-time error - Cannot make a static reference to the non-static type T.

6. In Java 7 and later you can replace the type arguments required to invoke the constructor of a generic class with an empty set of type arguments (<>) as long as the compiler can determine, or infer, the type arguments from the context. This pair of angle brackets, <>, is informally called the diamond.

   As example

   \- List<Integer> ls = new ArrayList<>();

   You don’t need to write <Integer> again, empty set of type argument (<>) will do, type will be inferred automatically.

That's all for this topic **Generics in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!