### Reflection in Java - Getting Class Information

In this post we'll see how to use [Reflection in Java](https://www.netjstech.com/2017/07/reflection-in-java.html) to get information about any class at run time.

For every type of object [JVM](https://www.netjstech.com/2015/05/what-are-jvm-jre-and-jdk-in-java.html) creates an immutable instance of **java.lang.Class** which provides methods to examine the runtime properties of the [object](https://www.netjstech.com/2015/04/object-in-java.html) including its members and type information. Class also provides the ability to create new classes and objects.

With the exception of **java.lang.reflect.ReflectPermission**, none of the classes in **java.lang.reflect** have public [constructors](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html). To get to these classes, it is necessary to invoke appropriate methods on Class. So you can say that the *entry point for all reflection operations is java.lang.Class*.



### How to get Class instance in Java

There are several ways to get a Class depending on whether the code has access to an object, the name of class, a type, or an existing Class.

**Object.getClass()**

If you have an instance of the class available, then you can get Class by invoking Object.getClass() method. This method will only work with reference types.

**For Example**– If you have a class called **ClassA** and an object **objA** of that class then you can get instance of Class as follows –

```
ClassA objA = new ClassA();
Class<?> cls = objA.getClass();
```

**Using .class Syntax**

If instance of the class is not available but the type is known, then you can get object of Class by using **.class** syntax. You just need to add “.class” to the name of the type. Using .class you can also get the Class for a primitive type.

**For Example**– If you want class object for type Boolean.

```
boolean b;
Class cls = boolean.class;
```

Note here that type (Boolean) is used to get instance of Class.

Same way if you want Class object for **ClassA**.

```
ClassA objA;
Class<?> cls1 = ClassA.class;
```

Then if you want to instantiate class object **objA** you can do that using **class.newInstance()** method.

```
try {
 objA = (ClassA)cls1.newInstance();
 objA.setI(12);
 System.out.println("Value of i " + objA.getI());
} catch (InstantiationException e) {
 // TODO Auto-generated catch block
 e.printStackTrace();
} catch (IllegalAccessException e) {
 // TODO Auto-generated catch block
 e.printStackTrace();
}
```

**Using Class.forName()**

You can also get the instance of the class using the [static](https://www.netjstech.com/2015/04/static-in-java.html) method **Class.forName()**.Make sure that you pass the fully-qualified class name. Which means you will have to include [package](https://www.netjstech.com/2016/07/package-in-java.html) names too.

**For example**– If you want instance of Class for **ClassA** which is residing in package **org.prgm** then do it as follows –

```
Class<?> c = Class.forName("org.prgm.ClassA");
```

**Using TYPE Field for Primitive Type Wrappers**

[Wrapper classes for the primitive types](https://www.netjstech.com/2017/03/typewrapper-classes-in-java.html) have a field named **TYPE** which is equal to the Class for the primitive type being wrapped.

```
Class cls1 = Integer.TYPE;
```

### Methods provided by class java.lang.Class

There are many methods provided by the class java.lang.Class to get the metadata about the class. Let’s go through an example to see some of these methods in practice.

- For the full list of methods please visit - https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Class.html

### Getting class information in Java using reflection

As a preparation for the example code let’s have a class called **Parent.java** which will be extended by the class **ChildClass.java** which is the class we are going to examine. There is also an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) **IntTest.java** which is implemented by ChildClass.java.

**Parent class**

```
public class Parent {
 String name;
 Parent(String name){
  this.name = name;
 }

 public void displayName(){
  System.out.println("Hello - " + name);
 }
 
 public String getName(){
  return name;
 }
}
```

**IntTest interface**

```
public interface IntTest {
   public void showValue();
}
```

**ChildClass.java**

```
public class ChildClass extends Parent implements IntTest{
 private int value;
 //Constructor
 public ChildClass(String name, int value) {
  super(name);
  this.value = value;
 }

 @Override
 public void showValue() {
  System.out.println("Value - " + value);
  
 }
}
```

Based on these classes let us see some of the methods provided by class Class.

### Getting class name using reflection

If you want to get the class name using the instance of the Class.

```
Class<?> c = Class.forName("org.prgm.ChildClass");
System.out.println("Class name " + c.getName());
System.out.println("Class name " + c.getSimpleName());
```

**Output**

```
Class name org.prgm.ChildClass
Class name ChildClass
```

As you can see using the method **getName()** gives you the fully qualified name whereas **getSimpleName()** method gives you only the class name.

### Getting super class using reflection

**getSuperClass()** method can be used for getting Super class. Returns the Class representing the superclass of the entity represented by this Class. If this Class represents either the Object class, an interface, a primitive type, or void, then null is returned. If this object represents an array class then the Class object representing the [Object class](https://www.netjstech.com/2017/06/object-class-in-java.html) is returned.

```
Class<?> c = Class.forName("org.prgm.ChildClass");
System.out.println("Super Class name " + c.getSuperclass());
```

**Output**

```
Super Class name - class org.prgm.Parent
```

### Getting implemented or extended interfaces using reflection

**getInterfaces()** method can be used for getting the interfaces. If the object on which you are calling this method represents a class, the return value is an [array](https://www.netjstech.com/2017/02/array-in-java.html) containing objects representing all interfaces implemented by the class. If this object represents an interface, the array contains objects representing all interfaces extended by the interface.

```
Class<?> c = Class.forName("org.prgm.ChildClass");
System.out.println("interfaces - " + Arrays.toString(c.getInterfaces()));
```

**Output**

```
interfaces - [interface org.prgm.IntTest]
```

### Getting class modifier using reflection

You can get the class modifier by using the getModifiers() method. Returns the Java language modifiers for this class or interface, encoded in an integer. The modifiers consist of the Java Virtual Machine's constants for public, protected, private, final, static, abstract and interface; they should be decoded using the methods of class Modifier.

```
Class<?> c = Class.forName("org.prgm.ChildClass");
   
int modifiers = c.getModifiers();
System.out.println("Modifier - " + Modifier.toString(modifiers));
```

**Output**

```
Modifier – public
```

### Getting fields of the class using reflection

There are 4 methods for getting fields of the class. If you want Field object for any specific field then you can use **getDeclaredField(String name)** or **getField(String name)** method. To get information for all the fields of the class you can following two methods.

- **getFields()**- Returns an array containing Field objects reflecting all the accessible public fields of the class or interface represented by this Class object.
- **getDeclaredFields()**- Returns an array of Field objects reflecting all the fields declared by the class or interface represented by this Class object. This includes public, protected, default (package) access, and private fields, but excludes inherited fields.

```
Class<?> c = Class.forName("org.prgm.ChildClass");

// Getting fields of the class
Field[] allFields = c.getFields();
System.out.println("All Fields - " + Arrays.toString(allFields));

// Getting fields of the class
Field[] fields = c.getDeclaredFields();
System.out.println("Fields - " + Arrays.toString(fields));
```

**Output**

```
All Fields - []
Fields - [private int org.prgm.ChildClass.value]
```

Since **ChildClass.java** doesn’t have any accessible public field (inherited or its own) so no value is returned for **getFields()** method.

There is one private field in **ChildClass.java**, getDeclaredFields() method shows that field.

- Refer [Reflection in Java - Field](https://www.netjstech.com/2017/07/reflection-in-java-field.html) to read in detail about reflection API for field.

### Getting constructors of the class using reflection

There are four methods for getting the constructors of the class. If you want [Constructor](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html) object for any specific constructor, then you can use **getConstructor(Class<?>... parameterTypes)** or **getDeclaredConstructor(Class<?>... parameterTypes)** method. To get all the constructors of the class you can use following two methods.

- **getDeclaredConstructors()**- Returns an array of Constructor objects reflecting all the constructors declared by the class represented by this Class object. These are public, protected, default (package) access, and private constructors.
- **getConstructors()**- Returns an array containing Constructor objects reflecting all the public constructors of the class represented by this Class object.

```
Class<?> c = Class.forName("org.prgm.ChildClass");

Constructor<?>[] constructors = c.getConstructors();
System.out.println("Constructors - " + Arrays.toString(constructors));
   
Constructor<?>[] Decconstructors = c.getDeclaredConstructors();
System.out.println("Declared constructors - " + Arrays.toString(Decconstructors));
```

**Output**

```
Constructors - [public org.prgm.ChildClass(java.lang.String,int)]
Declared constructors - [public org.prgm.ChildClass(java.lang.String,int)]
```

Since class ChildClass has only one public constructor so both methods are returning the same array of Constructor objects.

- Refer [Reflection in Java - Constructor](https://www.netjstech.com/2017/07/reflection-in-java-constructor.html) to read in detail about reflection API for constructor.

### Getting methods of the class using reflection

There are four methods for getting the methods of the class. If you want Method object for any specific constructor, then you can use **getMethod(String name, Class<?>... parameterTypes)** or **getDeclaredMethod(String name, Class<?>... parameterTypes)** method. To get all the methods of the class you can use following two methods.

- **getDeclaredMethods()**- Returns an array containing Method objects reflecting all the declared methods of the class or interface represented by this Class object, including public, protected, default (package) access, and private methods, but excluding inherited methods.
- **getMethods()**- Returns an array containing Method objects reflecting all the public methods of the class or interface represented by this Class object, including those declared by the class or interface and those inherited from superclasses and superinterfaces.

```
Class<?> c = Class.forName("org.prgm.ChildClass");

// Getting all methods (even inherited) of the class
Method[] methods = c.getMethods();
System.out.println("All Methods - " + Arrays.toString(methods));
   
// Getting methods of the class
methods = c.getDeclaredMethods();
System.out.println("Class Methods - " + Arrays.toString(methods));
```

**Output**

```
All Methods - [public void org.prgm.ChildClass.showValue(), public java.lang.String org.prgm.Parent.getName(), 
public void org.prgm.Parent.displayName(), public final void java.lang.Object.wait() throws java.lang.InterruptedException, 
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, 
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, 
public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), 
public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), 
public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]

Class Methods - [public void org.prgm.ChildClass.showValue()]
```

Here you can see how **getMethods()** shows all the methods even inherited one.

- Refer [Reflection in Java - Method](https://www.netjstech.com/2017/07/reflection-in-java-method.html) to read in detail about reflection API for method.

### Getting annotations using reflection

You can use following two methods in the class Class to get information about the annotations.

- **getAnnotations()**- Returns annotations that are present on this element. If there are no annotations present on this element, the return value is an array of length 0.
- **getDeclaredAnnotations()**- Returns annotations that are directly present on this element. This method ignores inherited annotations. If there are no annotations directly present on this element, the return value is an array of length 0.

```
Class<?> c = Class.forName("org.prgm.ChildClass");
   
Annotation[] annotations = c.getAnnotations();
System.out.println("Annotations - " + Arrays.toString(annotations));
   
Annotation[] Decannotations = c.getDeclaredAnnotations();
System.out.println("Annotations - " + Arrays.toString(Decannotations));
```

**Output**

```
Annotations - []
Annotations - []
```

Since there are no annotations present on the class so array length is zero for both methods. Note that these methods can be used in the same way with the object of Method or Field class to get annotations present on a method or a field.

That's all for this topic **Reflection in Java - Getting Class Information**. If you have any doubt or any suggestions to make please drop a comment. Thanks!