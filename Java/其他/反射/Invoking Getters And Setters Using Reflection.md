### Invoking Getters And Setters Using Reflection in Java

In the post [reflection in java – method](https://www.netjstech.com/2017/07/reflection-in-java-method.html) it is already explained how you can invoke a method of the class at runtime. In this post we’ll use that knowledge to **invoke getters and setters of the class using Java reflection API**. In Java you can do it using two ways.

- [Using the PropertyDescriptor class.](https://www.netjstech.com/2017/08/invoking-getters-and-setters-using-reflection-java.html#PropertyDescriptor)
- [Scanning methods of the class and look for set and get methods.](https://www.netjstech.com/2017/08/invoking-getters-and-setters-using-reflection-java.html#ScanningMethods)

In this post we'll see example of both of these ways to invoke getters and setters of the class.

### Using PropertyDescriptor class

A PropertyDescriptor describes one property that a Java Bean exports via a pair of accessor methods. Then using the **getReadMethod()** and **getWriteMethod()** you can call the setter and getter for the property.

### Invoking getters and setters using PropertyDescriptor example

Here in the example we have a class **TestClass** which has getter and setter for three fields which are of type int, String and boolean. Then in the **GetterAndSetter** class there are methods to invoke the getters and setters of the given class.

**TestClass.java**

```
package org.prgm;

public class TestClass {
 private int value;
 private String name;
 private boolean flag;
 public int getValue() {
  return value;
 }
 public void setValue(int value) {
  this.value = value;
 }
 public String getName() {
  return name;
 }
 public void setName(String name) {
  this.name = name;
 }
 public boolean isFlag() {
  return flag;
 }
 public void setFlag(boolean flag) {
  this.flag = flag;
 }
}
```

**GetterAndSetter.java**

```
import java.beans.IntrospectionException;
import java.beans.PropertyDescriptor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

public class GetterAndSetter {
 public static void main(String[] args) {
  GetterAndSetter gs = new GetterAndSetter();
  TestClass tc = new TestClass();
  gs.callSetter(tc, "name", "John");
  gs.callSetter(tc, "value", 12);
  gs.callSetter(tc, "flag", true);
  // Getting fields of the class
  Field[] fields = tc.getClass().getDeclaredFields();
  
  for(Field f : fields){
   String fieldName = f.getName();
   System.out.println("Field Name -- " + fieldName);
  }
  gs.callGetter(tc, "name");
  gs.callGetter(tc, "value");
  gs.callGetter(tc, "flag");
 }
 
 private void callSetter(Object obj, String fieldName, Object value){
  PropertyDescriptor pd;
  try {
   pd = new PropertyDescriptor(fieldName, obj.getClass());
   pd.getWriteMethod().invoke(obj, value);
  } catch (IntrospectionException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 
 private void callGetter(Object obj, String fieldName){
  PropertyDescriptor pd;
  try {
   pd = new PropertyDescriptor(fieldName, obj.getClass());
   System.out.println("" + pd.getReadMethod().invoke(obj));
  } catch (IntrospectionException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}
```

**Output**

```
Field Name -- value
Field Name -- name
Field Name -- flag
John
12
true
```

### Scanning methods of the class and look for set and get methods

Another way to invoke the getters and setters using Java reflection is to scan all the methods of the class through reflection and then find out which are the getters and setters method.

It is particularly useful to use this way to call get methods if you have lots of fields in a class. Calling set method that way won’t be of much help as you will have to still invoke individual method with the value that has to be set.

**Logic to identify get method**

get method starts with **get** or **is** (in case of boolean), it should not have any parameters and it should return a value.

**Logic to identify set method**

set method starts with **set** and it should have a parameter and it shouldn't return any value which means it should return void.

In the example same Java bean class as above **TestClass** is used.

In the **GetterAndSetter** class there are methods to identify the getters and setters of the given class. If it is a get method that is invoked to get the value. For set method invocation is done to set the property values.

```
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class GetterAndSetter {
  public static void main(String[] args) {
    TestClass tc = new TestClass();
    // get all the methods of the class
    Method[] methods = tc.getClass().getDeclaredMethods();
    // Initially calling all the set methods to set values
    for(Method method : methods){
      if(isSetter(method)){
        System.out.println("Setter -- " + method.getName());
        try {
          if(method.getName().contains("Name")) {
            method.invoke(tc, "John");
          } 
          if(method.getName().contains("Value")) {
            method.invoke(tc, 12);
          }
          if(method.getName().contains("Flag")) {
            method.invoke(tc, true);
          }
        } catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }    
      }
    }
    // calling getters
    for(Method method : methods){
      if(isGetter(method)){
        try {
          Object obj = method.invoke(tc);
          System.out.println("Invoking "+ method.getName() + " Returned Value - " + obj);
        } catch (IllegalAccessException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (IllegalArgumentException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (InvocationTargetException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      }   
    }
  }

  private static boolean isGetter(Method method){
    // identify get methods
    if((method.getName().startsWith("get") || method.getName().startsWith("is")) 
        && method.getParameterCount() == 0 && !method.getReturnType().equals(void.class)){
      return true;
    }
    return false; 
  }

  private static boolean isSetter(Method method){
    // identify set methods
    if(method.getName().startsWith("set") && method.getParameterCount() == 1 
        && method.getReturnType().equals(void.class)){
      return true;
    }
    return false; 
  }
}
```

**Output**

```
Setter -- setName
Setter -- setValue
Setter -- setFlag
Invoking getName Returned Value - John
Invoking getValue Returned Value - 12
Invoking isFlag Returned Value - true
```

That's all for this topic **Invoking Getters And Setters Using Reflection in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!