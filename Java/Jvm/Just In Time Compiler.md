### Just In Time Compiler (JIT) in Java

In this post we'll see what is Just-In-Time (JIT) compiler in Java, how it works and how JIT compiler optimizes code.

When we start studying Java we get to know one of the prominent feature of Java is that **Java is platform independent**.

### How is platform independence achieved in Java

In a short sentence, Java is platform independent because of the fact it is *both compiled and interpreted*.

To explain it further; first when we compile our JAVA code **.class** files are generated having the **platform independent byte code** which is interpreted to machine native code at run time by the JVM.

- Refer [What Are JVM, JRE And JDK in Java](https://www.netjstech.com/2015/05/what-are-jvm-jre-and-jdk-in-java.html) to read about JVM.

### Where does JIT compiler fit

This two step process, though makes Java platform independent, where you are not worried about the OS or processor while writing your code, but at the same time execution is slow because byte code is interpreted to the native code at the run time. Because of this interpretation of the byte code to the host CPU's native instruction set there is an overhead of processor and memory usage which results in slow execution of the code.

That’s where **Just-In-Time (JIT) compiler** comes into the picture. In simple terms you can say JIT compiler compiles the already compiled code to the native code as per the processor. Since this compilation of the compiled byte code to the native machine code is done at the run time (Just in time) thus the name Just in Time compiler (JIT).

### How does JIT compiler work in Java

While the code is executed JVM automatically monitors which methods are being executed frequently and start marking the methods that are “hot”. Note that JIT compiler initially itself doesn’t compile all the code at once, initially methods are interpreted from byte code.

Marked "hot" methods are scheduled for compilation into machine code. This compilation into machine code happens on a separate JVM thread without interrupting the execution of the program. While the hot method is compiled by the JIT compiler, the JVM will keep using the interpreted version of the method and switch over to the compiled method once it is ready.

Initial compilation of the method is quick but the resulting code may not be as efficient as it could be. If a method is used quite frequently the system can get a performance boost if the code for that particular method is regenerated in a more efficient way. That is why same method may get compiled more than once and more optimizations may get applied while the code is compiled.

Once the method is marked for compilation and compiled, its count is set to zero. If method call again reaches the call count threshold the method is recompiled and more optimizations are applied to the method to make it more efficient.

That recompilation and more optimization happens because the optimization techniques are often layered and once the compiler has applied one optimization then only it may be able to see other optimizations that can be applied.

[![JIT compiler in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:47:37-JIT%252BCompiler.png)](https://3.bp.blogspot.com/-e6-5UERUOxg/Wdxc2exi4KI/AAAAAAAAAbI/9dhSvq-tpnMO_Jlv7fU84RVCo3ZHwtMpACPcBGAYYCw/s1600/JIT%2BCompiler.png)

### How does JIT complier optimize code

Some of the optimization techniques used by JIT complier are-

1. Inlining methods

   – One of the most common technique for optimizing code is method inlining. In method inlining the method call is substituted by the method body in the places where method is called. By replacing the method body in the method call the call to the method, resultant creation of stack frames is saved.

   **Example code**

   ```
   class A {
     B b;
     public void doProcessing() {
       x = b.getValue();
       ...
       ...
       y = b.getValue();
       ………………………….
   
     }
   }
   
   class B {
      int value;
      final int getValue() {
         return value;
      }
   }
   ```

   **Inlining final method**

   Notice in Class A method calls (getValue) are eliminated.

   ```
   class A {
     B b;
     public void doProcessing() {
       x = b.value;
       ...
       ...
       y = b.value;
       ………………………….
   
     }
   }
   ```

   **Removing redundant loads**

   y = b.value; is replaced with y = x so that local value itself can be used.

   ```
   class A {
     B b;
     public void doProcessing() {
       x = b.value;
       ...
       ...
       ...
       y = x;
       ………………………….
   
     }
   }
   ```

2. Monomorphic dispatch

   – Java being an object-oriented language uses subtyping/polymorphism which means most of the method invocations are virtual method lookup. The JVM checks how many different implementations of the method are there –

   - If there is only one implementation it is a **monomorphic dispatch**.
   - If there are 2 then it is **bimorphic dispatch**, in case of more than 2 it is **megamorphic dispatch**.

   In the monomorphic case, path-dependent types (sub type or super type) does not happen. So, JIT compiler knows the exact method definitions that will be called when methods are called on the passed object, because there is no need to check which override is actually being used. This means JIT compiler can eliminate the overhead of doing virtual method lookup.

3. Removing unwanted synchronization

   – Overhead of obtaining

    

   locks

    

   to enter a monitor can be eliminated by removing the

    

   synchronization

   , if synchronized block can only be accessed by a single thread.

   **As example**

   ```
   public void methodA{
     B b = new B()
     Synchronized(b){
      …..
      ……
     }
   }
   ```

   Here new object is created with in the method so every [thread](https://www.netjstech.com/2015/06/thread-priorities-java-multithreading.html) will have its own object thus the synchronized block has no effect here. JVM will notice that and the method will be optimized by JIT compiler.

4. Merging adjacent synchronized block with same object

    

   – If there are adjacent synchronized block on the same object those will be merged as an optimization by the JIT compiler.

   ```
   Public void testMethod{
     synchronized(Test.class){
      ……
      …
     }
     synchronized(Test.class){
      ……
      …
     }
   }
   ```

   Here these 2 synchronized blocks can be merged to create a single synchronized block.

There are many other optimizations applied like **loop optimization**, **dead code elimination**.

Reference-

http://www.oracle.com/technetwork/articles/java/architect-evans-pt1-2266278.html
https://docs.oracle.com/cd/E15289_01/doc.40/e15058/underst_jit.htm

That's all for this topic **Just In Time Compiler (JIT) in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!