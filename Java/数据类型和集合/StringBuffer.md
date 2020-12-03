### StringBuffer Class in Java With Examples

StringBuffer class in Java is the companion class of [Java String class](https://www.netjstech.com/2016/07/string-in-java.html). StringBuffer in Java is a **mutable** (modifiable) sequence of characters which is in contrast to String class which is an **immutable** sequence of characters. Thus in case of StringBuffer length and content of the sequence can be changed through certain method calls.

This characterstic of **StringBuffer** becomes very handy if you are doing a *lot of changes to your String class object by appending or inserting to that string*. Since StringBuffer in Java is mutable a new String object is not created every time string is modified, which in turn results in less memory consumptions and not having lots of intermediate String object for [garbage collection](https://www.netjstech.com/2017/11/garbage-collection-in-java.html).

### Constructors in Java StringBuffer class

- **public StringBuffer()**- Constructs a string buffer with no characters in it and an initial capacity of 16 characters.
- **public StringBuffer(int capacity)**- Constructs a string buffer with no characters in it and the specified initial capacity.
- **public StringBuffer(String str)**- Constructs a string buffer initialized to the contents of the specified string. The initial capacity of the string buffer is 16 plus the length of the string argument.
- **public StringBuffer(CharSequence seq)**- Constructs a string buffer that contains the same characters as the specified CharSequence. The initial capacity of the string buffer is 16 plus the length of the CharSequence argument.

### StringBuffer in Java is thread-safe

String buffers are safe for use by multiple threads. If you don't have to bother about thread safety then use [StringBuilder class](https://www.netjstech.com/2016/07/stringbuilder-in-java.html) as it supports all of the same operations but it is faster, as it performs no [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html).

### Capacity of Java StringBuffer

Every string buffer has a **capacity**. As long as the length of the character sequence contained in the string buffer does not exceed the capacity, it is not necessary to allocate a new internal buffer array. If the internal buffer overflows, it is automatically made larger.

### length and capacity methods in StringBuffer

Since we are talking about capacity here so it is very relevant to discuss length() and capacity() methods of Java StringBuffer here. As it becomes confusing for some people to distinguish between these two methods.

**public int capacity()**- Returns the current capacity. The capacity is the amount of storage available for newly inserted characters, beyond which an allocation will occur.

**public int length()**- Returns the length (character count).

So you can see where **length()** method returns the character count with in the string buffer the **capacity()** method will return the current capacity. So if you create an empty StringBuffer object its length will be 0 but capacity will be 16 (default).

```
public class SBCapacity {

 public static void main(String[] args) {
  StringBuffer sb = new StringBuffer();
  System.out.println("length " + sb.length());
  System.out.println("capacity " + sb.capacity());
 }
}
```

**Output**

```
length 0
capacity 16
```

Where as when you initialize StringBuffer with a string "Example" where length of the string is 7 then length method will return 7 where as capacity method will return 23 as in that case initial capacity of the string buffer is 16 plus the length of the string argument.

```
public class SBCapacity {
 public static void main(String[] args) {
  StringBuffer sb = new StringBuffer("Example");
  System.out.println("length " + sb.length());
  System.out.println("capacity " + sb.capacity());
 }
}
```

**Output**

```
length 7
capacity 23
```

### insert and append methods in StringBuffer class

The principal operations on a StringBuffer in Java are the **append** and **insert** methods. Most of the times you will use StringBuffer class over String when you are appending or inserting to a string. That won't result in creating lots of new string objects with every append as StringBuffer object is mutable.

append and insert methods are overloaded so as to accept data of any type. So there are overloaded versions which takes primitive data types like int, float, long, double as parameter apart from having versions which take String, StringBuffer, Object as parameter.

Each of these overloaded versions effectively converts a given datum to a string and then appends or inserts the characters of that string to the string buffer.

The append method always adds these characters at the end of the buffer; the insert method adds the characters at a specified point.

### Java StringBuffer append method example

```
public class SBDemo {

 public static void main(String[] args) {
  StringBuffer sb = new StringBuffer();
  StringBuffer sb1 = sb.append("This").append(" is").append(" Number ").append(1).append(" and ").append(1.101);
  System.out.println("After append -- " + sb.toString());
  
  if(sb == sb1){
   System.out.println("True");
  }else{
   System.out.println("false");
  }

  String str = new String();
  String str1 = str.concat("This").concat(" is");
  if(str == str1){
   System.out.println("True");
  }else{
   System.out.println("false");
  }
 }
}
```

**Output**

```
After append -- This is Number 1 and 1.101
True
false
```

Here note that append method is used with string and primitive data types as parameters and appended to the StringBuffer. Just to check whether the reference remains same or changes a new reference **sb1** of StringBuffer is created. It can be seen that **sb** and **sb1** both are pointing to the same StringBuffer object, new object is not created for every append.

Same thing is done with String and data is concatenated to the original string, here again another reference str1 is created but **str** and **str1** are not pointing to the same String object because new String object is created with every concatenation as String is immutable.

### Java StringBuffer insert method example

```
public class SBDemo {

 public static void main(String[] args) {
  StringBuffer sb = new StringBuffer("let");
  sb.insert(2, "n");
  System.out.println("After insert -- " + sb.toString());
 }
}
```

**Output**

```
After insert -- lent
```

### Java StringBuffer toString method

Another important method of StringBuffer class in Java is toString(), this method returns a string representing the data in this sequence. A new String object is allocated and initialized to contain the character sequence currently represented by this object. This String is then returned.

### Java StringBuffer reverse method

One more convenient utility method provided by StringBuffer class in Java is **reverse()** method, in String class there is no such method. In case you want to [reverse a string](https://www.netjstech.com/2016/05/how-to-reverse-string-in-java.html) with StringBuffer it is just a method call.

**public StringBuffer reverse()**- Causes this character sequence to be replaced by the reverse of the sequence.

```
public class SBRevDemo {

 public static void main(String[] args) {
  StringBuffer sb = new StringBuffer("Test String");
  sb.reverse();
  System.out.println("reversed - " + sb.toString());
 }
}
```

**Output**

```
reversed - gnirtS tseT
```

StringBuffer class also has [subString](https://www.netjstech.com/2016/07/string-charat-and-substring-methods-in-java.html) and [indexOf(), lastIndexOf()](https://www.netjstech.com/2016/07/searching-string-indexof-lastindexof-contains-java.html) methods which provide the same functionality as in String class.

That's all for this topic **StringBuffer Class in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!