### Type Erasure in Java Generics

When [generics](https://www.netjstech.com/2017/01/generics-in-java.html) was introduced in Java there was a requirement for it to be compatible with the existing code, written in previous versions, which of course was non-generic. That is why you can still add **raw types**, as example- **List alist = new ArrayList();**
This will give you warning for raw type but you can still have a non-generic list like this. Another thing that happens internally for compatibility is to replace all type parameters which is known as **type erasure in Java generics**

The Java compiler applies type erasure in generics to-

- Replace all type parameters in generic types with their bounds or Object if the type parameters are unbounded. Which means, if you have a unbounded type parameter that will be replaced by Object while compiling the code. If there is a [bounded parameter](https://www.netjstech.com/2017/02/bounded-type-parameter-in-java-generics.html) that will be replaced by the provided bound.
  The produced bytecode, therefore, contains only ordinary classes, interfaces, and methods.
- Insert type casts if necessary to preserve type safety.
- Generate **bridge methods** to preserve [polymorphism](https://www.netjstech.com/2015/04/polymorphism-in-java.html) in extended generic types.

**Table of contents**

1. [How type erasure works in Java](https://www.netjstech.com/2017/02/type-erasure-in-java-generics.html#TypeErasure)
2. [Bridge methods in Java](https://www.netjstech.com/2017/02/type-erasure-in-java-generics.html#BridgeMethod)
3. [Generics Ambiguity errors because of type erasure](https://www.netjstech.com/2017/02/type-erasure-in-java-generics.html#GenericsAmbiguity)



### How type erasure works in Java

As mentioned above, during the type erasure process, the Java compiler erases all type parameters and replaces each with its first bound if the type parameter is bounded, or Object if the type parameter is unbounded.

As example if you have the following generic class

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

here type parameter **T** is **unbounded**, thus the Java compiler replaces it with Object, so your compiled class won’t have any type parameter

```
public class GenType {
  Object obj;
  public Object getObj() {
    return obj;
  }

  public void setObj(Object obj) {
    this.obj = obj;
  }
}
```

If your class were using a **bounded parameter** as below-

```
public class GenType<T extends Number> {
  T obj;
  public T getObj() {
    return obj;
  }

  public void setObj(T obj) {
    this.obj = obj;
  }   
}
```

Here it will be replaced by the bound i.e. Number

```
public class GenType {
  Number obj;
  public Number getObj() {
    return obj;
  }

  public void setObj(Number obj) {
    this.obj = obj;
  }  
}
```

### Bridge methods in Java

When compiling a class or [interface](https://www.netjstech.com/2015/05/interface-in-java.html) that extends a parameterized class or implements a parameterized interface, the compiler may need to create a synthetic method, called a bridge method, as part of the type erasure process. In order to get an idea what bridge methods are let’s see an example.

If we have the following two classes-

**GenType**

```
public class GenType<T> {
  T obj;
  public GenType(T obj) { 
    this.obj = obj; 
  }

  public void setObj(T obj) {
    this.obj = obj;
  }  
}
```

**MyGen**

```
public class MyGen extends GenType<Integer> {
  public MyGen(Integer num) { 
    super(num); 
  } 
  public void setObj(Integer data) {
    System.out.println(" In MyGen.setData");
    super.setObj(data);
  } 
}
```

Here note that **MyGen** class extends the **GenType** class which has the Integer parameter. After compilation and type erasure, type parameters will be removed from both the classes and these classes will look as follows-

```
public class GenType {
  Object obj;
  public GenType(Object obj) { 
    this.obj = obj; 
  }

  public void setObj(Object obj) {
    this.obj = obj;
  }
}
public class MyGen extends GenType {
  public MyGen(Integer num) { 
    super(num); 
  } 
  public void setObj(Integer num) {
    System.out.println(" In MyGen.setData");
    super.setObj(num);
  }
}
```

If you have noticed, after type erasure **setObj()** method signatures don’t match in **GenType** and **MyGen** classes. In GenType class it becomes **setObj(Object obj)** where as in MyGen it becomes **setObj(Integer num)**.

Therefore, the MyGen setObj method does not override the GenType setObj method.

To solve this problem and preserve the polymorphism of generic types after **type erasure**, a Java compiler generates a **bridge method** to ensure that subtyping works as expected. For the **MyGen** class, the compiler generates the following bridge method for setObj:

```
public class MyGen extends GenType {
  public MyGen(Integer num) { 
    super(num); 
  } 

  // Bridge method generated by the compiler
  public void setObj(Object num) {
    setObj((Integer) num);
  }
  public void setObj(Integer num) {
    System.out.println(" In MyGen.setData");
    super.setObj(num);
  }
}
```

Here you can see a bridge method which has the same signature as the setObj method of the GenType class is inserted and it delegates to the actual setObj method.

### Generics Ambiguity errors because of type erasure

Let’s say you have the following class where you are trying to have two overloaded methods (set) with different type parameters K and V respectively.

```
class GenClass<K, V>{
  private K key;
  private V value;
  public GenClass(K key, V value) {
    this.key = key;
    this.value = value;
  }

  public void set(K key){
    this.key = key;
  }

  public void set(V value){
    this.value = value;
  }

  public K getKey(){
    return key;
  }
  public V getValue(){
    return value;
  }
}
```

This class will give compilation error
*Erasure of method set(K) is the same as another method in type GenClass<K,V>
Erasure of method set(V) is the same as another method in type GenClass<K,V>*

Though you may think since K and V are two different parameters so its ok to write overloaded methods using these parameters as arguments. But there is no such compulsion that these two parameters will be different, at the time of creating objects you can provide same type for both the type parameters.

Also type erasure will replace the type parameter with Object in this case so both the methods will become-

```
public void set(Object key){
 this.key = key;
}
public void set(Object value){
 this.value = value;
}
```

In a case like this it is better to give different name for the methods and avoid any ambiguity.

That's all for this topic **Type Erasure in Java Generics**. If you have any doubt or any suggestions to make please drop a comment. Thanks!