### How to Run a Shell Script From Java Program

This post talks about how you can execute a shell script from a Java program.

If you have a shell script say test.sh then you can run it from a Java program using RunTime class or ProcessBuilder (Note ProcessBuilder is added in Java 5).

**Shell script**

```
echo 'starting script'
mkdir test
cd test
touch SAMPLE
```

### Using Runtime.getRunTime().exec to execute shell script

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class RunningSS {
 public static void main(String[] args) {
  Process p;
  try {
   String[] cmd = { "sh", "/home/adb/Documents/test.sh"};
   p = Runtime.getRuntime().exec(cmd); 
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

After executing this Java program with the given shell script, if you verify at the location where your Java program is you will see a directory test created and a file SAMPLE with in that directory.

**Runtime.getRuntime().exec** method is used to run the command.

- **public static Runtime getRuntime()** - Returns the runtime object associated with the current Java application.

- **public Process exec(String command) throws IOException** - Executes the specified string command in a separate process.

**cmd /c** which is used with the command has the following explanantion -

- **cmd**- Starts a new command shell
- **/c**- Executes the given command and terminates

Execution of the command returns instance of class Process. Using the **getInputStream()** method of Process class output of the executed command can be printed by reading the stream.

### Using ProcessBuilder to execute shell script in Java

If you have to run the same command as above using ProcessBuilder, which is a much clearer way to do that, you can create a list with the command and the required arguments and then pass it to ProcessBuilder instance as command.

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class RunningSS {
  public static void main(String[] args) {
    Process p;
    try {        
      List<String> cmdList = new ArrayList<String>();
      // adding command and args to the list
      cmdList.add("sh");
      cmdList.add("/home/adb/Documents/test.sh");
      ProcessBuilder pb = new ProcessBuilder(cmdList);
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

