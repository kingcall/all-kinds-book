### Generic Class, Interface And Generic Method in Java

In the post [generics in Java](https://www.netjstech.com/2017/01/generics-in-java.html) basics of Java generics are already covered. In this post we’ll see how to create generic class, generic method and generic interface in Java.

### Generic class in Java

A generic class is defined with the following format:

```
class name<T1, T2, ..., Tn> { 
  /* ... */ 
} 
```

The type parameter section, delimited by angle brackets (<>), follows the class name. It specifies the type parameters (also called type variables) T1, T2, ..., and Tn.

### Java Generic class example

Let us create a generic class with two type parameters and see it in use with different data types.

```
class GenClass<K, V>{
  private K key;
  private V value;
  public GenClass(K key, V value) {
    this.key = key;
    this.value = value;
  }
  public K getKey(){
    return key;
  }
  public V getValue(){
    return value;
  }
}

public class GClassDemo {
  public static void main(String[] args) {
    GenClass<String, String> g1 = new GenClass<>("A", "Value A");
    System.out.println("Key- " + g1.getKey());
    System.out.println("Value- " + g1.getValue());
    
    GenClass<Integer, String> g2 = new GenClass<>(1, "Value 1");
    System.out.println("Key- " + g2.getKey());
    System.out.println("Value- " + g2.getValue());
  }
}
```

**Output**

```
Key- A
Value- Value A
Key- 1
Value- Value 1
```

Here you can see that first time [String](https://www.netjstech.com/2016/07/string-in-java.html) is passed as the type for both generic types K and V, where as it is passed as Integer and String second time.

### Generic Interface in Java

You can create a generic interface much the same way as a generic class but there are some conditions while implementing the generic [interface](https://www.netjstech.com/2015/05/interface-in-java.html).

### Java Generic interface example

```
public interface GInterface<E> {
  void setValue(E e);
  E getValue();
}
```

**Class implementing Generic interface**

```
import org.netjs.examples.interfaces.GInterface;

public class GClassImpl<E> implements GInterface<E> {
  E e;
  @Override
  public void setValue(E e) {
    this.e = e;   
  }

  @Override
  public E getValue() {
    return e;
  }
}
```

**Points to note here**

1. Here note that a class that implements a generic interface has to be a generic class.

   ```
   public class GClassImpl<E> implements Ginterface<E>
   ```

   If implementing class of the generic interface is not a generic class that will result in compile time error because the type parameter E is not known in that case.

   ```
   public class GClassImpl implements Ginterface<E>
   ```

   This will result in compile-time error.

2. Of course providing a proper data type with the interface while implementing it is OK, in that case normal class can be used.

   ```
   public class GClassImpl implements GInterface<String> {
     String str;
     @Override
     public void setValue(String e) {
       this.str = e;  
     }
   
     @Override
     public String getValue() {
       return str;
     }
   }
   ```

   Here you have used String as a type parameter with the interface so it is OK to use a normal class but type will become String in the class then.

3. A generic class implementing a generic interface can have other parameters too. This is perfectly ok -

   ```
   public class GClassImpl<E, K> implements Ginterface<E>
   ```

### Generic method in Java

Any method in the generic class can use the type parameter of the class so that way methods in a generic class are generic. Apart from that keep in mind the following points.

- Generic methods can add more parameters of their own.
- There can be generic methods even in a non-generic class.

When you are writing a generic method after the [access modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) you need to declare the type parameters then the return type. For example if you are writing a public method that uses one type parameter and doesn’t return anything then it will be written as-

```
public <T> void MethodName(T obj1){

} 
```

### Java Generics - generic method example

If you want to write a generic method that can be used to display elements of an array of any type.

```
public class GenericMethodDemo {  
  public static void main(String[] args) {
    GenericMethodDemo gm = new GenericMethodDemo();
    Integer[] intArray = {1, 2, 3, 4, 5, 6, 7};
    Double[] doubleArray = {3.4, 5.6, 7.8, 1.2, 4.5};
    // integer array
    gm.printElements(intArray);
    // double array
    gm.printElements(doubleArray);
  }
    
  public <T> void printElements(T[] arr){
    // Displaying elements
    for(int i = 0; i < arr.length; i++){
      System.out.print(" " + arr[i]);
    }
    System.out.println();
  }
}
```

**Output**

```
1 2 3 4 5 6 7
3.4 5.6 7.8 1.2 4.5
```

When you are calling a generic method there is no need to specify type (though you can do it if you want). Type will be inferred automatically based on the type of the method arguments. So calling your method using this form **gm.<Integer>printElements(intArray);** for Integer argument is also OK but it is not needed at all.

That's all for this topic **Generic Class, Interface And Generic Method in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!