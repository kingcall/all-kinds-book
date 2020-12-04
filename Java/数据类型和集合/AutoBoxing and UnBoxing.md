### AutoBoxing and UnBoxing in Java

In Java 5 two new features Autoboxing and Unboxing were added. **Autoboxing in Java** automates the wrapping of [primitive data types](https://www.netjstech.com/2017/03/primitive-data-types-in-java.html) into the [wrapper class](https://www.netjstech.com/2017/03/typewrapper-classes-in-java.html) for that particular data type when an object instead of primitive type is needed.

**Unboxing in Java** does the exact opposite, that is it gets the value out of a wrapped object automatically.



### Autoboxing and Unboxing example

Before this feature autoboxing was added in Java, wrapping a primitive int to its corresponding wrapper class Integer would have looked like–

```
int num = 25;
Integer i = Integer.valueOf(num);
```

Now with Java autoboxing feature you can directly assign value to a wrapper class –

```
Integer i = 25;
```

Now **valueOf()** method call will be done by the compiler behind the scene.

Same way for unboxing in Java; getting a primitive's value out of a wrapped object previously meant calling the method intValue(), longValue(), doubleValue() based on the type

```
Integer i = new Integer(10);
int num = i.intValue();
```

Now it can be done directly –

```
Integer i = new Integer(10);
//unboxing
int num = i;
```

Or for float type

```
Float f = new Float(56.78);
float fNum = f;
```

### Benefits of Autoboxing and Unboxing in Java

As you have already seen, with this feature you don’t need to wrap primitive types or unwrap them manually. That is a great help with the other feature [generics](https://www.netjstech.com/2017/01/generics-in-java.html) also added in Java 5. Since generics work only with object so having the auto facility to box and unbox greatly simplifies the coding.

As the Collection classes also work only with objects so it makes working with collections easy too.

**As example –**

```
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 10; i++){
    li.add(i);
}
```

Even though you have a List which stores objects of type Integer you can still add primitive int directly to [List](https://www.netjstech.com/2015/08/list-iterator-in-java.html) because of Autoboxing. Earlier when the primitive ints were added to the list, behind the scene those values were wrapped into the Integer objects and then added to the [ArrayList](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html).

Same way unboxing also happens automatically.

```
int elem = li.get(0);
System.out.println("elem " + elem);
```

When you are trying to get the first element out of the list and assigned it to an int variable, then Integer is unboxed to get the int value out of the wrapped Integer class and that value is assigned to an int variable elem.

**More examples of autoboxing and unboxing in Java**

We have already seen few examples of assignments where primitive types are autoboxed into wrapper classes and the boxed objects are automatically unboxed to get the primitive values. But that’s not the only place you can see autoboxing and unboxing.

- You can see it in methods.
- You can see it in expressions.
- Due to Autoboxing and unboxing a wrapper class can be used in a switch statement.

### Autoboxing and Unboxing in method parameters

When an argument is passed to a method or a value is returned from a method autoboxing/unboxing can happen, if required.

Here the method **calcValue** takes Double object as a parameter but primitive value double is passed while calling the method thus the double value is autoboxed to a Double type object.

Then there is an expression also which has a mix of Double object and double value -

```
dOb = dNum + 13.4;
```

And it is again assigned to a Double type.

Again the return type of the method is double and the returned value is assigned to a Double object.

```
public class BoxDemo {

 public static void main(String[] args) {
  BoxDemo bd = new BoxDemo();
  Double dVal = bd.calcValue(15.6);
  System.out.println("Value " + dVal);
 }
 
 public double calcValue(Double dNum){
  Double dOb;
  dOb = dNum + 13.4;
  return dOb;
 }
}
```

### Autoboxing and Unboxing in switch statement

Due to unboxing Integer object can be used in a [switch-case statement](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html) too. With in the switch statement object is unboxed and the int value is used for the conditions.

```
Integer iVal = 7;
switch(iVal) {
 case 1: System.out.println("First step");
 break;
 case 2: System.out.println("Second step");
 break;
 default: System.out.println("There is some error");
}
```

### AutoBoxing and Unboxing overhead

There is some overhead involved in the process of autoboxing and unboxing in Java so make sure you don’t start using objects exclusively thinking autoboxing/unboxing will happen anyway behind the scenes. It will happen but don’t forget the involved overhead thus making the code a little less efficient.

That's all for this topic **AutoBoxing and UnBoxing in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!