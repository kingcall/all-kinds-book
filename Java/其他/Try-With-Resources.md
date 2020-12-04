### Try-With-Resources in Java With Examples

Java 7 introduced a new form of try statement known as try-with-resources in Java for **Automatic Resource Management (ARM)**. Here resource is an object that must be closed after the program is finished with it. **Example of resources** would be an opened file handle or database connection etc.

Note that any resource declared in a **try-with-resource** statement, will be closed regardless of whether the try statement completes normally or abruptly.

Apart from try-with-resources, Java 7 also introduced [Multi catch statement](https://www.netjstech.com/2015/05/multi-catch-statement-in-java-7.html) to decrease code verbosity.

**Table of contents**

1. [General Form of try-with-resources in Java](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcessyntax)
2. [Example of try-with-resources in Java](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcesexample)
3. [Try-with-resources - Java 9 enhancement](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcesjava9)
4. [Java 9 enhancement example](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcesjava9exp)
5. [How resources are closed automatically](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcesautomatic)
6. [Using Multiple Resources with try-with-resources in Java](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcesmultiple)
7. [Suppressed Exceptions with try-with-resource](https://www.netjstech.com/2015/05/try-with-resources-java7.html#trywithresourcessuppress)
8. [Custom implementation of AutoCloseable](https://www.netjstech.com/2015/05/try-with-resources-java7.html#AutoCloseable)



### General Form of try-with-resources in Java

try-with-resources statement is a try statement that declares one or more resources with in parenthesis.

```
try(resources declared here){
  // code that uses resources
} catch (ExceptionObject eObj) {
  // exception handling code
}
```

When try-with-resources is used in Java for exception handling, [finally block](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html) **is not required** as resources are closed automatically as soon as [try-catch block](https://www.netjstech.com/2015/05/java-exception-handling-try-catch-block.html) is executed.

### Example of try-with-resources in Java

First let's see how it was done without try-with-resources. Before the introduction of try-with-resources we had to explicitly close the resources once the try block completed normally or abruptly. Usually, finally block of a try-catch statement was used for it.That's how it looked like-

```
public class ResourceMgmtDemo {
  public static void main(String[] args) {
    BufferedReader br = null;
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
  }
}
```

Note how **finally block** is used here to close the file handle. We had to write one whole block just for closing the resources and there again wrap the code in try-catch block.

**try-with-resources** helps in reducing such boiler plate code. Let's see the same example using try-with-resources in Java

```
public class ResourceMgmtDemo {
  public static void main(String[] args) {  
    try(BufferedReader br = new BufferedReader(new FileReader("C:\\test.txt"))) {            
      String strLine;
      while((strLine = br.readLine()) != null){
        System.out.println("Line is - " + strLine);
      }
    } catch (IOException e) {
      e.printStackTrace();
    } 
  }
}
```

It can be seen how resource is declared with the try statement itself here. Also notice the reduction in the number of lines of code by using try-with-resources instead of a finally block!

### Try-with-resources - Java 9 enhancement

Before Java 9 the resource which has to be closed automatically using try-with-resource must be created with in the try statement. Java 9 onward you can create the resource outside and use the reference with in the try statement. Only condition is that the referenced variable with in the try-with-resource construct must be [effectively final](https://www.netjstech.com/2016/03/effectively-final-in-java-8.html).

### Using try-with-resources - Java 9 enhancement example

```
public class ResourceMgmtDemo {
 public static void main(String[] args) {  
  try {
   readFile("C:\\test.txt");
  } catch (IOException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 public static void readFile(String filePath) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(filePath));
  try(br) {  
   System.out.println(br.readLine());
  }
 }
}
```

As you can see that the BufferedReader instance is created outside now and reference is used with the try block.

### try-with-resource - How resources are closed automatically

Any resource that is used with try-with-resource is closed automatically because of the [interface](https://www.netjstech.com/2015/05/interface-in-java.html) **java.lang.AutoCloseable**. AutoCloseable interface in Java has a **close method** and any resource used with try-with-resources should implement this interface and provide implementation for the close method.

It should be noted that the close method of **AutoCloseable** interface [throws exceptions](https://www.netjstech.com/2015/05/throws-keyword-in-java-exception-handling.html) of type Exception. Consequently, subclasses of the AutoCloseable interface can override this behavior of the close method to throw specialized exceptions. Like in Java 7 **Closeable interface** extends AutoCloseable and override the behavior of close method to throw **IOException**.
In the above example BufferedReader class is used which implements the close method of Closeable interface and throw IOException.

### Using Multiple Resources with try-with-resources in Java

It is possible to use multiple resources with Java try-with-resources statement, all of them will be closed automatically.

```
public class ResourceMgmtDemo {
  public static void main(String[] args) {  
    try(FileReader fr = new FileReader("C:\\test.txt");
      BufferedReader br = new BufferedReader(fr)) {            
        System.out.println(br.readLine());
    } catch (IOException e) {
        e.printStackTrace();
    } 
  }
}
```

This example creates two resources inside the parentheses after the try keyword. **A FileReader and a BufferedReader**. Note that the created resources should be separated by a **semicolon**. Both of these resources will be closed automatically when execution leaves the try block.
The resources will be **closed in reverse order** of the order in which they are listed inside the try parentheses. First BufferedReader and then the FileReader will be closed.

### Suppressed Exceptions with try-with-resource

If an exception is thrown from the try block and one or more exceptions are thrown from the try-with-resources statement, then the exceptions thrown from the try-with-resources statement are suppressed. The exception thrown by the method is the one that is thrown in try block. You can retrieve these suppressed exceptions by calling the Throwable.getSuppressed method from the exception thrown by the try block.

This behavior is different from what will happen in case of try-catch-finally statement. If exceptions are thrown in both try block and finally block, the method returns the exception thrown in finally block.

Let's clarify the difference, with help of **example code**

```
public class ResourceMgmtDemo {
  public static void main(String[] args) {  
    try {
      normalTry();
    } catch (Exception e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    try {
      tryWithResource();
    } catch (Exception e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
 
  private static void normalTry() throws Exception {
    MyAutoResource myAutoResource = null;
    try {
      myAutoResource = new MyAutoResource();
      System.out.println("MyAutoResource created in try block");            
      throw new Exception("Exception in try block");
    } finally {
        if (myAutoResource != null)
          myAutoResource.close();
    }
  }
    
  private static void tryWithResource() throws Exception {
    try (MyAutoResource myAutoResource = new MyAutoResource()) {
      System.out.println("MyAutoResource created in try-with-resources");
      throw new Exception("Exception in try-with-resources block");
    }
  }
}
// Custom implementation of AutoCoseable
class MyAutoResource implements AutoCloseable { 
  @Override
  public void close() throws Exception {
    System.out.println("Closing MyAutoResource");
    // Here exception is thrown
    throw new Exception("Exception in Closing");
  }
}
```

**Output**

```
MyAutoResource created in try block
Closing MyAutoResource
java.lang.Exception: Exception in Closing
 at org.netjs.examples.impl.MyAutoResource.close(ResourceMgmtDemo.java:51)
 at org.netjs.examples.impl.ResourceMgmtDemo.normalTry(ResourceMgmtDemo.java:31)
 at org.netjs.examples.impl.ResourceMgmtDemo.main(ResourceMgmtDemo.java:10)
MyAutoResource created in try-with-resources
Closing MyAutoResource
java.lang.Exception: Exception in try-with-resources block
 at org.netjs.examples.impl.ResourceMgmtDemo.tryWithResource(ResourceMgmtDemo.java:39)
 at org.netjs.examples.impl.ResourceMgmtDemo.main(ResourceMgmtDemo.java:16)
 Suppressed: java.lang.Exception: Exception in Closing
  at org.netjs.examples.impl.MyAutoResource.close(ResourceMgmtDemo.java:51)
  at org.netjs.examples.impl.ResourceMgmtDemo.tryWithResource(ResourceMgmtDemo.java:40)
  ... 1 more
```

In the code we have a *custom implementation of AutoCloseable* called **MyAutoResource** with its own implementation of the **close() method**. Notice that an exception is thrown in the close method.

We have a method **normalTry()** with usual try-catch-finally block. In the try block of the method **normalTry()** exception is thrown. In the finally block there is a method call **myAutoResource.close()** which will again throw the exception in the close method of the MyAutoResource class. If you see the output, the exception thrown in the finally block is the returned exception.

Now in the **tryWithResource()** method an exception is thrown because of which try-with-resources will try to close the resource and call the close method of the **MyAutoResource** class. In the close method again an exception is thrown. If you have seen the output exception that is thrown from the try block is shown and the exception thrown from try-with-resources statement is shown as Supressed exception.

### Custom implementation of AutoCloseable

In the above example a custom implementation of AutoCloseable is already used. Your custom class needs to implement the AutoCloseable interface and provide implementation of the close method.

**Points to note-**

- try-with-resources in Java helps in reducing the boiler plate code by providing automatic resource management.
- With try-with-resources there is no need of having a finally block just to close the resources.
- Any resource that is used with try-with-resource is closed automatically because of the interface java.lang.AutoCloseable.
- Multiple resources can be opened in try-with-resources statement separated by a semicolon.
- try-with-resources along with another feature introduced in Java 7, multi catch statement helps in reducing the number of lines in the code and increasing readability.

That's all for this topic **Try-With-Resources in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!

