### Bounded Type Parameter in Java Generics

In the post [generics in Java](https://www.netjstech.com/2017/01/generics-in-java.html) you would have already seen examples where type parameters are replaced by any class type. But there are times when **you want to restrict the types** that can be used as type arguments in a parameterized type. That can be done using bounded type parameter in Java Generics.

**As example** if you have a [generic class](https://www.netjstech.com/2017/02/generic-class-interface-and-generic-method.html) with a method that operates on numbers, you would want to restrict it to accept instances of Number or its subclasses only.

Let’s first see an example where you don’t bind type parameters to analyse what happens in that case-

You have a generic class **Test** with a method **average** which returns the average of the numbers in the array passed to it. Since the class is generic so you intend to pass [array](https://www.netjstech.com/2017/02/array-in-java.html) of any type integer, double, float. Here return type of method average is double as you want an accurate value. Since Number class (Super class of all numeric classes) has **doubleValue()** method so you can always get the double value out of any type of number.

```
public class Test<T> {
  T[] numArr;
  Test(T[] numArr){
    this.numArr = numArr;
  }
  public double getAvg(){
    double sum = 0.0;
    for(int i = 0; i < numArr.length; i++){
      sum += numArr[i].doubleValue();
    }
    double avg = sum/numArr.length;
    return avg;
  }
}
```

This code will give you compile-time error-

*The method doubleValue() is undefined for the type T*

You get this error as there is no way for compiler to know type **T** will always be used for numeric classes. You need to let the compiler know that type T will always be Number and doubleValue() method can be called safely. That’s when you need **bounded type** to restrict the types that can be used for parameterized type. In the above case that restriction is; *the type should be Number*.

### Bounded type in Java generics

In order to create a bounded type you need to provide an upper bound which acts as a restriction for types. As this upper bound is a superclass, the type that can be used has to be a subclass of that upper bound.

### General form of bounded type parameter

To declare a bounded type parameter, list the type parameter's name, followed by the extends keyword, followed by its upper bound.

```
T extends superclass
```

In the example used above that upper bound has to be the Number class as Number class is the super class of all the numeric classes. Thus in that case your bounded type will be - **T extends Number**

### Java Generics - Bounded type parameter example

```
public class Test<T extends Number> {
  T[] numArr;
  Test(T[] numArr){
    this.numArr = numArr;
  }
  public double getAvg(){
    double sum = 0.0;
    for(int i = 0; i < numArr.length; i++){
      sum += numArr[i].doubleValue();
    }
    double avg = sum/numArr.length;
    return avg;
  }
}
```

Now you won’t get compile-time error as you have provided the **Number class as upper bound** for your generic type **T**. Which means any type passed for the generic type T has to be the sub class of class Number. Since **doubleValue()** method is in Number class it will be part of any sub class of Number through [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html). So no compile-time error!

### Multiple Bounds in Java generics

A type parameter can have multiple bounds:

```
<T extends B1 & B2 & B3>
```

A type variable with multiple bounds is a subtype of all the types listed in the bound. If one of the bounds is a class, it must be specified first. **For example**:

```
Class A { /* ... */ }
interface B { /* ... */ }
interface C { /* ... */ }

class D <T extends A & B & C> { /* ... */ }
```

Not specifying bounds in this order will result in compile-time error.

### Generic Methods and Bounded Type Parameters

You can also use bounded types with generic methods. Let’s see an example where it becomes necessary to use bounded types. Consider a method where you want to count the number of elements in an array greater than a specified element elem.

```
public static <T> int countGreaterThan(T[] anArray, T elem) {
  int count = 0;
  for (T e : anArray)
    if (e > elem)  // compiler error
      ++count;
  return count;
}
```

This method will result in compile-time error as greater than (>) operator can be used only with primitive types such as short, int, double, long, float, byte, and char. It can’t be used to compare objects, you have to use types that implement [Comparable interface](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) in order to compare objects. Thus, Comparable interface becomes the upper bound in this case.

**Code with upper bound**

```
public class Test{
  public <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray){
      if (e.compareTo(elem) > 0) {
        ++count;
      }
    }
    return count;
  }
}
```

You can use the following code to run it-

```
Test test = new Test();
Integer[] numArr = {5, 6, 7, 1, 2};
int count = test.countGreaterThan(numArr, 5);
System.out.println("count - " + count);
```

**Output**

```
count – 2
```

**Reference**- https://docs.oracle.com/javase/tutorial/java/generics/boundedTypeParams.html

That's all for this topic **Bounded Type Parameter in Java Generics**. If you have any doubt or any suggestions to make please drop a comment. Thanks!