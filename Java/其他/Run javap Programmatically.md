This post is about running javap command programmatically from a Java program. In Java it can be done using **ProcessBuilder** class which is used to create operating system processes.

### When do we need javap command

If you have a **.class** file or a jar with .class files and you want to see the structure of a .class file **javap** is a good option to do that.

The **javap** command **disassembles** one or more class files. It comes with JDK under **tools.jar** and used to get the *mnemonical representation of the .class file*.

### ProcessBuilder class

ProcessBuilder has a [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) that takes the command as a list

- ProcessBuilder(List<String> command) - Constructs a process builder with the specified operating system program and arguments.

You can use this constructor to construct a process with **javap** command, **-c argument** and **path of the class file**. Also make sure that you have **tools.jar** (which comes with JDK) in your classpath.

### Java Program to run javap

Suppose you have a class called **First.java**

```
public class First {

 public static void main(String[] args) {
  First first = new First();
  first.showDetails();
 }
 
 public void showDetails(){
  System.out.println("In Show details" );
 }
}
```

You want to run javap command for the compiled .class file of First.java.

So, the command you want to run is :
**javap –c First.class**. Let’s see how you can run this command from a Java program to disassemble a .class file at run time.

```
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class ClassStructure {

 public static void main(String[] args) {
  try {
   List<String> cmdList = new ArrayList<String>();
   cmdList.add("C:\\Program Files\\Java\\jdk1.8.0_111\\bin\\javap.exe");
   cmdList.add("-c");
   cmdList.add("D:\\First.class");
   
   // Constructing ProcessBuilder with List as argument
   ProcessBuilder pb = new ProcessBuilder(cmdList);
   
   Process p = pb.start();
   p.waitFor();
   InputStream fis = p.getInputStream();
   
   DisplayClassStructure(fis);
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (IOException e1) {
   // TODO Auto-generated catch block
   e1.printStackTrace();
  }
 }
 
 // Method used for displaying the disassembled class
 private static void DisplayClassStructure(InputStream is){
  
  InputStream stream;
  
  try {
   
   BufferedReader reader = new BufferedReader(new InputStreamReader(is));
   String line;   
   while ((line = reader.readLine()) != null) {   
        System.out.println(line);   
   }
   // Better put it in finally
   reader.close();
  } catch (FileNotFoundException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  catch (IOException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}
```

**Output**

```
Compiled from "First.java"
public class org.test.First {
  public org.test.First();
    Code:
       0: aload_0
       1: invokespecial #8                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #1                  // class org/test/First
       3: dup
       4: invokespecial #16                 // Method "<init>":()V
       7: astore_1
       8: aload_1
       9: invokevirtual #17                 // Method showDetails:()V
      12: return

  public void showDetails();
    Code:
       0: getstatic     #23                 // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #29                 // String In Show details
       5: invokevirtual #31                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```

That's all for this topic **How to Run javap Programmatically From Java Program**. If you have any doubt or any suggestions to make please drop a comment. Thanks!