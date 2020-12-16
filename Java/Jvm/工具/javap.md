javap是JDK提供的一个命令行工具,javap能对给定的class文件提供的字节代码进行反编译。

通过它，可以对照源代码和字节码，从而了解很多编译器内部的工作,对更深入地理解如何提高程序执行的效率等问题有极大的帮助。

使用格式

```
javap <options> <classes>
```

常用: javap -c -v 类名



```
  -help  --help  -?        输出此用法消息
  -version                 版本信息
  -v  -verbose             输出附加信息
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类
                           和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的
                           系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示最终常量
  -classpath <path>        指定查找用户类文件的位置
  -cp <path>               指定查找用户类文件的位置
  -bootclasspath <path>    覆盖引导类文件的位置
```

新建Hello.java



```
public class Hello {
    private Integer aa = 1;
    public String ss = "sss";
    public static void main(String[] args) {   
    System.out.println("Hello Java");
    }
}
```



 不带参数

javap Hello

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)

```
Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;
  public Hello();
  public static void main(java.lang.String[]);
}
```

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

 c `对代码进行反汇编`

 javap -c Hello

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

```
Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;

  public Hello();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":
()V
       4: aload_0
       5: iconst_1
       6: invokestatic  #2                  // Method java/lang/Integer.valueOf:
(I)Ljava/lang/Integer;
       9: putfield      #3                  // Field aa:Ljava/lang/Integer;
      12: aload_0
      13: ldc           #4                  // String sss
      15: putfield      #5                  // Field ss:Ljava/lang/String;
      18: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #6                  // Field java/lang/System.out:Ljava/
io/PrintStream;
       3: ldc           #7                  // String Hello Java
       5: invokevirtual #8                  // Method java/io/PrintStream.printl
n:(Ljava/lang/String;)V
       8: return
}
```

```
输出附加信息
javap -v Hello
```

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

```
Classfile /D:/WWW/11/javap/Hello.class
  Last modified 2019-4-23; size 602 bytes
  MD5 checksum 9eb7401f16043460fa36db8275c0f7c4
  Compiled from "Hello.java"
public class Hello
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #10.#23        // java/lang/Object."<init>":()V
   #2 = Methodref          #24.#25        // java/lang/Integer.valueOf:(I)Ljava/
lang/Integer;
   #3 = Fieldref           #9.#26         // Hello.aa:Ljava/lang/Integer;
   #4 = String             #27            // sss
   #5 = Fieldref           #9.#28         // Hello.ss:Ljava/lang/String;
   #6 = Fieldref           #29.#30        // java/lang/System.out:Ljava/io/Print
Stream;
   #7 = String             #31            // Hello Java
   #8 = Methodref          #32.#33        // java/io/PrintStream.println:(Ljava/
lang/String;)V
   #9 = Class              #34            // Hello
  #10 = Class              #35            // java/lang/Object
  #11 = Utf8               aa
  #12 = Utf8               Ljava/lang/Integer;
  #13 = Utf8               ss
  #14 = Utf8               Ljava/lang/String;
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               main
  #20 = Utf8               ([Ljava/lang/String;)V
  #21 = Utf8               SourceFile
  #22 = Utf8               Hello.java
  #23 = NameAndType        #15:#16        // "<init>":()V
  #24 = Class              #36            // java/lang/Integer
  #25 = NameAndType        #37:#38        // valueOf:(I)Ljava/lang/Integer;
  #26 = NameAndType        #11:#12        // aa:Ljava/lang/Integer;
  #27 = Utf8               sss
  #28 = NameAndType        #13:#14        // ss:Ljava/lang/String;
  #29 = Class              #39            // java/lang/System
  #30 = NameAndType        #40:#41        // out:Ljava/io/PrintStream;
  #31 = Utf8               Hello Java
  #32 = Class              #42            // java/io/PrintStream
  #33 = NameAndType        #43:#44        // println:(Ljava/lang/String;)V
  #34 = Utf8               Hello
  #35 = Utf8               java/lang/Object
  #36 = Utf8               java/lang/Integer
  #37 = Utf8               valueOf
  #38 = Utf8               (I)Ljava/lang/Integer;
  #39 = Utf8               java/lang/System
  #40 = Utf8               out
  #41 = Utf8               Ljava/io/PrintStream;
  #42 = Utf8               java/io/PrintStream
  #43 = Utf8               println
  #44 = Utf8               (Ljava/lang/String;)V
{
  public java.lang.String ss;
    descriptor: Ljava/lang/String;
    flags: ACC_PUBLIC

  public Hello();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>
":()V
         4: aload_0
         5: iconst_1
         6: invokestatic  #2                  // Method java/lang/Integer.valueO
f:(I)Ljava/lang/Integer;
         9: putfield      #3                  // Field aa:Ljava/lang/Integer;
        12: aload_0
        13: ldc           #4                  // String sss
        15: putfield      #5                  // Field ss:Ljava/lang/String;
        18: return
      LineNumberTable:
        line 1: 0
        line 2: 4
        line 3: 12

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #6                  // Field java/lang/System.out:Ljav
a/io/PrintStream;
         3: ldc           #7                  // String Hello Java
         5: invokevirtual #8                  // Method java/io/PrintStream.prin
tln:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 8
}
SourceFile: "Hello.java"
```

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

```
输出行号和本地变量表
javap -l Hello
```

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

```
Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;

  public Hello();
    LineNumberTable:
      line 1: 0
      line 2: 4
      line 3: 12

  public static void main(java.lang.String[]);
    LineNumberTable:
      line 5: 0
      line 6: 8
}
```

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

javap -p Hello

[![复制代码](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/14/21:44:43-copycode-20201214214442068.gif)](javascript:void(0);)

```
Compiled from "Hello.java"
public class Hello {
  private java.lang.Integer aa;
  public java.lang.String ss;
  public Hello();
  public static void main(java.lang.String[]);
}
```



```
Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;
  public Hello();
  public static void main(java.lang.String[]);
}
```

javap -private Hello

```
Compiled from "Hello.java"
public class Hello {
  private java.lang.Integer aa;
  public java.lang.String ss;
  public Hello();
  public static void main(java.lang.String[]);
}
```



```
输出内部类型签名
javap -s Hello
```



```
Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;
    descriptor: Ljava/lang/String;
  public Hello();
    descriptor: ()V

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
}
```



```
显示正在处理的类的
javap -sysinfo  Hello
```

```
Classfile /D:/WWW/11/javap/Hello.class
  Last modified 2019-4-23; size 602 bytes
  MD5 checksum 9eb7401f16043460fa36db8275c0f7c4
  Compiled from "Hello.java"
public class Hello {
  public java.lang.String ss;
  public Hello();
  public static void main(java.lang.String[]);
}
```

 