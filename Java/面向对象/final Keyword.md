### final Keyword in Java With Examples

**final keyword in java** has its usage in preventing the user from modifying a field, method or class.

- **final field**- A variable declared as final prevents the content of that variable being modified.
- **final method**- Final method in Java prevents the user from [overriding](https://www.netjstech.com/2015/04/method-overriding-in-java.html) that method.
- **final class**- Final class in Java can not be extended thus prevents [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html).

### final variable in Java

A variable can be declared as final in Java which prevents its contents from being modified. A final variable can be initialized only once but it can be done in two ways.

- Value is assigned when the variable is declared.
- Value is assigned with in a [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html).

Please note that the final variable that has not been assigned a value while declaring a variable is called **blank final variable** in Java, in that case it forces the constructor to initialize it.

### Java final variable examples

In the following examples you can see that any attempt to modify value of a final field results in compile time error.

[![final variable value can't be changed](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:57-final%252Bvar-1.png)](https://3.bp.blogspot.com/-viw_7igGmhc/VT--FpTGmFI/AAAAAAAAAFI/DAMr29eCNtw/s1600/final%2Bvar-1.png)



**Compilation error if value of a final variable is changed in the setter method.**



[![final variable value can't be changed](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:57-final%252Bvar-2.png)](https://1.bp.blogspot.com/-9FW-L9mnOho/VT--u0jmA1I/AAAAAAAAAFc/c2ksUq2AxLU/s1600/final%2Bvar-2.png)



**Compilation error if value of a final variable is changed using an object.**



### Final blank variable example

[![final blank variable in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:57-final%252Bblank%252Bvar.png)](https://3.bp.blogspot.com/-leeOtnmFTdY/VT-_GGdap0I/AAAAAAAAAFg/mVRX1VarXFM/s1600/final%2Bblank%2Bvar.png)

Compilation error that the blank final variable is not initialized. In that case this variable should be initialized in a constructor of the class.

### final object in Java

Making an object reference variable final is a little different from a normal variable as in that case **object fields can still be changed** but the **reference can't be changed**. That is **applicable to collections too**, a collection declared as final means only its reference can't be changed values can still be added, deleted or modified in that collection.

**Example of final object**

[![final object in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:58-final%252Bobj.png)](https://2.bp.blogspot.com/-WkReVTw-ufc/VT-_eSlUGtI/AAAAAAAAAFo/GzzQO3W4azo/s1600/final%2Bobj.png)

Here you can see that the value of field i can be changed but trying to change object reference results in compile time error.

### final method parameter



Parameters of a method can also be declared as final in Java to make sure that parameter value is not changed in the method. Since [java is pass by value](https://www.netjstech.com/2015/04/java-pass-by-value-or-pass-by-reference.html) so changing the parameter value won't affect the original value anyway. But making the parameter final ensures that the passed parameter value is not changed in the method and it is just used in the business logic the way it needs to be used.



**As example**

```
private static Employee getData(final String empId, final String lastName, final String firstName, final int age){
  ...
  ...
}
```

### Final variables and thread safety

Variables declared as final are essentially read-only thus [thread safe](https://www.netjstech.com/2016/08/string-and-thread-safety-in-java.html).

Also if any field is final it is ensured by the [JVM](https://www.netjstech.com/2015/05/what-are-jvm-jre-and-jdk-in-java.html) that other threads can't access the **partially constructed object**, thus making sure that final fields always have correct values when the object is available to other [threads](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html).

### How final keyword in Java relates with inheritance

The use of final with the method or class in Java relates to the concept of [inheritance](https://www.netjstech.com/2015/04/inheritance-in-java.html). You can have a final method or a final class in Java ensuring that a *method declared as final can't be overridden* and the *class declared as final can't be extended*.

### final method in Java

A method can be declared as final in order to avoid [method overriding](https://www.netjstech.com/2015/04/method-overriding-in-java.html). Method declared as final in super class cannot be overridden in subclass.

If creator of the class is sure that the functionality provided in a method is complete and should be used as is by the sub classes then the method should be declared as final in Java.

### Benefit of final method

Final method in Java may provide performance enhancement. Generally in Java, calls to method are resolved at run time which is known as **late binding**.

Where as in case of final methods since compiler knows these methods can not be overridden, call to these methods can be resolved at compile time. This is known as **early binding**. Because of this early binding compiler can inline the methods declared as final thus avoiding the overhead of method call.

### Final method example in Java

[![final method in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:58-final%252Bmethod-1.png)](https://1.bp.blogspot.com/-IXbGFc42GOw/VT-_0WKKbmI/AAAAAAAAAFw/R9Wftg0T2so/s1600/final%2Bmethod-1.png)

### final Class in Java

A class declared as final can't be extended thus avoiding inheritance altogether.

If creator of the class is sure that the class has all the required functionality and should be used as it is with out extending it then it should be declared as final in Java.

Declaring a class as final implicitly means that all the methods of the class are final too as the class can't be extended.

It is **illegal to declare a class as both final and abstract**. Since abstract class by design relies on subclass to provide complete implementation.

### Java final class example

[![final class in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:47:58-final%252Bclass.png)](https://3.bp.blogspot.com/-49WaCrvndp4/VT_AIqJj81I/AAAAAAAAAF4/TdGJHzpH88Y/s1600/final%2Bclass.png)

### Benefits of final class in Java

- Since all the methods in a final class are implicitly final thus final class may provide the same performance optimization benefit as final methods.
- Restricting extensibility may also be required in some cases essentially when using third party tools.
- A class can be declared final when [creating Immutable Class in Java](https://www.netjstech.com/2017/08/how-to-create-immutable-class-in-java.html).

**Points to note-**

- final keyword in Java can be used with fields, methods and classes.
- final variables are essentially read only and it is a common convention to write the final variables in all caps. **As example**
  Private final int MAXIMUM_VALUE = 10
- final variables that are not initialized during declaration are called blank final variable in that case it forces the constructor to initialize it. Failure to do so will result in compile time error.
- final variables can be initialized only once, assigning any value there after will result in compile time error.
- final variables are read-only thus thread safe.
- In case of final object, reference can't be changed.
- final methods can't be overridden
- final classes can't be extended.
- final methods are bound during compile time (early binding), which means compiler may inline those methods resulting in performance optimization.

That's all for this topic **final Keyword in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!