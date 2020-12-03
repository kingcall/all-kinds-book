### Marker Interface in Java

Marker interface in Java is an interface that has **no method declarations or fields** in it. It is used as a tag to let the compiler know it needs to add some special behavior to the [class](https://www.netjstech.com/2015/04/class-in-java.html) implementing marker interface. That is why marker [interface](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html) is also known as **tag interface** in Java.

Some of the examples of marker interface in Java are-

- java.lang.Cloneable
- java.io.Serializable
- java.util.RandomAccess



If you see Java code for Serializable or Cloneable interface, which are Java marker interfaces, these are just empty interfaces.

**Serializable interface**

```
public interface Serializable {
}
```

**Cloneable interface**

```
public interface Cloneable {
}
```

So the question is what purpose does the marker interface serve when it is just an empty interface with no methods or variables of its own.

### Why marker interfaces in Java

The main purpose of Java marker interfaces is to create special types where types themselves have no behavior of their own.

In some cases the implementing class might only need to flag that it belongs to that particular type and some special behavior should be added to it, how and what is handled by some other entity.

Like in case of **Serializable marker interface**, classes that need to be [serialized](https://www.netjstech.com/2017/04/serialization-in-java.html) just implement Serializable interface. After that it is the task of **ObjectOutputStream** class to make sure that the classes that are implementing Serializable interface should be serialized properly. For example, if there is a class Address which you want to be serialized then Address class needs to implement Serializable interface.

```
public class Address implements Serializable{
  private String addressLine1;
  ....
  ....
}
```

Since Address class is implementing Serializable interface, it is of type Serializable. In the case of serializing an object, the entity which handles the serialization is ObjectOutputStream class. It is in the ObjectOutputStream class that it will check whether the object is of type Serializable.

```
if (obj instanceof Serializable)
```

If object is of type Serializable then object will be serialized otherwise **java.io.NotSerializableException** exception is thrown.

So, you can see Serializable which is a marker interface in Java is just providing the type here, actual serialization is done by ObjectOutputStream class. Address class by implementing Serializable is marking itself that it can be serialized. The actual serialization behavior is added by the ObjectOutputStream class.

### User defined marker interface in Java

You can create your own custom marker interface in Java. Here is an example using custom Marker interface in Java.

As we know marker [interfaces](https://www.netjstech.com/2015/05/interface-in-java.html) define a type, that can be used with [instanceof operator](https://www.netjstech.com/2017/03/instanceof-operator-in-java.html) to test whether object is an instance of the specified type.

```
public interface MarkerEntity {

}

public boolean save(Object object) throws InvalidEntityFoundException {
  if(!(object instanceof MarkerEntity)) {
    throw new InvalidEntityFoundException("Invalid Entity Found, can't be saved);
  } 
  return db.save(object);
}
```

Here save method makes sure that only the objects of classes that implement the **MarkerEntity** interface are saved, for other types **InvalidEntityFoundException** is thrown. So here **MarkerEntity** marker interface is defining a type that adds special behavior to the classes implementing it

### Annotations or Marker interface in Java

Though annotations can also be used now to mark classes for some special treatments but marker annotations are **replacement for naming pattern** not for Marker interfaces.
For example in Junit before annotations it was required to begin the test methods name with "test" (thus requiring a specific naming pattern), now we can use @test marker annotation to specify junit test methods.

Though in some cases where marker interfaces were previously used to provide metadata about the class now marker annotations can be used.

But **marker annotations can't fully replace the Java marker interfaces** because; marker interfaces are used to define type (as already explained above) where as marker annotations do not.

**Points to note-**

- Marker interface in Java is an interface that has no method declarations or fields in it.
- Marker interface is also known as tag interface.
- Marker interfaces define a type.
- We can create our custom marker interface.

That's all for this topic **Marker Interface in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!