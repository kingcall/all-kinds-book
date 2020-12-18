

**Table of contents**

1. [How to create String in Java](https://www.netjstech.com/2016/07/string-in-java.html#createstring)
2. [String pool in Java](https://www.netjstech.com/2016/07/string-in-java.html#stringpool)
3. [Java String intern() method](https://www.netjstech.com/2016/07/string-in-java.html#stringintern)
4. [Java String is immutable](https://www.netjstech.com/2016/07/string-in-java.html#stringimmutable)
5. [String class in Java is final](https://www.netjstech.com/2016/07/string-in-java.html#stringfinal)
6. [String and thread-safety](https://www.netjstech.com/2016/07/string-in-java.html#stringthreadsafe)
7. [Overloaded operators in String](https://www.netjstech.com/2016/07/string-in-java.html#stringoverloadedoperators)
8. [Comparing Strings using .equals method](https://www.netjstech.com/2016/07/string-in-java.html#stringequalsmethod)
9. [Java String class methods](https://www.netjstech.com/2016/07/string-in-java.html#stringmethods)





### String pool in Java



When String object is created by assigning a string literal, pool will be checked to verify if there is any existing object with the same content if there is then that existing reference is used, no new object is created in that case. If no object is found with the same content then this new literal will be added in the pool.

For example, if two strings str1 and str2 are created as follows-

```
String str1 = "abc";
String str2 = "abc";
```

Then the string object reference is shared between these two literals.

[![String pool in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:53:57-String%252Bpool.png)](https://3.bp.blogspot.com/-kgriJV8blmg/V4e8tJhntdI/AAAAAAAAAR4/njhuhyo-gCoCTeHw5cvcHXOlNR9H7G-uQCLcB/s1600/String%2Bpool.png)

**String pool in Java**

Let’s see it with an **example**–

In this program two string literals will be created with the same content and then these two string objects are checked for reference equality. Since we are not comparing the content but the references of two objects so [“==” operator](https://www.netjstech.com/2017/06/difference-between-equals-method-and-equality-operator-java.html) will be used.

```
public class StringDemo {
 public static void main(String[] args) {
  String str1 = "abc";
  String str2 = "abc";
  if(str1 == str2){
   System.out.println("str1 and str2 are same");
  }else{
   System.out.println("str1 and str2 are not same");
  }
 }
}
```

**Output**

```
str1 and str2 are same
```

- Refer [String Pool in Java](https://www.netjstech.com/2019/07/string-pool-in-java.html) to know more about String pool.

### Java String intern() method

Using Java String's **intern()** method you can still get string object from the pool (if it exists) even if new operator is used to create a string.

When the intern method is invoked, if the pool already contains a string equal to this String object as determined by the equals(Object) method, then the string from the pool is returned. Otherwise, this String object is added to the pool and a reference to this String object is returned.

In the previous Java program if str4 is changed to have **interned string** then the code will look like–

```
public class StringDemo {

 public static void main(String[] args) {
  String str1 = "abc";
  String str2 = "abc";
  if(str1 == str2){
   System.out.println("str1 and str2 are same");
  }else{
   System.out.println("str1 and str2 are not same");
  }
  String str3 = new String("abc");
  String str4 = new String("abc").intern();
  if(str3 == str4){
   System.out.println("str3 and str4 are same");
  }else{
   System.out.println("str3 and str4 are not same");
  }
  
  if(str1 == str4){
   System.out.println("str1 and str4 are same");
  }else{
   System.out.println("str1 and str4 are not same");
  }
 }
}
```

**Output**

```
str1 and str2 are same
str3 and str4 are not same
str1 and str4 are same
```

It can be seen that str1 and str4 are having the same reference now.

### Java String is immutable

Once you create a String object the content of that **string cannot be modified**. As we have already seen Java maintains a string pool where references are shared thus changing content of any of the String will also affect the other strings sharing the same references that’s one reason why String is **immutable** in Java.

Here being [immutable](https://www.netjstech.com/2017/08/how-to-create-immutable-class-in-java.html) means whenever you perform any operation on string which alters its content a new string object is created which contains the modified string. Original string is left as it is. If there are no references to the original string it is garbage collected.

Using any of the methods that modify the original String like **toLowerCase**, **toUpperCase**, concatenating using **concatenate()** method or **‘+’** operator will result in creation of a new string object.

Let's try to see the immutability of the String using as example.

```
String str = "hello";
str.concat("world");
System.out.println("Value of str- " + str);
```

**Output**

```
Value of str- hello
```

You can see that the original String is not changed, when the concatenation is done a new String object is created.

- Refer [Why is String Immutable in Java](https://www.netjstech.com/2019/07/why-string-immutable-in-java.html) to know more about immutable strings and why are strings immutable in Java.

### String class in Java is final

As already mentioned above whenever you perform any operation on string which alters its content a **new string object is created** containing the modified string. Which means all the methods of the String class in Java that modify the content in any way return a new String object with the modified content.

Now, what if you can override the method of the String class and provide an implementation to modify the content of the String and return the original String itself? What will happen to the String pool then where strings having the same data share the same reference?

**Another scenario** - You extend the String class and [override hashCode() and equals() method](https://www.netjstech.com/2015/06/overriding-hashcode-and-equals-method.html) in such a way that two dissimilar strings return the same **hashCode** and at the same time **equals()** return true. Then you can have different strings sharing the same reference in the **String pool**.

To avoid these kind of scenarios String class is declared as [final in Java](https://www.netjstech.com/2015/04/final-in-java.html) and it can’t be overridden.

### String and thread-safety

Since String objects are immutable thus thread-safe.

- Refer [Is String Thread Safe in Java](https://www.netjstech.com/2016/08/string-and-thread-safety-in-java.html) to know more about String and thread safety.

### Overloaded operators in String

Apart from using concatenate method to concatenate two strings ‘+’ operator can be used to do the same. Actually + and += are two operators which are overloaded for String in Java.

So, if you have two strings

```
String str1 = "Hi";
String str2 = "Hello";
```

You can use ‘+’ operator to concatenate them

```
str1 = str1 + str2;
System.out.println("str1 " + str1);
```

Or, to make it more concise

```
str1 += str2;
System.out.println("str1 " + str1);
```

### Comparing Strings using .equals method

In the section about string pool we used == to compare references but what if you want to compare content of two strings even if their references are different. You will have to use **.equals** method in that case.

```
public class StringDemo {

 public static void main(String[] args) {
  String str1 = "abc";
  String str4 = new String("abc");
  // comparing content
  if(str1.equals(str4)){
   System.out.println("str1 and str4 are same");
  }else{
   System.out.println("str1 and str4 are not same");
  }
  // comparing references
  if(str1 == str4){
   System.out.println("str1 and str4 are same");
  }else{
   System.out.println("str1 and str4 are not same");
  }
 }
}
```

**Output**

```
str1 and str4 are same
str1 and str4 are not same
```

Though str1 and str4 have same content but they will have different references as str4 is created using new operator. That is why comparing using "==" prints "str1 and str4 are not same" as references are different but comparing using **.equals** prints "str1 and str4 are same", as content is compared in that case.

### Java String class methods

String class has lots of methods for various operations. These String class methods can be grouped as per functionality.

1. **Methods for String comparison**- In Java String class there are methods like equals, compareTo, regionMatches for comparing strings. Refer [String Comparison in Java](https://www.netjstech.com/2016/07/string-comparison-in-java-startswith-equals-method.html) to see examples of comparing strings in Java.
2. **Methods for searching in String**- If you want to find characters or substrings within a String you can use methods like indexOf() and lastIndexOf(). Refer [Searching Within a String Using indexOf(), lastIndexOf() And contains() Methods](https://www.netjstech.com/2016/07/searching-string-indexof-lastindexof-contains-java.html) to see examples of searching with in a String.
3. **Getting specific character from String**- If you are trying to get specific character from a String then you can use charAt() method. Refer [Java String charAt() Method With Examples](https://www.netjstech.com/2019/08/java-string-charat-method-with-examples.html) to see examples of getting characters from a String.
4. **Getting a substring**- If you are trying to get substring with in a String then you can use substring() method. Refer [Getting Substring - Java String substring() Method](https://www.netjstech.com/2016/07/string-charat-and-substring-methods-in-java.html) to see examples of getting substring from a String.
5. **Splitting a String**- If you want to split a string into one or more substrings then you can use split() method. Refer [Splitting a String Using split() Method in Java](https://www.netjstech.com/2016/07/splitting-string-using-split-method-in-java.html) to see examples of splitting a String in Java.
6. **Merging Strings**- For merging multiple strings in Java you can use join() method. Refer [String join() Method in Java](https://www.netjstech.com/2016/07/string-join-method-stringjoiner-class-in-java-8.html) to see examples of joining Strings in Java.
7. **Checking String null or empty**- For checking if the String is null or empty you can use isEempty(), length() or isBlank() method. Refer [Check String Null or Empty in Java](https://www.netjstech.com/2019/08/check-string-null-or-empty-java.html) to see examples.
8. **intern() Method**- For interning strings in Java you can use intern() method. Refer [intern() Method in Java String](https://www.netjstech.com/2019/08/intern-method-in-java-string.html) to know more about interning Strings.
9. **matches() Method**- Using matches() method you can check whether or not this string matches the given regular expression. Refer [matches() method in Java String](https://www.netjstech.com/2019/08/matches-method-in-java-string.html) to see examples of matches() method.

### Points to note about Java String

1. Internally in String class, Strings are stored as character array.
2. Strings in Java are objects and all strings are instances of the String class.
3. Strings can be created by assigning String literal directly to a String reference like **String str = “abc”;** which may look like assigning to a primitive data type but don't forget Strings are objects.
4. String literals in Java are treated differently, they are stored in a String pool and that is a common pool.
5. If there are two strings literals having the same content then those string will share the space in the pool.
6. String is immutable in Java once you create a String object the content of that string cannot be modified.
7. Since String is immutable in Java whenever you perform any operation on string which alters its content a new string object is created which contains the modified string. Original string is left as it is.
8. Since String is immutable it is also thread safe.
9. String class is declared as final and it can’t be overridden.
10. "+" operator is overloaded for String and it is used for concatenating strings.
11. Using **intern()** method you can still get string object from the pool (if it exists) even if **new** operator is used to create a string.
12. For comparing the content of two strings .equals() method is used. If you want to ignore case then use .equalsIgnoreCase().
13. From Java 7 string can also be used in [switch case statement](https://www.netjstech.com/2016/08/switch-case-statement-in-java.html).
14. **join() method** is added in String class in Java 8 which makes it very easy to join multiple strings.

That's all for this topic **String in Java Tutorial**. If you have any doubt or any suggestions to make please drop a comment. Thanks!