### Read or List All Files in a Folder in Java

In this post we'll see how to read or list all the files in a directory using Java.

Suppose you have folder with files in it and there are sub-folders with files in those sub-folders and you want to read or list all the files in the folder recursively.

Here is a folder structure used in this post to read the files. Test, Test1 and Test2 are directories here and then you have files with in those directories.

```
Test
  abc.txt
  Test1
    test.txt
    test1.txt
  Test2
    xyz.txt
```

### Java Example to read all the files in a folder recursively

There are two ways to list all the files in a folder; one is using the **listFiles() method** of the **File** class which is there in Java from 1.2.

Another way to list all the files in a folder is to use **Files.walk()** method which is a recent addition in [Java 8](https://www.netjstech.com/2015/05/interface-static-method-in-java-8.html).

```
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;
import java.util.stream.Stream;

public class ListFiles {
  public static void main(String[] args) {
    File folder = new File("G:\\Test");
    ListFiles listFiles = new ListFiles();
    System.out.println("reading files before Java8 - Using listFiles() method");
    listFiles.listAllFiles(folder);
    System.out.println("-------------------------------------------------");
    System.out.println("reading files Java8 - Using Files.walk() method");
    listFiles.listAllFiles("G:\\Test");

  }
  // Uses listFiles method  
  public void listAllFiles(File folder){
    System.out.println("In listAllfiles(File) method");
    File[] fileNames = folder.listFiles();
    for(File file : fileNames){
      // if directory call the same method again
      if(file.isDirectory()){
         listAllFiles(file);
      }else{
        try {
          readContent(file);
        } catch (IOException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      }
    }
  }
  // Uses Files.walk method   
  public void listAllFiles(String path){
    System.out.println("In listAllfiles(String path) method");
    try(Stream<Path> paths = Files.walk(Paths.get(path))) {
      paths.forEach(filePath -> {
        if (Files.isRegularFile(filePath)) {
          try {
            readContent(filePath);
          } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
        }
      });
    } catch (IOException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
   } 
  }
     
  public void readContent(File file) throws IOException{
    System.out.println("read file " + file.getCanonicalPath() );
    try(BufferedReader br  = new BufferedReader(new FileReader(file))){
      String strLine;
      // Read lines from the file, returns null when end of stream 
      // is reached
      while((strLine = br.readLine()) != null){
      System.out.println("Line is - " + strLine);
      }
    }
  }
     
  public void readContent(Path filePath) throws IOException{
    System.out.println("read file " + filePath);
    List<String> fileList = Files.readAllLines(filePath);
    System.out.println("" + fileList);
  }   
}
```

**Output**

```
reading files before Java8 - Using listFiles() method
In listAllfiles(File) method
read file G:\Test\abc.txt
Line is - This file is in Test folder.
In listAllfiles(File) method
read file G:\Test\Test1\test.txt
Line is - This file test is under Test1 folder.
read file G:\Test\Test1\test1.txt
Line is - This file test1 is under Test1 folder.
In listAllfiles(File) method
read file G:\Test\Test2\xyz.txt
Line is - This file xyz is under Test2 folder.
-------------------------------------------------
reading files Java8 - Using Files.walk() method
In listAllfiles(String path) method
read file G:\Test\abc.txt
[This file is in Test folder.]
read file G:\Test\Test1\test.txt
[This file test is under Test1 folder.]
read file G:\Test\Test1\test1.txt
[This file test1 is under Test1 folder.]
read file G:\Test\Test2\xyz.txt
[This file xyz is under Test2 folder.]
```

Here we have two overloaded **listAllFiles()** methods. First one takes File instance as argument and use that instance to read files using the **File.listFiles()** method. In that method while going through the list of files under a folder you check if the next element of the list is a file or a folder. If it is a folder then you recursively call the listAllFiles() method with that folder name. If it is a file you call the **readContent()** method to [read the file using BufferedReader](https://www.netjstech.com/2016/05/how-to-read-file-java-bufferedreader.html).

Another version of **listAllFiles()** method takes [String](https://www.netjstech.com/2016/07/string-in-java.html) as argument. In this method whole folder tree is traversed using the **Files.walk()** method. Here again you verify if it is a regular file then you call the **readContent()** method to read the file.

Note that **readContent()** method is also overloaded one takes File instance as argument and another Path instance as argument.

If you just want to list the files and sub-directories with in the directory then you can comment the readContent() method.