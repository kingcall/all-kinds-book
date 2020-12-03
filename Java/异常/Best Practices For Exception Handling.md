

https://www.netjstech.com/2015/06/best-practices-for-exception-handling-java.html

### Best Practices For Exception Handling in Java

If you know about these 5 keywords try, catch, [finally](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html), [throw and throws](https://www.netjstech.com/2015/05/difference-between-throw-and-throws-java.html) and how to use them, you pretty much know [what Java exception handling is](https://www.netjstech.com/2015/05/overview-of-java-exception-handling.html) all about.

But there are also some best practices for exception handling in Java which you should try to follow. That is what this post talks about; the do’s and don'ts of the exception handling in Java.

### Java Exception Handling Best Practices

1. Use Specific Exceptions not Exception or Throwable

   \- It is always better to

    

   throw specific exception

    

   (specific exception sub-classes) rather than the more generic one (

   i.e. super class

   ) like

    

   Throwable

   ,

    

   Exception

    

   or

    

   RunTimeException

   .

   By doing that we can provide more information to the user what exactly went wrong, the code is also more readable by giving info about various exceptions it can throw rather than everything gobbled up by Exception or Throwable class.

   We should be specific when

    

   catching exceptions

    

   too.

    

   As Example

   \- For RunTimeExceptions (unchecked exceptions) it is said that we should not catch them as they indicate application code errors. If we catch

    

   Exception

    

   class directly we also catch RuntimeExceptions as RuntimeException class inherits from Exception.

   Same way if we catch throwable directly that is also wrong-

   ```
   try {
   } catch(Throwable t) {
       t.printStackTrace();//Should not do this
   }
   ```

   Throwable is the superclass of all errors and exceptions in Java. Error is the superclass of all errors which are not meant to be caught by applications. *Thus, catching Throwable would essentially mean that Errors such as system exceptions (e.g., OutOfMemoryError, StackOverFlowError) would also get caught*. And, the recommended approach is that application should not try and recover from Errors such as these. Thus, **Throwable and Error** classes should not be caught. Only Exception and its subclasses should be caught.

2. Throw Early or Fail-Fast

   \- Since an exception stack trace shows the exact sequence of method calls till the point of exception, along with class name, file name and the line number where the exception occurs it becomes very important to throw exception as early as possible.

   Let's see what will happen if we don't do that.

   ```
   public class ShowFile {
     public static void main(String[] args) {
       File propFile = new File("");
       try{
         readFile(propFile);
       }catch (FileNotFoundException e){
         e.printStackTrace();
       }catch (EOFException e){
         e.printStackTrace();
       }catch (IOException e){
         e.printStackTrace();
       }
     }
       
     private static void readFile(File filename) throws 
         FileNotFoundException, EOFException{
       InputStream in = new FileInputStream(filename);       
     }
   }
    
   ```

   **StackTrace**

   ```
   java.io.FileNotFoundException: 
    at java.io.FileInputStream.open0(Native Method)
       at java.io.FileInputStream.open(Unknown Source)
       at java.io.FileInputStream.<init>(Unknown Source)
       at org.netjs.example.ShowFile.readFile(ShowFile.java:37)
       at org.netjs.example.ShowFile.main(ShowFile.java:18)
   ```

   In the stack trace it is a little difficult to point out the real origin of exception, seeing the stack trace it looks like problem is in **FileInputStream** though in reality, problem in the code is that there is no check for space passed as file name.

   Let's see how we should do that.

   ```
   public class ShowFile {
     public static void main(String[] args) {
       File propFile = new File("");
       try{
         readFile(propFile);
       } catch (FileNotFoundException e){
         e.printStackTrace();
       } catch (IOException e){
         e.printStackTrace();
       }
     }
       
     private static void readFile(File fileName) throws FileNotFoundException, 
         EOFException, IllegalArgumentException{
       if(fileName == null || fileName.getPath().equals("")){
         throw new IllegalArgumentException("File Name not given");
       }
       InputStream in = new FileInputStream(fileName);  
     }
   }
   ```

   **StackTrace**

   ```
   Exception in thread "main" java.lang.IllegalArgumentException: File Name not given
    at org.netjs.example.ShowFile.readFile(ShowFile.java:42)
    at org.netjs.example.ShowFile.main(ShowFile.java:18)
   ```

   It can be seen now with the check for the filename the stack trace gives precise information about the problem.

3. Catch Late

   \- In case of

    

   checked exceptions

   , it is enforced by Java compiler to either catch the exception or declare it in

    

   throws clause

   . So generally developer tends to catch it and do nothing, except printing stacktrace or put a logger, in order to avoid the compiler error. But that way we are not providing the true information of what exactly happened.

   Lets see an example-

   ```
   public class ShowFile {
     public static void main(String[] args) {
       File propFile = new File("");
       readFile(propFile);        
     }
       
     private static void readFile(File fileName) {
       InputStream in = null;
       try {
         in = new FileInputStream(fileName);
       } catch (FileNotFoundException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
       }
       try {
         in.read();
       } catch (IOException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
       }        
     }
   }
   ```

   **Stacktrace**

   ```
   java.io.FileNotFoundException: 
       at java.io.FileInputStream.open0(Native Method)
       at java.io.FileInputStream.open(Unknown Source)
       at java.io.FileInputStream.<init>(Unknown Source)
       at org.netjs.example.ShowFile.readFile(ShowFile.java:41)
       at org.netjs.example.ShowFile.main(ShowFile.java:18)
   Exception in thread "main" java.lang.NullPointerException
       at org.netjs.example.ShowFile.readFile(ShowFile.java:47)
       at org.netjs.example.ShowFile.main(ShowFile.java:18)
   ```

   Here you can see that instead of declaring the exception in throws clause and catching it at the point where it can be handled, [try-catch block](https://www.netjstech.com/2015/05/java-exception-handling-try-catch-block.html) is used and the exception is caught using **FileNotFoundException**. If we see the stack trace it is not providing appropriate and precise information why the exception occurred.

   It is better to catch exception only when it can be handled appropriately. We can use throws clause to declare the exception and put the responsibility of catching them on the caller method. This way exception handling has been passed further up the call chain.

   ```
   public class ShowFile {
     public static void main(String[] args) {
       File propFile = new File("");
       try{
         readFile(propFile);
       } catch (FileNotFoundException e){
         e.printStackTrace();
       } catch (IOException e){
         e.printStackTrace();
       }    
     }
       
     private static void readFile(File fileName) throws FileNotFoundException, 
         IOException, IllegalArgumentException{
       if(fileName == null || fileName.getPath().equals("")){
         throw new IllegalArgumentException("File Name not given");
       }                 
       InputStream in = new FileInputStream(fileName);
       in.read();                
     }
   }
   ```

4. Do not suppress exceptions

   \- The whole idea of having a checked exception in Java is to give a chance to take some action to recover from the raised exception. So ignoring it by having empty braces in the catch block or just printing the stack trace or logging a checked exception and continue with the code is not a best practice.

   ```
   try {
     /* ... */
   } catch( Exception e ) {
   
   }
    
   ```

   ```
   try {
     /* ... */
   } catch( Exception e ) {
     // The exception thrown is lost
     Logger.info( "some exception occured" ); 
   }
   ```

   We should always avoid empty [catch block](https://www.netjstech.com/2015/05/multiple-catch-blocks-in-java-exception-handling.html) that just consumes the [exception](https://www.netjstech.com/2015/05/overview-of-java-exception-handling.html) and doesn't provide any meaningful details of exception for debugging purposes.

5. Don't lose the original exception 

   \- Almost all the exception classes provide a

    

   constructor

    

   with the cause as parameter.

   ```
    public Exception(String message, Throwable cause)
   ```

   Always use the constructor with cause parameter to keep the original exception.
   **As example-** If some exception is thrown and you want to wrap it in a [custom exception](https://www.netjstech.com/2015/05/how-to-create-custom-exception-class-java.html)-

   ```
   catch (IllegalArgumentException exp) {
      throw new MyCustomException("Exception caught: " + e.getMessage());  //Incorrect way
   }
   ```

   This destroys the stack trace of the original exception, and is always wrong. The correct way of doing this is:

   ```
   catch (IllegalArgumentException exp) {
      throw new MyCustomException ("Exception caught: " , exp);  //Correct way
   }
   ```

6. Custom Exception as checked Exception or Unchecked Exception

    

   \- If user can take some action to recover from the expected error then make the custom exception a checked exception. On the other hand if user cannot do anything useful in case of error then make the

    

   custom exception

    

   as an unchecked exception (i.e. inheriting from RunTimeException).

   Most of the time only function of the custom exceptions is to log an error; in that case it should definitely be an unchecked exception.

7. Follow Naming Convention for exceptions

   \- When creating your own custom exception follow the already established naming convention for exception classes like always end the class name with Exception like NoSuchMethodExcpetion, MyAppException etc.

   Also try to keep the same type of exceptions in the same hierarchy like IOExcpetion is the base class exception for all the IO related exceptions.

8. Java exception performance consideration

   \-

    

   Exceptions

    

   do have an impact on the overall performance of the application so use them judiciously. Don't just

    

   throw

    

   and

    

   catch exceptions

   , if you can use boolean variable to indicate whether an operation was successful or not they try to return a Boolean variable to a caller program and decide the flow of the code based on the returned Boolean.

   Avoid unnecessary Exception handling by fixing root cause. Also check for null yourself before performing any operation or check the array length yourself rather than relying on ArrayIndexOutOfBoundException.

9. **Document the Exceptions Thrown**- Use javadoc @throws to clearly specify the exceptions thrown by the method, it's very helpful when you are providing an interface to other applications to use.

10. Exceptions should not be used for flow control

     

    \- Raising and handling exception is an expensive operation and should be used for exceptional conditions only. Using it for flow control hits the overall performance and is a strict no no.

    Always check for array length and null values rather than relying on exception handling to do that

    .

    **As example**, always check for the array length in a loop

    ```
    for (int i=0; i < tempArray.length; i++) {
      // code using the tempArray
    }
    ```

    Don't rely on try-catch block

    ```
    try {
      for (int i=0; ; i++) {
        // code using the tempArray
      }
    } catch (ArrayIndexOutOfBoundsException ex) {
       …..
    }
    ```

    **Check for null with in a code**-

    ```
    if(obj != null){
    	obj.getValue();
    }
    ```

    Or

    ```
    if(someMap.contains(obj)){
        someMap.get(obj);
    } 
    ```

    Is a better solution than having a try-catch block to do catch a null pointer exception.

    ```
    try{
        Obj.getValue()   
    }catch(RuntimeException exp){
       ....
    }
    ```

11. Clean up the resources

    \- If you are using resources like database connections, network connections or IO streams, make sure you clean them up using

     

    finally block

     

    or using the

     

    Java 7 ARM feature

    .

    

12. Logging Exceptions and providing helpful message

     

    \- We should always

     

    log exception messages

     

    and while throwing exception

     

    provide clear message

     

    so that caller will know easily why the exception occurred.

     

    Always append values in the exception message if possible.

    **As example**- If we are writing some code where some specific logic is there for senior citizens (Age greater than or equal to 60) and the method throws **InvalidAgeException** if age is less than 60.

    In that case if exception is thrown then displaying a message "**Exception thrown from method METHOD_NAME**" or "**Invalid parameter**" will not be as helpful as saying **"Invalid age exception for " + {age}**

    That way whoever is seeing the log can easily figure out what was the parameter given when the exception was thrown.

13. Log and throw anti-pattern

    \- Logging and throwing the exception with in a catch block is considered an

     

    error handling anti-pattern

     

    and should be avoided.

    ```
    catch (NoSuchFieldException exp) {
       LOG.error("Exception occurred", exp);
       throw exp;
    }
    ```

    Consider the above example code, doing both logging and throwing will result in

     

    multiple log messages in log files

    , for a single problem in the code. Whoever is going through the log will

     

    have to go through multiple logs for the same error rather than at one place in a log

    .

    

14. Preserve loose coupling

    \- One of the best practices for the exception handling is to preserve loose coupling. According to that an implementation specific checked exception should not propagate to another layer.

     

    As Example

     

    SQL exception from the DataAccessCode (DAO layer) should not propagate to the service (Business) layer. The general practice in that case is to convert database specific SQLException into an unchecked exception and throw it.

    ```
    catch(SQLException ex){
        throw new RuntimeException("DB error", ex);
    }
    ```

That's all for this topic **Best Practices for Exception Handling in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!