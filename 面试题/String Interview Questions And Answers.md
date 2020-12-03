### Java String Interview Questions And Answers

In this post some of the Java String Interview Questions are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is String in Java?**

   In Java String class represents character strings which means; Strings in Java are objects and all strings are instances of the String class. Internally in String class Strings are stored as character array.

   Read more about String in Java

    

   here

   .

2. ------

3. **In how many ways String object can be created?**

   Since strings are objects so strings can of course be created using new operator. String class has more than 10 constructors to create Strings which ranges from taking nothing as parameter to taking char array, StringBuffer, StringBuilder, another String as argument.
   Another and more preferred way to create Strings is to assign String literal directly to a String reference as you will do for any primitive type. For every string literal Java will automatically constructs a String object.

   ```
   As example- String str = “abc”; 
   ```

   

   Read more about String in Java

    

   here

   .

4. ------

5. **If String can be created using String str = “test” then String is a primitive data type.Yes/No?**

   No. For every string literal Java automatically constructs a String object.

6. ------

7. **What is String pool? Where is it created in memory?**

   When String literals are created they are stored in a String pool and that is a common pool; which means if there are two strings literals having the same content then those string will share the space in the pool.
   When String object is created by assigning a string literal, pool will be checked to verify if there is any existing object with the same content if there is then that existing reference is used, no new object is created in that case. If no object is found with the same content then this new literal will be added in the pool.
   String pool is stored in the heap.

   Read more about String Pool in Java

    

   here

   .

8. ------

9. **What is immutable object? Is String object immutable?**

   An immutable object is an object that would not be able to change its state after creation. Thus immutable object can only be in one state and that state can not be changed after creation of the object.
   Yes String object is immutable. Once you create a String object the content of that string cannot be modified.

10. ------

11. **Why is String class immutable?**

    Since Java maintains a string pool where String references are shared thus changing content of any of the String will also affect the other strings sharing the same references that’s one reason why string is immutable.

    Read more about String immutability

     

    here

    .

12. ------

13. **Why is String class final in Java?**

    Since String is immutable, whenever you perform any operation on string which alters its content a new string object is created containing the modified string. Which means all the methods of the String class that modify the content in any way return a new String object with the modified content.
    Now, What if you can override the method of the String class so that it modifies and return the original string reference itself? In that case all the other strings having the same data in the string pool will also get affected as the reference is shared for the String literals having the same content.
    To avoid these kind of scenarios String class is declared as final and it can’t be overridden.

14. ------

15. **Which operator is overloaded for String?**

    ‘+’ operator is overloaded in Java for String. It is used for concatenating two strings.

16. ------

17. **How many objects will be created if two Strings are created this way?**

    ```
    String s1 = “test”; 
    String s2 =  “test”;
    ```

    

    

    Since s1 and s2 are string literals and having the same content object reference will be shared by them in the string pool. Therefore only one object is created.

18. ------

19. **How many objects will be created if two Strings are created this way?**

    ```
    String s1 = “test”;
    String s2 =  new String(“test”);
    ```

    

    

    In this case string literal goes to string pool and s2 is another object created using new. So, in this case two objects are created even if content is same.

20. ------

21. **How many objects will be created if two Strings are created this way?**

    ```
    String s1 = new String(“test”);
    String s2 =  new String(“test”);
    ```

    

    

    Two separate objects are created.
    If you check it using the following code

    ```
    if(s1 == s2){
      System.out.println("s1 and s2 are same");
    }else{
      System.out.println("s1 and s2 are not same");
    }
    ```

    s1 and s2 are not same will be displayed.

    

22. ------

23. **How many object will be created if Strings are created this way?**

    ```
    String s1 = “test”;
    String s2 =  new String(“test”);
    String s3 = new String(“test”).intern();
    ```

    

    

    s1 will go to string pool, for s2 new object is created. S3, though created using new will still search in the string pool for any reference having the same content as intern() method is used. So two objects will be created.

24. ------

25. **What is intern() method in String?**

    Using intern() method you can still get string object from the pool (if it exists) even if new operator is used to create a string.
    When the intern method is invoked, if the pool already contains a string equal to this String object as determined by the equals(Object) method, then the string from the pool is returned. Otherwise, this String object is added to the pool and a reference to this String object is returned.

    Read more about intern() method in Java

     

    here

    .

26. ------

27. **Is String thread safe in Java?**

    Yes string is thread safe in Java as String is immutable.

    Read more about String and thread safety in Java

     

    here

    .

28. ------

29. **What is StringBuffer in Java?**

    StringBuffer class is the companion class of String. StringBuffer is a mutable(modifiable) sequence of characters which is in contrast to String class which is an immutable sequence of characters. Thus in case of StringBuffer length and content of the sequence can be changed through certain method calls.
    Since StringBuffer is mutable a new String object is not created every time string is modified, which in turn results in less memory consumptions and not having lots of intermediate String object for garbage collection.

    Read more about StringBuffer in Java

     

    here

    .

30. ------

31. **What is StringBuilder in Java?**

    StringBuilder class (Added in Java 5),just like StringBuffer, is a mutable(modifiable) sequence of characters which is in contrast to String class which is an immutable sequence of characters. Thus in case of StringBuilder length and content of the sequence can be changed through certain method calls.

    Read more about StringBuilder in Java

     

    here

    .

32. ------

33. **Differences among String, StringBuffer and StringBuilder in Java?**

    String is immutable where as both StringBuffer and StringBuilder are mutable.
    String and StringBuffer are thread safe where as StringBuilder is not thread safe.

    Read more about String Vs StringBuffer Vs StringBuilder in Java

     

    here

    .

34. ------

35. **Is StringBuffer class also immutable in Java?**

    No, StringBuffer is not immutable.

    Read more about StringBuffer in Java

     

    here

    .

36. ------

37. **Is StringBuffer class also final in Java?**

    Yes, StringBuffer class is final in Java.

38. ------

39. **Is StringBuffer class thread safe?**

    Yes StringBuffer class is thread safe. Methods in StringBuffer class are synchronized.

40. ------

41. **Is StringBuilder class thread safe?**

    No StringBuilder class is not thread safe. That makes it faster than StringBuffer.

42. ------

43. **Is StringBuilder class also final in Java?**

    Yes StringBuilder class is final in Java.

    Read more about StringBuilder in Java

     

    here

    .

44. ------

45. **How to compare two strings in Java?**

    equals() method can be used for comparing two strings in Java. If you want to ignore case then you can use equalsIgnoreCase(String anotherString) method.
    There are also compareTo() and compareToIgnoreCase() methods for comparing two strings lexicographically. Returns an integer indicating whether this string is greater than (result is > 0), equal to (result is = 0), or less than (result is < 0) the argument.
    You can also use matches() method where you can pass a regular expression for matching strings.

    Read more about String comparison in Java

     

    here

    .

46. ------

47. **What will happen if “==” operator is used to compare two strings in Java?**

    “==” operator will compare the references of the strings not the content.

    ```
    String str1 = "abc";
    String str4 = new String("abc");
    ```

    Comparing these two strings using “==” operator

    ```
     if(str1 == str4)
    ```

    will return false as the references are different.

    

    ------

48. **How to get characters and substrings by index with in a String?**

    You can get the character at a particular index within a string by invoking the charAt() accessor method.

    ```
    String str = "Example String";
    char resChar = str.charAt(3);
    ```

    Will give char ‘m’. If you want to get more than one consecutive character from a string, you can use the substring method. The substring method has two versions -

    - **String substring(int beginIndex, int endIndex)** - Returns a new string that is a substring of this string.
    - **String substring(int beginIndex)** - Returns a new string that is a substring of this string.

    

    Read more about String charAt() and subString() methods in Java

     

    here

    .

    ------

49. **How can you find characters or substrings within a string?**

    To find characters or substrings with in a string indexOf() and lastIndexOf() methods can be used.
    You can also use contains() method
    **public boolean contains(CharSequence s)** - Returns true if and only if this string contains the specified sequence of char values. Otherwise it returns false.

    Read more about charAt() and subString() methods in Java

     

    here

    .

    ------

50. **How can you split a string in Java?**

    String provides a split method in order to split the string into one or more substring based on the given regular expression.
    **As example** If you have a string where one (or more) spaces are used and you want to split it around those spaces.

    ```
    String str1 = "split example    program";
    String[] strArray = str1.split("\\s+");
    ```

    

    Read more about Splitting a String using split() method in Java

     

    here

    .

    ------

51. **How can you join strings in Java?**

    With Java 8 join() method has been added in the String class which makes it very easy to join the multiple strings.
    join method has two overloaded versions -

    - **public static String join(CharSequence delimiter, CharSequence... elements)** - Returns a new String composed of copies of the CharSequence elements joined together with a copy of the specified delimiter.
    - **public static String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)** – Here elements is an Iterable that will have its elements joined together and delimiter is a sequence of characters that is used to separate each of the elements in the resulting String.

    

    Read more about String join() method in Java 8

     

    here

    .

    ------

52. **Can we use String in switch case statement?**

    Yes from Java 7 string can be used in switch case statement.