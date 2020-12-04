### Type Wrapper Classes in Java

As explained in the post [primitive data types in Java](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html) there are eight primitive data types and most of the time you will use the primitive types in your code as it reduces the [object creation](https://www.netjstech.com/2017/02/object-creation-using-new-operator-java.html) overhead making it more efficient to use primitive types. But there are scenarios when you would want to use objects in place of primitives for that Java platform provides wrapper classes for each of the 8 primitive data types. These classes "wrap" the primitive in an object thus the name wrapper classes. Note that all the wrapper classes in Java are [immutable](https://www.netjstech.com/2017/08/how-to-create-immutable-class-in-java.html).

### Java Wrapper Classes

Eight wrapper classes used to wrap primitive data types are as given below-

| Primitive Type | Type Wrapper class |
| -------------- | ------------------ |
| boolean        | Boolean            |
| char           | Character          |
| byte           | Byte               |
| short          | Short              |
| int            | Integer            |
| long           | Long               |
| float          | Float              |
| double         | Double             |

Note that 6 of these are numeric and numeric wrapper classes are subclasses of the [abstract class](https://www.netjstech.com/2015/04/abstract-class-in-java.html) Number class in Java:

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/04/09:54:21-20201204095421.png)

### When do we need wrapper classes in Java

You need to use wrapper classes when you want an object holding primitive data, some of the scenarios where you will need wrapper classes are–

1. You want to add primitive value in an Object[] [array](https://www.netjstech.com/2017/02/array-in-java.html).
2. You want to add primitive type to any collection like [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html), [HashMap](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html) as you can add only objects to collection classes.
3. You want to use any of the utility function provided by the wrapper classes for converting values to and from other primitive types, for converting to and from strings, and for converting between number systems (decimal, octal, hexadecimal, binary).

### Java Wrapper classes examples

1. If you want to convert int to a float number.

   In Integer class there is a floatValue() method that can be used for the purpose.

   ```
   int num = 25;
   Integer i = new Integer(num);
   float fNum = i.floatValue();
   System.out.println("float Value " + fNum);
   ```

2. If you want to

    

   convert double value to a string

   .

   ```
   double d = 25.67;
   String str = Double.toString(d);
   System.out.println("string " + str);
   ```

3. If you want to know the min and max range of any type, like for integer

   ```
   System.out.println("Integer min value " + Integer.MIN_VALUE);
   System.out.println("Integer max value " + Integer.MAX_VALUE);
   ```

   **Output**

   ```
   Integer min value -2147483648
   Integer max value 2147483647
   ```

   **For double**

   ```
   System.out.println("Double min value " + Double.MIN_VALUE);
   System.out.println("Double max value " + Double.MAX_VALUE);
   ```

   **Output**

   ```
   Double min value 4.9E-324
   Double max value 1.7976931348623157E308
   ```

### Autoboxing and unboxing

Here [autoboxing and unboxing in Java](https://www.netjstech.com/2017/03/autoboxing-and-unboxing-in-java.html) should get an honorable mention; autoboxing and unboxing feature was added in Java 5 and it converts primitive into object and object into primitive automatically. In many cases now you don’t need to convert using utility methods as it will happen automatically.

**As example** you can directly assign int value to an Integer object–

```
Integer i = 25;
```

Now conversion and method call (**valueOf()**) in this case will be done by compiler.

Equivalent code if you were converting yourself –

```
int num = 25;
Integer i = Integer.valueOf(num);
```

That's all for this topic **Type Wrapper Classes in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!