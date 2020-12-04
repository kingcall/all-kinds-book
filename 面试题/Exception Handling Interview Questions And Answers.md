### Java Exception Handling Interview Questions And Answers

In this post interview questions and answers for exception handling in Java are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is Exception Handling?**

   Exception Handling in Java provides a way to handle a situation when an exception is thrown and shows a meaningful message to the user and continue with the flow of the program.

   When an exceptional condition occurs with in a method, the method (where the exception occurred) creates an Exception Object and throws it. The created exception object contains information about the error, its type and the state of the program when the error occurred.

   The method where the exception is thrown may handle that exception itself or pass it on. In case it passes it on, run time system goes through the method hierarchy that had been called to get to the current method to search for a method that can handle the exception.

   Five keywords used to manage Java exception handling

   - **try**- Any code that might throw an exception is enclosed within a try block.
   - **catch**- If an exception occurs in try block, catch block can provide exception handlers to handle it in a rational manner.
   - **finally**- The finally block always executes when the try block exits. So, any code that must execute after a try block is completed should be put in finally block.
   - **throw**- throw is used to manually thrown an exception.
   - **throws**- Any exception that is thrown in a method but not handled there must be specified in a throws clause.

   Read more about Exception handling in Java

    

   here

   .

2. ------

3. **Explain the exception hierarchy in Java?**

   Throwable class is the super class of all the exception types. Below Throwable class there are two subclasses which denotes two distinct branches of exceptions-

   - **Exception**- An Exception indicates that a problem has occurred, but it is not a serious system problem. The user programs you write will throw and catch Exceptions.
   - **Error**- It defines exceptions that are not expected to be caught by your program. Exceptions of type Error are used by the Java run-time system to indicate errors having to do with the run-time environment, itself.
     Examples of **error** are StackOverflowError, OutOfMemoryError etc.
   - Below Exception there is a distinct subclass **RunTimeExcpetion**- RunTimeExcpetion and its descendants denote the exceptional conditions that are external to the application, and the application usually cannot anticipate or recover from them.

   Read more about exception hierarchy in Java

    

   here

   .

4. ------

5. **What is the difference between Checked Exception and Unchecked Exception?**

   Checked Exception is a direct subclass of Exception where as unchecked exception is a subclass of RunTimeException.

   Checked exception should be wrapped in a try-catch block or specified as throws clause where as there is no such requirement for unchecked exception.

   Failure to provide exception handling mechanism for checked exception result in compiler error whereas no compile time error for unchecked exception.

   Checked exceptions are designed to reduce the number of exceptions which are not properly handled and where there is a reasonable chance for recovery. UnCheckedExceptions are mostly programming errors.

   Read more about difference between Checked Exception and Unchecked Exception

    

   here

   .

6. ------

7. **What is the difference between error and exception?**

   **Exception**- An Exception indicates that a problem has occurred, but it is not a serious system problem. The user programs you write will throw and catch Exceptions.

   **Error**- It defines exceptions that are not expected to be caught by your program. Exceptions of type Error are used by the Java run-time system to indicate errors having to do with the run-time environment, itself.
   **Examples** of error are StackOverflowError, OutOfMemoryError etc.

8. ------

9. **Is it necessary that each try block must be followed by a catch block?**

   No it is not mandatory that there should be a catch block after a try block. **try** block can have only a matching finally block. So there are these valid combinations try-catch-finally, try-catch, try-finally.

   Read more about try-catch block

    

   here

   .

10. ------

11. **What is finally block?**

    When an exception occurs in the code, the flow of the execution may change or even end abruptly. That may cause problem if some resources were opened in the method.
    **For example**, if a file was opened in a method and it was not closed in the end as some exception occurred then the resources may remain open consuming memory. finally provides that exception-handling mechanism to clean up.

    Code with in the finally block will be executed after a try/catch block has completed. The finally block will be executed whether or not an exception is thrown.

    Read more about finally block

     

    here

    .

12. ------

13. **Is it possible to have a finally block without catch?**

    Yes we can have a try-finally block, catch is optional. We can have these combinations try-catch-finally, try-catch, try-finally.

    Read more about finally block

     

    here

    .

14. ------

15. **Are you aware of any scenario when finally will not be executed?**

    According to [Java docs](https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html). If the JVM exits (By explicitly using System.exit() or a JVM crash) while the try or catch code is being executed, then the finally block may not execute. Likewise, if the thread executing the try or catch code is interrupted or killed, the finally block may not execute even though the application as a whole continues.

    Read more about finally

     

    here

    .

16. ------

17. **What is a nested try statement?**

    A try-catch-finally block can reside inside another try-catch-finally block that is known as nested try statement.

    ```
    public class NestedTryDemo {
      public static void main(String[] args) {
        try{
          System.out.println("In Outer try block");
          try{
            System.out.println("In Inner try block");
            int a = 7 / 0;
          }catch (IllegalArgumentException e) {
            System.out.println("IllegalArgumentException caught");
          }finally{
            System.out.println("In Inner finally");
          }
        }catch (ArithmeticException e) {
          System.out.println("ArithmeticException caught");
        }finally {
          System.out.println("In Outer finally");
        }
      }
    }
    ```

    Read more about nested try statement

     

    here

    .

18. ------

19. **What are multiple catch blocks?**

    There might be a case when a code enclosed with in a try block throws more than one exception. To handle these types of situations, two or more catch clauses can be specified where each catch clause catches a different type of exception. When an exception is thrown, each of the catch statement is inspected in order, and the first one whose type matches that of the thrown exception is executed.

    ```
    int a[] = {0};
    try{
      int b = 7/a[i];
    }catch(ArithmeticException aExp){
      aExp.printStackTrace();
    }catch(ArrayIndexOutOfBoundsException aiExp){
      aiExp.printStackTrace();
    }
    ```

    Read more about multiple catch blocks

     

    here

    .

20. ------

21. **What is exception propagation?**

    When an exceptional condition occurs within a method, the method (where the exception occurred) creates an Exception Object and throws it. The created exception object contains information about the error, its type and the state of the program when the error occurred.
    The method where the exception is thrown may handle that exception itself or pass it on. In case it passes it on, run time system goes through the method hierarchy that had been called to get to the current method to search for a method that can handle the exception.
    If your program is not able to catch any particular exception, that will ultimately be processed by the default handler. This process of going through the method stack is known as **Exception propagation**.

    Read more about exception propagation

     

    here

    .

22. ------

23. **What is throw keyword?**

    It is possible for a Java program to throw an exception explicitly that is done using the throw statement.

    The general form of throw is-
    throw throwableObject;

    We can get this throwableObject in 2 ways-

    - By using the Exception parameter of catch block.
    - Create a new one using the new operator.

    ```
    try{
       throw new NullPointerException();   
      }catch(NullPointerException nExp){
       System.out.println("Exception caught in catch block of displayValue");
       throw nExp;
      }
     }
    ```

    Read more about throw keyword

     

    here

    .

24. ------

25. **What is throws clause?**

    If in a method we don't want to handle any exception but want to leave it to the calling method to handle any exception that is thrown by the called method, it is done using throws keyword.

    Using throws a method can just declare the exception it may throw and callers of the method have to provide exception handling for those exceptions (or they can also declare them using throws).

    General form of a method declaration that includes a throws clause

    ```
    type method-name(parameter-list) throws exception-list
    {
    // body of method
    }
    ```

    Here, exception-list is a comma-separated list of the exceptions that a method can throw.

    Read more about throws clause

     

    here

    .

26. ------

27. **Difference between throw and throws?**

    - **throw** is used to throw an exception.
    - **throws** is used to declare an exception, in the method signature, that can be thrown from a method.

    Read more about difference between throw and throws

     

    here

    .

28. ------

29. **final Vs finally Vs finalize**

    - **final**- final keyword is used to restrict in some way. It can be used with variables, methods and classes. When a variable is declared as final, its value can not be changed once it is initialized. Except in case of blank final variable, which must be initialized in the constructor.
      If you make a **method final in Java**, that method can't be overridden in a sub class.
      If a **class is declared as final** then it can not be sub classed.

    - **finally**- finally is part of exception handling mechanism in Java. finally block is used with try-catch block. finally block is always executed whether any exception is thrown or not and raised exception is handled in catch block or not. Since finally block always executes thus it is primarily used to close the opened resources like database connection, file handles etc.

    - finalize()

      \- finalize() method is a protected method of java.lang.Object class. Since it is in Object class thus it is inherited by every class. This method is called by garbage collector thread before removing an object from the memory. This method can be overridden by a class to provide any cleanup operation and gives object final chance to cleanup before getting garbage collected.

      ```
      protected void finalize() throws Throwable
      {
        //resource clean up operations
      }
      ```

    Read more about final Vs finally Vs finalize

     

    here

    .

30. ------

31. **What are the rules of exception handling with respect to method overriding?**

    There are certain restrictions while overriding a method in case of exception handling in Java. Broadly there are two rules-

    - If superclass method has not declared any exception using throws clause then subclass overridden method can't declare any checked exception though it can declare unchecked exception.

    - If superclass method has declared an exception using throws clause then subclass overridden method can do one of the three things.

    - - sub-class can declare the same exception as declared in the super-class method.
      - subclass can declare the subtype exception of the exception declared in the superclass method. But subclass method can not declare any exception that is up in the hierarchy than the exception declared in the super class method.
      - subclass method can choose not to declare any exception at all.

    Read more about exception handling and method overriding

     

    here

    .

32. ------

33. **What is the error in the following code?**

    ```
    class Parent{
       public void displayMsg() throws IOException{
         System.out.println("In Parent displayMsg()");
        throw new IOException("Problem in method - displayMsg - Parent");
       }
    }
    public class ExceptionOverrideDemo extends Parent{
      public void displayMsg() throws Exception{  
        System.out.println("In ExceptionOverrideDemo displayMsg()"); 
        throw new Exception("Problem in method - displayMsg - ExceptionOverrideDemo");
      }  
    }
     
    ```

    Here parent class had declared IOException where as subclass has declared Exception. Exception is the super class of IOException thus it is wrong according to the rules of method overriding and exception handling. Thus the code will give compiler error.

    Read more about exception handling and method overriding

     

    here

    .

34. ------

35. **What is multi-catch statement in Java 7?**

    Before Java 7 multi-catch statement, if two or more exceptions were handled in the same way, we still had to write separate catch blocks for handling them.

    ```
    catch(IOException exp){
      logger.error(exp);
      throw exp;
    }catch(SQLException exp){
      logger.error(exp);
      throw exp;
    }
    ```

    With Java 7 and later it is possible to catch multiple exceptions in one catch block, which eliminates the duplicated code. Each exception type within the multi-catch statement is separated by Pipe symbol (|).

    ```
    catch(IOException | SQLException exp){
      logger.error(exp);
      throw exp;
    }
    ```

    Read more about multi-catch statement in Java 7

     

    here

    .

36. ------

37. **What is try-with-resources or ARM in Java 7?**

    Java 7 introduced a new form of try known as try-with-resources for Automatic Resource Management (ARM). Here resource is an object that must be closed after the program is finished with it. Example of resources would be an opened file handle or database connection etc.

    Before the introduction of try-with-resources we had to explicitly close the resources once the try block completes normally or abruptly.

    ```
    try {
        br = new BufferedReader(new FileReader("C:\\test.txt"));
        System.out.println(br.readLine());
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
      try {
        if (br != null){
          System.out.println("Closing the file");
          br.close();
        }                
      } catch (IOException ex) {
        ex.printStackTrace();
      }
    }
    ```

    try-with-resources helps in reducing such boiler plate code. Let's see the same example using try-with-resources.

    ```
    try(BufferedReader br = new BufferedReader(new FileReader("C:\\test.txt"))) {            
      System.out.println(br.readLine());
    } catch (IOException e) {
      e.printStackTrace();
    } 
    ```

    Read more about try-with-resources in Java 7

     

    here

    .

38. ------

39. **When is custom exception class needed? How to create a custom exception class?**

    According to Java Docs, you should write your own exception classes if you answer yes to any of the following questions; otherwise, you can probably use someone else's.

    - Do you need an exception type that isn't represented by those in the Java platform?
    - Would it help users if they could differentiate your exceptions from those thrown by classes written by other vendors?
    - Does your code throw more than one related exception?
    - If you use someone else's exceptions, will users have access to those exceptions? A similar question is, should your package be independent and self-contained?

    Read more about creating custom exception class

     

    here

    .