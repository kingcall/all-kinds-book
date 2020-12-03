### Wildcards in Java Generics

In [Java generics](https://www.netjstech.com/2017/01/generics-in-java.html) there is also an option of giving question mark (**?**) as type which is called **wildcard** in Java generics. This question mark (?) represents an **unknown type**.

The wildcard can be used in a variety of situations-

- As the type of a parameter, field, or local variable;
- Sometimes as a return type (though it is better programming practice to be more specific).

### Types of wildcards in Java

Wildcards in Java generics can be classified into three types-

1. [Upper bounded wildcards](https://www.netjstech.com/2017/02/wildcard-in-java-generics.html#UpperBoundedWildcard)
2. [Lower bounded wildcards](https://www.netjstech.com/2017/02/wildcard-in-java-generics.html#LowerboundedWildcard)
3. [Unbounded wildcards](https://www.netjstech.com/2017/02/wildcard-in-java-generics.html#UnboundedWildcard)

### Upper bounded wildcards in Java generics

The way we can have [bounded parameters in generics](https://www.netjstech.com/2017/02/bounded-type-parameter-in-java-generics.html) same way we can have bounded wildcards too. So let’s see with an example when we should have upper bounded wildcards in Java generics.

Suppose you want to write a method that can take [list](https://www.netjstech.com/2015/08/joining-merging-lists-in-java.html) of any type of Number which means a method that works on lists of Number and the subtypes of Number, such as Integer, Double, and Float i.e. List<Integer>, List<Double>, and List<Number>.

Here **note** one thing, though Integer is a subtype of Number, List<Integer> is not a subtype of List<Number> and, in fact, these two types are not related. Same applies for Double and Float too. So you can’t have your method as follows-

```
public static  double sumOfElements(List<Number> list){
  double s = 0.0;
  for (Number n : list)
    s += n.doubleValue();
  return s;
}
```

If you try to write your method as above you can’t call it with a List with integers because of the reason stated above.

```
List<Integer> li = Arrays.asList(1, 2, 3);
System.out.println("sum = " + sumOfElements(li));
```

Trying to call the sumOfElements method with argument List<Integer> will result in compile-time error

*The method sumOfElements(List<Number>) in the type WildCardDemo is not applicable for the arguments (List<Integer>)*

In the case like this where you want your method to be generic enough to be able to work with a list of type Number or any of its subclasses. you can achieve this by using an upper bounded wildcard.

### General form of upper bounded wildcard

To declare an upper-bounded wildcard, use the wildcard character ('?'), followed by the extends keyword, followed by its upper bound.

```
<? extends UpperBoundType>
```

### Upper bounded wildcard Java example

To write the above mentioned method sumOfElements so that it works with Number or any of its subtype you need to provide a wild card and Number as the upper bound i.e. <? extends Number>. This will match Number or any of its subtype.

```
import java.util.Arrays;
import java.util.List;

public class WildCardDemo {
  public static void main(String[] args) {
    // With List<Integer>
    List<Integer> li = Arrays.asList(5, 6, 7);
    System.out.println("sum = " + sumOfElements(li));
    // With List<Double>
    List<Double> ld = Arrays.asList(1.2, 3.8, 8.2);
    System.out.println("sum = " + sumOfElements(ld));
  }
    
  public static double sumOfElements(List<? extends Number> list){
    double s = 0.0;
    for (Number n : list)
      s += n.doubleValue();
    return s;
  }
}
```

**Output**

```
sum = 18.0
sum = 13.2
```

### Lower Bounded Wildcards in Java Generics

The way Upper bounded wildcard restricts the unknown type to be a specific type or a subtype of that type same way lower bounded wildcard restricts the unknown type to be a specific type or a super type of that type.

### General form of the lower bounded wildcard

A lower bounded wildcard is expressed using the wildcard character ('?'), following by the super keyword, followed by its lower bound:

```
<? super bounded_type>
```

### Lower Bounded Wildcard Java Example

Suppose you want to write a method that put elements of type Integer into a List. If you want to make sure that method works for List<Integer>, List<Number>, and List<Object> — anything that can hold Integer values. Here note that Number and Object are super type of Integer.

That generic method can be written as follows by using the lower bounded wildcard where lower bound is Integer.

```
public static void addNumbers(List<? super Integer> list) {
  for (int i = 1; i <= 10; i++) {
    list.add(i);
  }
}
```

You can invoke this method using the following code -

```
// Using object
List<Object> lo = new ArrayList<Object>();
addNumbers(lo);
// Using Integer
List<Integer> li = new ArrayList<Integer>();
addNumbers(li);
```

**Note**- You can specify an upper bound for a wildcard, or you can specify a lower bound, but you cannot specify both.

### Unbounded wildcards in Java generics

Unbounded wildcards are, as the name suggests, wild cards *with out any upper or lower bound*. Unbounded wildcard type is specified using the wildcard character (?), for example, **List<?>**.This is called a list of unknown type.

There are two scenarios where an unbounded wildcard is a useful approach -

- If you are writing a method that can be implemented using functionality provided in the Object class.
- When the code is using methods in the generic class that don't depend on the type parameter. For example, **List.size** or **List.clear**.

### Java unbounded wildcard example

Suppose you want to write a method that can print elements of a List of any type. If you write that method using Object as type for the List, you will get an error when you pass it a List of integers.

```
public static void printElements(List<Object> list){
  for (Object elem : list){
    System.out.println(elem + " ");
  }
  System.out.println();
}
```

If you try to invoke this method using the following lines-

```
List<Integer> li = Arrays.asList(5, 6, 7);
printElements(li);
```

You will get **compile-time error**-

*The method printElements(List<Object>) in the type WildCardDemo is not applicable for the arguments (List<Integer>)*

You get this error because the above method prints only a list of Object instances; it cannot print List<Integer>, List<String>, List<Double>, and so on, because they are not subtypes of List<Object>. To write a generic **printElements** method, use List<?>:

```
import java.util.Arrays;
import java.util.List;

public class WildCardDemo {
  public static void main(String[] args) {
    // With List<Integer>
    List<Integer> li = Arrays.asList(5, 6, 7);
    printElements(li);
    // With List<Double>
    List<Double> ld = Arrays.asList(1.2, 3.8, 8.2);
    printElements(ld);
  }
    
  public static void printElements(List<?> list){
    for (Object elem : list){
      System.out.print(elem + " ");
    }
    System.out.println();
  }
}
```

**Output**

```
5 6 7 
1.2 3.8 8.2
```

Reference: https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html

That's all for this topic **Wildcard in Java Generics**. If you have any doubt or any suggestions to make please drop a comment. Thanks!