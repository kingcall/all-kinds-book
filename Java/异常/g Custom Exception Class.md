### Creating Custom Exception Class in Java

In this post we'll see how to create a custom exception in Java.

### custom exception in Java

Though Java's [exception handling](https://www.netjstech.com//2015/05/overview-of-java-exception-handling.html) covers the whole gamut of errors and most of the time it is recommended that you should go with standard exceptions rather than creating your own custom exception classes in Java, but you might need to create custom exception types to handle situations which are specific to your application.

### When to go for custom exception class in Java

According to [Java Docs](https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html), you should write your own custom exception classes if you answer yes to any of the following questions; otherwise, you can probably use someone else's.

- Do you need an exception type that isn't represented by those in the Java platform?
- Would it help users if they could differentiate your exceptions from those thrown by classes written by other vendors?
- Does your code throw more than one related exception?
- If you use someone else's exceptions, will users have access to those exceptions? A similar question is, should your package be independent and self-contained?



Think of creating a custom exception class only if you want to [throw an exception](https://www.netjstech.com/2015/05/throw-statement-in-java-exception-handling.html) that will give more readability to the exception as the matching exception type is not there in the existing exception classes in Java. Otherwise stick to one of the already existing exceptions.

**As Example**- Let's assume there is a code in which some logic is executed for senior citizens (i.e. only if entered age is greater than or equal to 60) otherwise it should throw exception. In that case you can create an **InvalidAgeException** as it will make it more readable and specific to your application logic.

Also in some applications they go with error codes and throw those error codes with in the exception message, in that case also we can create a custom exception with error code and caller program can handle these error codes through a utility method.

### Java custom exception example code

If we take the example of already mentioned **InvalidAgeException** then it can be implemented the following way.
**Note that**, here custom exception class is implemented as [unchecked exception](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html) (inheriting from RunTimeException) though custom exception can be implemented as checked exception too.

```
class InvalidAgeException extends RuntimeException{
 private int age;
 InvalidAgeException(){
  super();
 }
 InvalidAgeException(int age){
  this.age = age;
 }
 InvalidAgeException(String msg, int age){
  super(msg);
  this.age = age;
 }
 InvalidAgeException(String msg, Throwable cause, int age){
  super(msg, cause);
  this.age = age;
 }
 @Override
 public String toString(){
  return "InvalidAgeException for Age: " + getAge();
 }
 @Override
 public String getMessage(){
  return "InvalidAgeException for Age: " + getAge();
 }
 public int getAge() {
  return age;
 }
}

public class CustomExceptionDemo {

 public static void main(String[] args) {
  CustomExceptionDemo ceDemo = new CustomExceptionDemo();
  ceDemo.calculateExtraBenefits(45);
 }
 public void calculateExtraBenefits(int age){
  // If age is less than 60 throw exception
  if(age < 60){
   throw new InvalidAgeException(age);
  }
 }
}
```

**Output**

```
Exception in thread "main" InvalidAgeException for Age: 45
 at org.netjs.examples.impl.CustomExceptionDemo.calculateExtraBenefits(CustomExceptionDemo.java:42)
 at org.netjs.examples.impl.CustomExceptionDemo.main(CustomExceptionDemo.java:38)
```

### Custom Exception as checked Exception or Unchecked Exception

For deciding whether to write your own custom exception class in Java as **checked exception** or an **unchecked exception** the rules are as follows.

If user can take some action to recover from the expected error then make the custom exception a checked exception. On the other hand if user cannot do anything useful in case of error then make the custom exception as an unchecked exception (i.e. inheriting from RunTimeException).

Most of the time only function of the custom exceptions is to log an error; in that case it should definitely be an unchecked exception.

**Prefer unchecked exceptions for all programming bugs**: By programming bug I mean invalid input parameters or incorrect method implementation, there is nothing the application can do to fix the problem in mid-execution. The best it can do is log the problem and wait for the developer to fix it at a later time.

**Remember that RunTime and Checked exceptions are functionally equivalent**, whatever handling or recovery checked exceptions can do, runtime exceptions can also do. For checked exceptions it has been made sure (forced) that handling is there by making it a compile time error.
Though in Java 7; some steps had been taken to reduce the boiler plate code by introducing [try-with-resources](https://www.netjstech.com/2015/05/try-with-resources-java7.html) and [multi-catch statement](https://www.netjstech.com/2015/05/multi-catch-statement-in-java-7.html).

(For a more detailed discussion on differences between Checked exception and Unchecked exception refer [Checked exception Vs Unchecked exception](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html))

**Points to Note**

- Don't create new custom exception classes if they don't provide any useful information for client code. Use any standard exception instead.
- If the user can take some action to recover from the exception, make custom exception a checked exception. If the user cannot do anything useful, then make the exception unchecked.
- Generally custom exceptions are used for logging purpose only so better to make them unchecked exception. Anyway we can always catch an unchecked exception too in a try-catch block. It's just that compiler won't force us to do that.
- Try to provide the parameter that caused the exception in the exception message. As in the above example age is also appended in the exception message. That makes it easy for the programmer who is looking in to the cause of error.
- There are four constructors provided by Throwable class try to provide all of them in your custom exception class, at least no-arg, one with string (for message) and one which accepts another Throwable as cause. Last one is important in the case we are wrapping the originally thrown error in some custom exception and rethrowing it. In that case having original exception as cause parameter helps in getting to the root cause of the error.

That's all for this topic **Creating Custom Exception Class in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!