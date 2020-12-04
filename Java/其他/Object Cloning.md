### Java Object Cloning - clone() Method

In this post we'll see what is Object cloning in java and what is shallow copy and deep copy in reference to Java object cloning.

In Java if you assign an object variable to another variable the reference is copied which means both variable will share the same reference. In this case any change in one object variable will be reflected in another.

For example, if there is a class Test and you create an object of that class and then assign that reference to another [object](https://www.netjstech.com/2015/04/object-in-java.html) variable.

```
Test obj1 = new Test();
Test obj2 = obj1;
```

Here both obj1 and obj2 will have the same reference.



### Java Object cloning

What is the option then if you want to quickly create an object using the existing object in such a way that you get a new instance (reference is not shared) with the same content for the fields in the new object as in existing object.

That’s when you can use **clone()** method which creates an exact copy of the existing object. Then you can modify the cloned object without those modification reflecting in original object (Well we’ll go into shallow copy and deep copy a little later).

### clone() method in Java

clone() method is defined as protected in the [Java Object class](https://www.netjstech.com/2017/06/object-class-in-java.html) which you must override as public in any derived classes that you want to clone.

**Signature of clone method in Object class**

```
protected native Object clone() throws CloneNotSupportedException;
```

### Process of object cloning in Java

There are two required steps if you want to clone any object.

1. You need to call clone() method of the Object class or provide your own implementation by overriding the clone() method in your class.
2. Your class, whose object you want to clone, must implement **Cloneable interface** which is part of java.lang package. Not implementing Cloneable [interface](https://www.netjstech.com/2015/05/interface-default-methods-in-java-8.html) will result in **CloneNotSupportedException** [exception](https://www.netjstech.com/2015/06/best-practices-for-exception-handling-java.html) being thrown when clone method is called.

Here note that Cloneable interface is a [marker interface](https://www.netjstech.com/2015/05/marker-interface-in-java.html) and defines no members of its own.

### Object cloning in Java example

When cloning an object in Java you need to call clone() method of the Object class. You can do it from a method or you can override clone() method of the Object class and then call the clone() method of the super class (Object class).

**Scenario 1- Calling clone() method from a method**

Here we have a class **Test** which implements **Cloneable interface** and it has a method **cloneIt()** which calls the **clone()** method of the Object class.

**Class Test**

```
public class Test implements Cloneable{
  int a;
  float f;
 
 Test cloneIt(){
  Test test = null;
  try {
   // Calling clone() method of Object class
   test = (Test)super.clone();
  } catch (CloneNotSupportedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  return test;
  
 }
}
```

**CloningDemo class**

```
public class CloningDemo {

 public static void main(String[] args) {
  Test t1 = new Test();
  t1.a = 10;
  t1.f = 13.4F;
  // Calling method to clone
  Test t2 = t1.cloneIt();
  System.out.println("t1.a " + t1.a + " t1.f " + t1.f);
  
  System.out.println("t2.a " + t2.a + "t2.f " + t2.f);

  if(t1 != t2){
   System.out.println("Different instances");
  }else{
   System.out.println("Same instances");
  }
 }
}
```

**Output**

```
t1.a 10 t1.f 13.4
t2.a 10t2.f 13.4
Different instances
```

Here you can see that **Test** class object **t1** is cloned and a new instance **t2** is created, in the code even reference equality is checked and you can see that the reference is not shared and both are indeed different instances.

**Points to note**

Some of the points to note from this code –

1. Class whose object has to be cloned should implement Cloneable interface, otherwise java.lang.CloneNotSupportedException exception will be thrown.
2. clone() method is a protected method in Object class, since all the classes inherit from Object class so you can call the protected method of the super class.
3. While cloning bitwise copy of the object is created.

**Scenario 2 – Overriding clone method**

Another way to provide object cloning functionality is to override the **clone()** method in the class in that case it has to be a public method in order to be accessible.

If we change the classes used in the above example to have overridden clone method and calling that clone method then the structure will be as follows–

**Test class**

```
public class Test implements Cloneable{
 int a;
 float f;
 // Override clone method
 public Object clone(){
  Object obj = null;
  try {
   // Calling clone() method of Object class
   obj = super.clone();
   } catch(CloneNotSupportedException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
  }
  return obj;
 }
}
```

**CloningDemo class**

```
public class CloningDemo {

 public static void main(String[] args) {
  Test t1 = new Test();
  t1.a = 10;
  t1.f = 13.4F;
  // Call clone method
  Test t2 = (Test)t1.clone();
  System.out.println("t1.a " + t1.a + " t1.f " + t1.f);
  
  System.out.println("t2.a " + t2.a + " t2.f " + t2.f);
  
  if(t1 != t2){
   System.out.println("Different instances");
  }else{
   System.out.println("Same instances");
  }
 }
}
```

**Output**

```
t1.a 10 t1.f 13.4
t2.a 10 t2.f 13.4
Different instances
```

### Advantages of object cloning

If you have an object, creation of which using the usual way is costly; **as example** if you have to call DB in order to get data to create and initialize your object. In that scenario rather than hitting DB every time to create your object you can cache it, clone it when object is needed and update it in DB only when needed.

Actually there is a design pattern called **prototype design pattern** which suggests the same approach.

### Shallow copy - Java object cloning

In the above examples only [primitive types](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html) are used so there is no problem, if you change any primitive value that won’t reflect in other object.

What if there is another object reference in your class? As already mentioned when you clone an object *all the values for the fields are copied* to the cloned object. Since [Java is pass by value](https://www.netjstech.com/2015/04/java-pass-by-value-or-pass-by-reference.html), if the *field value is a reference to an object (a memory address) it copies that reference to the field of the cloned object*. In that case referenced field is shared between both objects and any change made to the referenced field will be reflected in the other object too.

This process of cloning when the field values are copied to the new object is known as **shallow copy**. Shallow copies are simple to implement and typically cheap, as they can be usually implemented by simply copying the bits exactly.

### Deep Copy - Java object cloning

If you don’t want references of object being copied during the cloning process then option is **deep copy**. When a deep copy is done objects referenced by the cloned object are distinct from those referenced by original object, and independent.

Deep copies are more expensive, as you need to create additional objects, and can be substantially more complicated, due to references possibly forming a complicated graph.

- Refer [Shallow Copy And Deep Copy in Java Object Cloning](https://www.netjstech.com/2019/01/shallow-copy-and-deep-copy-in-java-object-cloning.html) to know more about shallow copy and deep copy in Java along with examples.

That's all for this topic **Java Object Cloning - clone() Method**. If you have any doubt or any suggestions to make please drop a comment. Thanks!