### Generating Getters And Setters Using Reflection in Java

When you right click on any Java Bean class name with in the [eclipse](https://www.netjstech.com/2017/07/how-to-pass-command-line-arguments-in-eclipse.html) IDE and click on **Source – Generate Getters and Setters** you get the getter and setter methods for the selected fields. Ever wondered what goes in the background to **generate the getter and setter methods?**

Yes, it is the magic of [reflection in Java](https://www.netjstech.com/2017/07/reflection-in-java.html) which gets the information about the fields of the class and their types and then generate the getters and setters using reflection accordingly.

If you have to create such a functionality yourself then using the reflection API you can create your own getters and setters generator class, of course just for academic purpose, as all the IDEs anyway provide the facility to do that.

### Getters & Setters generator using reflection example

Let’s say you have a class TestClass with three fields of type int, String and Boolean and you want to generate getters and setters for these 3 fields using Java reflection API.

**TestClass**

```
public class TestClass {
 private int value;
 private String name;
 private boolean flag;
} 
```

**GetterSetterGenerator class**

Using the [reflection API for the field](https://www.netjstech.com/2017/07/reflection-in-java-field.html) you can get information about the fields of the given class – like name and type. Once you have that information you can create set and get methods for the fields. In this code set and get methods are just printed after creating them as it is just for illustrating the usage of reflection in Java so not going into File I/O.

```
import java.lang.reflect.Field;
import java.util.Arrays;

public class GetterSetterGenerator {

 public static void main(String[] args) {
  try {
   GetterSetterGenerator gt = new GetterSetterGenerator();
   StringBuffer sb = new StringBuffer();
   
   Class<?> c = Class.forName("org.prgm.TestClass");
   // Getting fields of the class
   Field[] fields = c.getDeclaredFields();
   System.out.println("Fields - " + Arrays.toString(fields));
   for(Field f : fields){
    String fieldName = f.getName();
    String fieldType = f.getType().getSimpleName();
    
    System.out.println("Field Name -- " + fieldName);
    System.out.println("Field Type " + fieldType);
    
    gt.createSetter(fieldName, fieldType, sb);    
    gt.createGetter(fieldName, fieldType, sb);
   }
   System.out.println("" + sb.toString());
   
  }catch (ClassNotFoundException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }  
 }

 private void createSetter(String fieldName, String fieldType, StringBuffer setter){
  setter.append("public void").append(" set");
  setter.append(getFieldName(fieldName));
  setter.append("(" + fieldType + " " + fieldName + ") {");
  setter.append("\n\t this."+ fieldName + " = " + fieldName + ";");
  setter.append("\n" + "}" + "\n");
 }
 
 private void createGetter(String fieldName, String fieldType, StringBuffer getter){
  // for boolean field method starts with "is" otherwise with "get"
  getter.append("public " + fieldType).append((fieldType.equals("boolean")?" 
    is" : " get") + getFieldName(fieldName) + "(){");
  getter.append("\n\treturn " + fieldName + ";");
  getter.append("\n" + "}" + "\n");
 }
 
 private String getFieldName(String fieldName){
  return fieldName.substring(0, 1).toUpperCase() + fieldName.substring(
    1, fieldName.length());
 }
}
```

**Output**

```
Fields - [private int org.prgm.TestClass.value, private java.lang.String org.prgm.TestClass.name, private boolean org.prgm.TestClass.flag]
Field Name -- value
Field Type int
Field Name -- name
Field Type String
Field Name -- flag
Field Type Boolean

public void setValue(int value) {
  this.value = value;
}
public int getValue(){
 return value;
}
public void setName(String name) {
  this.name = name;
}
public String getName(){
 return name;
}
public void setFlag(boolean flag) {
  this.flag = flag;
}
public boolean isFlag(){
 return flag;
}
```

That's all for this topic **Generating Getters And Setters Using Reflection in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!