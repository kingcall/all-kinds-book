### JVM Run-Time Data Areas - Java Memory Allocation

The Java Virtual Machine (JVM) defines various **run-time data areas** that are used during the execution of the program. Some of these JVM data areas are created per [thread](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) where as others are created on JVM startup and memory area is shared among threads.

The JVM run-time data areas can be divided into six areas as per usage-

- [The program counter (PC) register](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#PCRegister)
- [Java Virtual Machine (JVM) stacks](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#JVMStacks)
- [Native method stacks](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#NativeMethodStacks)
- [Heap Area](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#HeapArea)
- [Method area](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#Methodarea)
- [Run-Time Constant Pool](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html#ConstantPool)

As stated above these memory areas can be categorized into two categories-

- **Created per thread**– PC register, JVM stack, Native method stack
- **Shared by threads**– Heap, Method area, Run-time constant pool

[![JVM run-time data areas](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/20:00:34-JVM%252BData%252BAreas.png)](https://4.bp.blogspot.com/-m1QNXYUo8dg/WriYyHlzeiI/AAAAAAAAAmo/YB1Bzw2a65EvS-hGvS1UczQLNo-dAf_mgCLcBGAs/s1600/JVM%2BData%2BAreas.png)

### The Program Counter (PC) Register

In a [JVM](https://www.netjstech.com/2015/05/what-are-jvm-jre-and-jdk-in-java.html) at any given time many threads may be executing. Each of the executing thread gets its own PC register.

If the method executed by the JVM thread is a JAVA method then the PC register contains the address of the Java Virtual Machine instruction currently being executed. In case, [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) is executing a native method, the value of the Java Virtual Machine's pc register is undefined.

### Java Virtual Machine (JVM) stacks

Each JVM thread has its own JVM stack which is created when the thread starts. JVM stack stores frames which are pushed and popped out of stack, a JVM stack is never manipulated directly.

At the time of any exception it is this stack trace you get where each element represents a single stack frame.

Exceptional conditions associated with Java Virtual Machine stacks:

1. If the computation in a thread requires a larger Java Virtual Machine stack than is permitted, the Java Virtual Machine throws a **StackOverflowError**.
2. If Java Virtual Machine stacks can be dynamically expanded, and expansion is attempted but insufficient memory can be made available to effect the expansion, or if insufficient memory can be made available to create the initial Java Virtual Machine stack for a new thread, the Java Virtual Machine throws an **OutOfMemoryError**.

**Frames in JVM Stacks**

A new frame is created when a method is invoked, this frame is then pushed into the JVM stack for the thread. The frame is destroyed when the its method invocation completes.

Each frame has its own [array](https://www.netjstech.com/2017/02/array-in-java.html) of local variables, its own operand stack and a reference to the run-time constant pool of the class of the current method. The sizes of the local variable array and the operand stack are determined at compile-time and are supplied along with the code for the method associated with the frame.

At any point only one frame is active which is the frame for the executing method. This frame is referred to as the current frame, and its method is known as the current method. The class in which the current method is defined is the current class.

Note that a frame created by a thread is local to that thread and cannot be referenced by any other thread.

- Local Variables

  \- Each frame that is created and added to the JVM stack contains an array of variables known as its local variables.

  The length of the local variable array is determined at compile-time itself and supplied in the binary representation of a class or interface along with the code for the method associated with the frame.

  The JVM uses local variables to pass parameters when the method is invoked.

  If it is a class method, any parameters are passed in consecutive local variables starting from local variable 0.

  If it is an instance method, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked i.e. this. Any parameters are subsequently passed in consecutive local variables starting from local variable 1.

- Operand Stack

  – Each frame contains a Last In First Out (LIFO) stack called the operand stack of the frame. The maximum depth of the operand stack is known as the compile time itself and supplied along with the code for the method associated with the frame.

  Operand stack is the actual storage place at the time of method execution. When the frame is created for the method its operand stack is empty. The Java Virtual Machine will supply instructions to load constants or values from local variables or fields onto the operand stack. Other JVM instructions take operands from the operand stack, operate on them, and push the result back onto the operand stack.

  The operand stack is also used to prepare parameters to be passed to methods and to receive method results.

- Performing Dynamic Linking

  \- In the compiled

   

  .class

   

  file code for a method refers to methods to be invoked and variables to be accessed via symbolic references.These symbolic method references are translated into concrete method references through dynamic linking, loading classes as necessary to resolve symbols that are undefined at that point.

  Each frame in the JVM stack contains reference to the **run-time constant pool** for the type of the current method to support dynamic linking of the method code.

### Native Method Stacks

A JVM may also use conventional stacks in order to support native methods. **Native methods** are the methods written in a language other than the Java programming language.

Native method stacks are allocated per thread when each thread is created.

The following exceptional conditions are associated with native method stacks:

- If the computation in a thread requires a larger native method stack than is permitted, the Java Virtual Machine throws a **StackOverflowError**.
- If native method stacks can be dynamically expanded and native method stack expansion is attempted but insufficient memory can be made available, or if insufficient memory can be made available to create the initial native method stack for a new thread, the Java Virtual Machine throws an **OutOfMemoryError**.

### Heap Area

Heap is the JVM run-time data area from which memory is allocated to objects, instance variables and arrays. Heap is created on the JVM start-up and *shared among all Java Virtual Machine threads*.

Once the object stored on the heap is not having any reference, memory for that object is reclaimed by **garbage collector** which is an automatic storage management system. Objects are never explicitly deallocated.

The following exceptional condition is associated with the heap:

- If a computation requires more heap than can be made available by the automatic storage management system, the Java Virtual Machine throws an OutOfMemoryError.

Refer [Heap Memory Allocation in Java](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html) to know more about Heap memory allocation and how garbage is collected here

### Method area

JVM has a method area that is *shared among all JVM threads*. Method area stores meta data about the loaded classes and interfaces. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and [constructors](https://www.netjstech.com/2015/04/constructor-in-java.html).

Type information that is stored in method area for each type loaded by JVM is as follows –

- Fully qualified name of the class/interface.
- Fully qualified name of any direct superclass.
- [Modifier](https://www.netjstech.com/2016/07/access-modifiers-in-java-public-private-protected.html) used.
- Fully qualified names of any extended super interfaces.
- Information to distinguish if loaded type is a class or [interface](https://www.netjstech.com/2015/05/interface-in-java.html).

Other than type information method area also stores–

- Run time constant pool.
- Field information which includes field name, type, modifier.
- Method information which includes method name, modifier, return type, parameters.
- Static (class) variables.
- Method code which contains byte code, local variable size, operand stack size.

Method area is generally part of non-heap memory which used to be termed as **PermGen space**. Note here that *PermGen Space is changed to MetaSpace from Java 8*.

- Refer [PermGen Space Removal in Java 8](https://www.netjstech.com/2017/07/permgen-space-removal-in-java-8-metaspace.html) to know more about MetaSpace in Java 8.

The following exceptional condition is associated with the method area:

- If memory in the method area cannot be made available to satisfy an allocation request, the Java Virtual Machine throws an **OutOfMemoryError**.

### Run-Time Constant Pool

A run time constant pool is a per class or per interface storage of the constant_pool table of the class. Constant_pool contains constants (string literals, numeric literals) which are known at compile-time, it also stores method and field references that must be resolved at run time.

Run-time constant pool is shared among the threads and allocated from the JVM’s method area.

Rather than storing everything in byte code a separate constant pool is maintained for the class and the byte code contains reference to the constant pool. These symbolic reference are translated into concrete method reference through dynamic linking.

**As example** – Here is a byte code snippet –

```
0: aload_0     
1: invokespecial #1       // Method java/lang/Object."<init>":()V      
4: aload_0       
5: new           #2       // class javafx/beans/property/SimpleStringProperty
```

If you notice here invokespecial operand has a prefix #1, this number (#1) is reference to the constant pool where instance initialization method is stored.

Same way in line 5 new Opcode is followed by number #2. Which means referring to the 2nd index in the constant pool.

The following exceptional condition is associated with the construction of the run-time constant pool for a class or interface:

- When creating a class or interface, if the construction of the run-time constant pool requires more memory than can be made available in the method area of the Java Virtual Machine, the Java Virtual Machine throws an OutOfMemoryError.

**Reference:** https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5

That's all for this topic **JVM Run-Time Data Areas - Java Memory Allocation**. If you have any doubt or any suggestions to make please drop a comment. Thanks!