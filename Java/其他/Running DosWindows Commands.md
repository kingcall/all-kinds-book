### Running Dos/Windows Commands From Java Program

If you want to run **DOS or Windows commands** from a Java program it can be done using **RunTime** class or **ProcessBuilder** (Note ProcessBuilder is added in Java 5).

### Java RunTime class

Every Java application has a single instance of class Runtime that allows the application to interface with the environment in which the application is running. The current runtime can be obtained from the getRuntime method.

In RunTime class there is a **exec()** method that executes the specified string command in a separate process. Using this exec() method dos or windows commands can be executed from Java.

**Runtime.getRunTime().exec to run dos/windows commands in Java example**

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class RunningCommand {
  public static void main(String[] args) {
    Process p;
    try {
      p = Runtime.getRuntime().exec("cmd /c dir");

      p.waitFor(); 
      BufferedReader reader=new BufferedReader(new InputStreamReader(
                  p.getInputStream())); 
      String line; 
      while((line = reader.readLine()) != null) { 
        System.out.println(line);
      } 
    } catch (IOException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
}
```

**Output**

```
Volume in drive C is OS
Volume Serial Number is AEC2-FEE9

 Directory of C:\workspace\abc

10/19/2016  12:39 PM    <DIR>          .
10/19/2016  12:39 PM    <DIR>          ..
10/24/2016  03:22 PM               592 .classpath
10/19/2016  12:39 PM               379 .project
10/19/2016  12:39 PM    <DIR>          .settings
10/21/2016  03:16 PM    <DIR>          bin
10/19/2016  12:39 PM    <DIR>          src
               2 File(s)            971 bytes
               5 Dir(s)  40,032,706,560 bytes free
```

Here it can be seen that directory listing is displayed for the directory which happens to be the workspace directory from where I executed the Java program.

**Runtime.getRuntime().exec** method is used to run the command.

- **public static Runtime getRuntime()** - Returns the runtime object associated with the current Java application.

- **public Process exec(String command) throws IOException** - Executes the specified string command in a separate process.

**cmd /c** which is used with the command has the following explanantion -

- **cmd**- Starts a new command shell
- **/c**- Executes the given command and terminates

Execution of the command returns instance of class Process. Using the **getInputStream()** method of Process class output of the executed command can be printed by reading the stream.

### Running command Using ProcessBuilder

You can also use ProcessBuilder class to run dos or windows command from Java. If you have to run the same command as used above using **ProcessBuilder**, which is a much clearer way to do that, you can create a list with the command and the required arguments and then pass it to ProcessBuilder instance as command.

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class RunningCommand {
  public static void main(String[] args) {
    Process p;
    try {
      List<String> cmdList = new ArrayList<String>();
      cmdList.add("cmd");
      cmdList.add("/c");
      cmdList.add("dir");
      ProcessBuilder pb = new ProcessBuilder();
      pb.command(cmdList);
      p = pb.start();
    
      p.waitFor(); 
      BufferedReader reader=new BufferedReader(new InputStreamReader(
                p.getInputStream())); 
      String line; 
      while((line = reader.readLine()) != null) { 
        System.out.println(line);
      } 
    } catch (IOException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
}
```

That's all for this topic **Running Dos/Windows Commands From Java Program**. If you have any doubt or any suggestions to make please drop a comment. Thanks!