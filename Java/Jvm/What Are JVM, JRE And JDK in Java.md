### What Are JVM, JRE And JDK in Java

This post gives a brief explanation of JVM, JRE and JDK in Java. Before going into that explanation you should also know what is bytecode in Java.

### What is bytecode in Java

When a Java program is compiled it is not directly compiled into machine language but into an intermediate code known as bytecode. Bytecode is platform independent and it still needs to be interpreted and executed by the JVM installed on the specific platform.

**For example** let's say you have a Java file called "Test.java". When you compile this file you get a file called "Test.class" which is the bytecode for your Java file. JVM interprets and executes this Test.class file.

### JVM

JVM meaning Java Virtual Machine is an abstract layer between a Java program and the platform that Java Program is running on. JVM is **platform dependent** and different implementations of JVMs are available for specific platforms.

A Java program can run on a specific platform only when-

- JVM has been implemented for a platform.
- JVM has been installed on a platform.

The JVM doesn't understand Java program as we write it, it understands the ".class" file which we get by compiling the .java file. This ".class" file contains the bytecode understandable by the JVM. It is because of JVM that Java is called a "portable language" (write once, run anywhere)

- Refer [JVM Run-Time Data Areas - Java Memory Allocation](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html) for better understanding of JVM.

Following figure shows the abstraction provided by JVM by sitting in between the bytecode and the specific platform.

[![Bytecode interpretation by JVM](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:54:27-JVM%252Babstraction.png)](https://4.bp.blogspot.com/-OSh8xx7-LBQ/XBnUqYc2wyI/AAAAAAAABD4/ZgBzwAIqJTk4x9Wo5oFXzqZVGX5Z7ty5ACLcBGAs/s1600/JVM%2Babstraction.png)

### JRE

JRE meaning **Java Runtime Environment** provides the libraries, the Java Virtual Machine, and other components to run applets and applications written in the Java programming language.

The compiled bytecode doesn't run on CPU directly, JVM sits in between and interpret the bytecode into readable machine language for the CPU. It is actually the JRE that enables Java bytecode to run on any platform. Bytecodes, which are interpreted by the JVM, simply call classes found in the JRE when they need to perform actions they cannot do by themselves

[![JVM and JRE in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:54:27-JVM%252Band%252BJRE%252Bin%252BJava.png)](https://3.bp.blogspot.com/-OipDX9FCjS0/XBnXjp1zGpI/AAAAAAAABEQ/KvgafCl0ev01QjvUCqMzJ7vPGQBLTR4lgCLcBGAs/s1600/JVM%2Band%2BJRE%2Bin%2BJava.png)

### JDK

JDK meaning **Java Development Kit** is a superset of the JRE, and contains everything that is in the JRE, plus development tools such as the compilers and debuggers necessary for developing applets and applications.

That's all for this topic **What Are JVM, JRE And JDK in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!

