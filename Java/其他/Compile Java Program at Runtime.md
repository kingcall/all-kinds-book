### How to Compile Java Program at Runtime

This post talks about how you can compile a [java program](https://www.netjstech.com/2015/06/java-programs.html) at runtime. You may have a case where you get a Java file path by [reading a property file](https://www.netjstech.com/2017/09/how-to-read-properties-file-in-java.html) and you need to compile and run that Java file or you may have a scenario where at run time a program file is created rather than some script which you need to compile and run.

In such cases you need to **compile your code at run time** from another Java program. It can be done using **JavaCompiler** [interface](https://www.netjstech.com/2015/05/interface-in-java.html) and **ToolProvider** class. Note that *these classes are provided from Java 6*.

Also note that you will need **JDK** to run it not **JRE**, so you need to have JDK libraries not JRE. If you are using eclipse and your JRE System library is pointing to JRE path make sure it points to JDK. You can do that by right clicking on your project and going to Java Build Path through properties. There click on Libraries tab and select the JRE System Library which points to jre path and click Edit.

[![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/19:59:36-Compiling%252BJava%252Bprogram.png)](https://4.bp.blogspot.com/-09Kp07JT55Q/WA4l8LMor8I/AAAAAAAAATc/22bP_dfb8CEfinwTY4-oqX3-rqyr3CpXgCPcB/s1600/Compiling%2BJava%2Bprogram.png)

In the next dialog box you can select the path to JDK after selecting Alternate JRE.

### Java code to compile Java program at runtime

Suppose there is a Java file HelloWorld.java which you need to compile at run time and execute its method displayMessage.

**HelloWorld.java**

```
public class HelloWorld {

 public static void main(String[] args) {
 

 }
 
 public void displayMessage(){
  System.out.println("Hello world from displayMessage method");
 }

}
```

**RTComp.java**

This is the class where runtime compilation is done.

```
import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;

import javax.tools.JavaCompiler;
import javax.tools.ToolProvider;

public class RTComp {

 public static void main(String[] args) {
  JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
  // Compiling the code
  int result = compiler.run(null, null, null, 
     "C:\\workspace\\Test\\src\\org\\test\\HelloWorld.java");
  System.out.println("result " + result);
  // Giving the path of the class directory where class file is generated..
  File classesDir = new File("C:\\workspace\\Test\\bin\\org\\test");
  // Load and instantiate compiled class.
  URLClassLoader classLoader;
  try {
   // Loading the class 
   classLoader = URLClassLoader.newInstance(new URL[] { classesDir.toURI().toURL() });
   Class<?> cls;
   
   cls = Class.forName("org.test.HelloWorld", true, classLoader);
  
   HelloWorld instance = (HelloWorld)cls.newInstance();
   instance.displayMessage();
  } catch (MalformedURLException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (ClassNotFoundException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (IllegalAccessException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  catch (InstantiationException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }  
 }
}
```

**Output**

```
result 0
Hello world from displayMessage method
```

Here it can be seen that **compiler.run** method is provided with the path of the class **HelloWorld**. Here I have used the package as org.test.

Also in eclipse, by default, bin is the location for putting .class files so that path is provided for the generated class. Once the java file is compiled it is loaded using the class loader and an instance of that class is created. Using that instance method of that class is called at runtime.

That's all for this topic **How to Compile Java Program at Runtime**. If you have any doubt or any suggestions to make please drop a comment. Thanks!