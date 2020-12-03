### PermGen Space Removal in Java 8

Though Java 8 introduced many new features to the Java language which are tangible in a way that you can see and use them like [lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) and [stream API](https://www.netjstech.com/2016/11/stream-api-in-java-8.html). But there are other changes too which you won’t use in your day to day work but are equally important. One of these changes is to remove **PermGen space** in Java and replacing it with **Metaspace**.

Knowledge about the memory management in Java will definitely help you in improving the performance of your application so in this post we’ll see what is PermGen space, why it is replaced with Metaspace and what options are there when using Metaspace in Java.

### What is PermGen space in Java

Let’s have a brief introduction of PermGen space first in order to understand this post better.

*PermGen (Permanent generation) space stores the meta-data about the class*. That involves information about the class hierarchy, information about the [class](https://www.netjstech.com/2015/04/class-in-java.html) like its name, fields, methods, bytecode. Run time constant pool that stores immutable fields that are pooled in order to save space like [String](https://www.netjstech.com/2016/07/string-in-java.html) are also kept in PermGen.

### Problem with PermGen space

PermGen space is contiguous to heap space and the information stored in PermGen is relatively permanent in nature (not garbage collected as swiftly as in young generation like Eden).

The space allocated to PermGen is controlled by the argument **-XX:MaxPermSize** and the default is 64M (30% higher in 64 bit JVM which means around 83M in a 64 bit JVM).

Here point to note is PermGen's size is **Fixed at start-up** and can't be changed dynamically which may lead to **java.lang.OutOfMemoryError: PermGen error**. As your application grows the classes that are loaded will grow and the size of class metadata may become more than what was the size of the PermGen space. Once the permanent generation space is full OutOfMemoryError: PermGen Space error occurs.

### Introducing MetaSpace in Java

*In Java 8 PermGen space has been completely removed* so using argument **-XX:MaxPermSize** won’t do anything.

**For example**, if I go to eclipse where compiler compliance level is Java 1.8 and set a VM argument for MaxPermSize, it will give me warning for this option.

[![permgen space in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:51:43-PermSize%252Berror.png)](https://4.bp.blogspot.com/-4rAcMNOPjPU/WVeOqIA6JSI/AAAAAAAAAZc/RRBV75nV_swKu68j23FuJoAejcf0kH4jwCLcBGAs/s1600/PermSize%2Berror.png)

*“Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0”*

In place of PermGen a new space **MetaSpace** has been introduced. How it differs from PermGen is that *it is not contiguous to Heap space*, it is **part of native memory** instead. You can also opt to use default value for MetaSpace (Though there are some flags provided to limit the memory usage) which theoretically means total available native memory.

### Options with Metaspace

There are some new flags added for Metaspace in JDK 8 two of them are analogous to the option provided with PermGen; PermSize and MaxPermSize. In Metaspace options are **MetaspaceSize** and **MaxMetaspaceSize**. Other two flags are **MinMetaspaceFreeRatio** and **MaxMetaspaceFreeRatio**.

Before going into those options just a brief introduction about the class metadata and how it is garbage collected.

Metadata of the class is deallocated when the corresponding Java class is unloaded. Garbage collections may be induced in order to unload classes and deallocate class metadata. Garbage collection is induced when the space committed for class metadata reaches a certain level (a high-water mark). This level (a high-water mark) in metaspace is defined by MetaspaceSize (Default - 12Mbytes on the 32bit client VM and 16Mbytes on the 32bit server VM with larger sizes on the 64bit VMs). After the garbage collection, the high-water mark may be raised or lowered depending on the amount of space freed from class metadata.

**-XX:MetaspaceSize=<M>** Where <M> is the initial amount of space(the initial high-water-mark) allocated for class metadata (in bytes) that may induce a garbage collection to unload classes. It is raised or lowered based on the options MaxMetaspaceFreeRatio and MinMetaspaceFreeRatio. If the committed space available for class metadata as a percentage of the total committed space for class metadata is greater than MaxMetaspaceFreeRatio, then the high-water mark will be lowered. If it is less than MinMetaspaceFreeRatio, then the high-water mark will be raised.

**-XX:MaxMetaspaceSize=<M>** Where <M> is the maximum amount of space to be allocated for class metadata (in bytes). The amount of native memory that can be used for class metadata is by default unlimited. Use the option MaxMetaspaceSize to put an upper limit on the amount of native memory used for class metadata.

**-XX:MinMetaspaceFreeRatio=<M>** Where <M> is the figure in percentage (minimum percentage of class metadata capacity free after a GC). It is used in conjunction with the MetaspaceSize option to determine whether high-water mark should be raised so as not to induce another garbage collection too soon.

**-XX:MaxMetaspaceFreeRatio=<M>** Where <M> is the figure in percentage (maximum percentage of class metadata capacity free after a GC). It is used in conjunction with the MetaspaceSize option to determine whether high-water mark should be reduced.

### Using default option

You may be tempted to use the default with the Metaspace so that more and more space is allocated for the class meta-data but in my opinion that may result in false sense of security that every thing is fine with your application where as in reality your application may have memory leak issues.

You should provide enough space for the Metaspace so that classes of your application and any third party libraries are loaded then you should monitor it for any memory leaks in case that space is not enough. There are instances when I had to increase PermGen space when application itself was big and using third party tools like reporting engines. But that was a genuine case where it had to be increased up to certain level. If it still gave “ OutOfMemoryError” that meant memory leaks which had to be addressed rather than increasing PermGen space even more.

That's all for this topic **PermGen Space Removal in Java 8**. If you have any doubt or any suggestions to make please drop a comment. Thanks!