### final Vs finally Vs finalize in Java

What are the differences among **final**, **finally** and [finalize in java](https://www.netjstech.com/2015/07/finalize-method-in-java.html) is one question asked a lot in [Java interviews](https://www.netjstech.com/p/core-java-interview-questions.html). This question is asked more to confuse a candidate as they all sound similar and of course by asking this question you get to assess the knowledge of the candidate about three things final, finally and finalize in Java!

Apart from having ‘final’ in all of these there is no similarity. Though it can be said that primary job of both finally and finalize is to clean up, so there is some similarity in functionality between finally and finalize.

So let’s go through differences among final, finally and finalize in Java.

### final Vs finally Vs finalize in Java

**final**– final keyword in Java is used to restrict in some way. It can be used with **variables, methods and classes**. When a **variable** is declared as final, its value can not be changed once it is initialized. Except in case of [blank final variable](https://www.netjstech.com/2015/04/final-in-java.html#finalvariable), which must be initialized in the [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html).
If you make a **method** final in Java, that method can’t be overridden in a sub class.
If a **class** is declared as final then it can not be sub classed.

- Refer [final in Java](https://www.netjstech.com/2015/04/final-in-java.html) for detailed information about final.



**finally**– finally is part of exception handling mechanism in Java. **finally block** is used with [try-catch block](https://www.netjstech.com/2015/05/java-exception-handling-try-catch-block.html). Along with a try block we can have both catch and finally blocks or any one of them. So we can have any of these combinations **try-catch-finally**, **try-catch**, **try-finally**. finally block is always executed whether any exception is thrown or not and raised exception is handled in catch block or not. Since finally block always executes thus it is **primarily used to close the opened resources** like database connection, file handles etc.
From Java 7 onwards [try with resources](https://www.netjstech.com/2015/05/try-with-resources-java7.html) provide another way to automatically manage resources.

```
try {
  // code that may throw an exception
} catch (Exception e) {
  // catch exception
} finally {
  //This block is always executed
  // so close any open resources here
}
```

- Refer [finally block in Java](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html) for detailed information about finally.



**finalize()**- finalize() method is a protected method of **java.lang.Object** class. Since it is in [Object class](https://www.netjstech.com/2017/06/object-class-in-java.html) thus it is inherited by every class. This method is called by garbage collector thread before removing an object from the memory. This method can be overridden by a class to provide any cleanup operation and gives object final chance to cleanup before getting garbage collected.

```
protected void finalize() throws Throwable
{
  //resource clean up operations
}
```

Please note that it is **not a good idea to rely** on finalize() method for closing resources as there is no guarantee when finalize() method will be called by Garbage collector.

- Refer [finalize method in Java](https://www.netjstech.com/2015/07/finalize-method-in-java.html) for detailed information about finalize.



That's all for this topic **final Vs finally Vs finalize in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!