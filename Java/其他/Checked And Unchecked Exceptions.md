### Difference Between Checked And Unchecked Exceptions in Java

Before going into the differences between checked and unchecked exceptions in Java let's first see what these two types of exceptions are.

**Table of contents**

1. [Checked Exception in Java](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#CheckedException)
2. [Checked exception classes in Java](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#CheckedExceptionclasses)
3. [Unchecked Exception in Java](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#UncheckedException)
4. [Java's unchecked exceptions](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#UncheckedExceptionclasses)
5. [Checked Vs Unchecked exception in Java](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#CheckedVsUnchecked)
6. [Usage of checked exception in Java](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#Checkedusage)
7. [Why unchecked exception](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#Whyunchecked)
8. [Proper usage of Unchecked Exception](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html#Uncheckedusage)



### Checked Exception in Java

As we know from the exception hierarchy, **Throwable** is the parent class of all the Exception types. Immediately below Throwable there is a subclass called **Exception**. This Exception class has one subclass called **RunTimeException**.

[![difference between checked and unchecked exception in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:07:46-overview%252Bof%252Bexception.png)](https://1.bp.blogspot.com/-oTNPzMsGKJs/VVjYdQCGKXI/AAAAAAAAAH4/Wh9uLNtsZos/s1600/overview%2Bof%2Bexception.png)

If an exception is a subclass of Exception but does not inherit from RuntimeException, it is a **checked exception**. The restriction with a checked exception is that it **needs to be either caught in a method** (with in a [try-catch block](https://www.netjstech.com/2015/05/java-exception-handling-try-catch-block.html)), or the method needs to **specify that exception in a [throws clause](https://www.netjstech.com/2015/05/throws-keyword-in-java-exception-handling.html)** declaration.

### Checked exception classes in Java

- IOException
- FileNotFoundException
- ParseException
- ClassNotFoundException
- CloneNotSupportedException
- InstantiationException
- InterruptedException
- NoSuchMethodException
- NoSuchFieldException

### Unchecked Exception in Java

With in the exception hierarchy if an exception is a subclass of **RuntimeException**, it is an **unchecked exception**. An unchecked exception can also be caught by wrapping the code in try-catch block, but it does not have to as it is **not verified at compile time**. Thus the name **'unchecked'**.

Most of the times these exceptions occur due to the programming errors. Unchecked exceptions can be thought of arising due to not having a proper testability condition with in a program. **Take 3 examples**.

- **NullPointerException**- If already tested for null using if condition it won't arise.
- **ArrayIndexOutOfBoundsException**- If already tested for length of an [array](https://www.netjstech.com/2017/02/array-in-java.html) it won't arise.
- **ClassCastException**- if already checked using [instanceof operator](https://www.netjstech.com/2017/03/instanceof-operator-in-java.html) it won't arise.

### Java's unchecked exceptions

- ArrayIndexOutOfBoundsException
- ClassCastException
- IllegalArgumentException
- IllegalStateException
- NullPointerException
- NumberFormatException
- AssertionError
- ExceptionInInitializerError
- StackOverflowError
- NoClassDefFoundError

### Checked Vs Unchecked exception in Java

Now when we know what are checked and unchecked exceptions, it is easy to list out the differences between checked and unchecked exceptions in Java.

- **Checked Exception** is a direct subclass of Exception where as **unchecked exception** is a subclass of RunTimeException.
- **Checked exception** should be wrapped in a try-catch block or specified as throws clause where as there is no such requirement for **unchecked exception**.
- Failure to provide exception handling mechanism for **checked exception** result in compiler error whereas no compile time error for **unchecked exception**.
- **Checked exceptions** are designed to reduce the number of exceptions which are not properly handled and where there is a reasonable chance for recovery. **UnCheckedExceptions** are mostly programming errors.

### Usage of checked exception in Java

There is a lot of debate over whether checked exceptions are at all needed or not. General complain is that having checked exceptions result in a lot of boiler plate code. That problem has been recognized and Java7 has a feature called [multi-catch statement](https://www.netjstech.com/2015/05/multi-catch-statement-in-java-7.html) to reduce exception handling code.

For checked exceptions **Java language Specification** says; "*This compile-time checking for the presence of exception handlers is designed to reduce the number of exceptions which are not properly handled*".

**Checked Exceptions** should be used to declare expected errors from which there are chances to recover from. It doesn't make sense telling callers to anticipate exceptions that they cannot recover from.

**As example** If a user attempts to read from a non-existing file, the caller can prompt him for a new filename. On the other hand, if the method fails due to a programming bug (invalid input parameters or incorrect method implementation) there is nothing the application can do to fix the problem in mid-execution. The best it can do is log the problem and wait for the developer to fix it at a later time.

### Why unchecked exception

**According to the Java Language Specification** "*run-time exception classes are exempted because, in the judgment of the designers of the Java programming language, having to declare such exceptions would not aid significantly in establishing the correctness of programs. Many of the operations and constructs of the Java programming language can result in exceptions at run time. The information available to a Java compiler, and the level of analysis a compiler performs, are usually not sufficient to establish that such run-time exceptions cannot occur, even though this may be obvious to the programmer. Requiring such exception classes to be declared would simply be an irritation to programmers.*"

### Proper usage of Unchecked Exception

One of the [best practices for the exception handling](https://www.netjstech.com/2015/06/best-practices-for-exception-handling-java.html) is to preserve loose coupling. According to that an implementation specific checked exception should not propagate to another layer. As Exp. SQL exception from the DataAccessCode (DAO layer) should not propagate to the service (Business) layer. The general practice in that case is to convert database specific SQLException into an unchecked exception and throw it.

```
catch(SQLException ex){
    throw new RuntimeException("DB error", ex);
```

**Points to note-**

- Checked exception classes are direct descendant of Exception class.
- Unchecked exception classes are direct descendant of RunTimeException class, where RunTimeException class is the sub class of Exception class.
- Checked exception must be handled either using try-catch block or declared using throws, not doing that will result in compile time error.
- Not handling unchecked exception does not result in compile time error.
- RunTime and Checked exceptions are functionally equivalent, whatever handling or recovery checked exceptions can do, runtime exceptions can also do. For checked exceptions it has been made sure (forced) that handling is there by making it a compile time error.

That's all for this topic **Difference Between Checked And Unchecked Exceptions in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!