### Heap Memory Allocation in Java

In the post [JVM run-time data areas](https://www.netjstech.com/2017/10/jvm-run-time-data-areas-java-memory-allocation.html) we have already got a brief idea about the memory areas used while running a Java application. In this post we’ll talk about Java Heap memory space in detail – How Heap memory is allocated, how garbage collection happens, heap memory tuning and heap memory profiling.

**Table of contents**

1. [Heap memory allocation in Java](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html#HeapMemory)
2. [Why is heap memory managed in generations](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html#HeapMemorygen)
3. [How does garbage collection work on the heap](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html#HeapMemoryGC)
4. [VM Heap Size tuning options in Java](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html#HeapSizeTuning)
5. [Heap memory profiling](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html#Heapprofiling)



### Heap memory allocation in Java

Heap is the JVM run-time data area where the Java objects reside. Apart from Java objects, memory for instance variables and arrays is also allocated on the heap. Heap is created on the JVM start-up and shared among all Java Virtual Machine [threads](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html).

Heap memory in Java is divided into two areas (or generations)-

- **Young Space (generation)**- The young generation consists of eden and two survivor spaces. Most objects are initially allocated in eden.
- **Old Space (or Tenured generation)**– When objects (which have survived garbage collection) residing in young space have reached a certain age threshold they are moved to old space.

These generations have their own memory pool allocated by the JVM.

[![Java heap memory generations](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:51:02-Heap%252BGenerations.png)](https://4.bp.blogspot.com/-D9fWr-aoQf8/Wf6ggShIDQI/AAAAAAAAAbk/XXNdaZIaLisyUbrglEm_2UK4xZV29jmnQCPcBGAYYCw/s1600/Heap%2BGenerations.png)

**Heap memory areas**

### Why is heap memory managed in generations

Here let’s deviate a little and try to understand why memory is managed in generations and what benefit do we get out of it. In one line it can be explained as these division into generations makes garbage collection more efficient.

As you must be knowing Garbage collection in Java is an automatic storage management system that frees space in the heap by reclaiming memory for objects that are not having any reference. Note that, an object is considered garbage when it can no longer be reached from any pointer in the running program.

A very simplistic garbage collection algorithms will iterate over every reachable object. Any objects left over are considered garbage. With this algorithm the time taken is proportional to the number of live objects in the whole heap.

In order to minimize the time it takes to do garbage collection the approach taken by JVM is known as *"generational collection"*. This approach works on the weak **weak generational hypothesis**, which states that *most objects survive for only a short period of time*.

In order to optimize for this hypothesis, memory is managed in generations. Initially objects are allocated in the young generation (Eden space) and most objects die there.

When the young generation fills up, it results in a **minor collection** (Minor GC) in which only the young generation is collected, that way rather than iterating over the objects in the whole heap only a small portion of the Heap (Young generation) is inspected for dead objects. If the hypothesis stated above holds true then most of the objects will die there and reclaimed with in the younger generation.

Only a small portion of the objects will survive the garbage collection in younger generation and after a certain time lapse will move to tenured generation. Eventually, the tenured generation will fill up and must be garbage collected, that will result in a **major collection** (Major GC), in which the entire heap is collected.

### How does garbage collection work on the heap

Now that you know that Heap is divided into generations and how this division helps garbage collector to run more efficiently as GC has to go through only a part of the heap space and within that less space iteration most of the objects (Remember most of the objects die young!) can be garbage collected.

Let’s see in more detail how garbage collection works across these generations and what happens when minor GC is triggered and what happens when major GC is triggered.

- Within the young generation initially any new objects are allocated to the eden space. Both survivor spaces (S0 and S1) are initially empty.
- A minor garbage collection is triggered when eden space is filled up. All the unreferenced objects are garbage collected and reference objects are moved to the first survivor space (S0). One survivor space is empty at any time.
- When the minor garbage collection is triggered next time, all the unreferenced objects are garbage collected and reference objects are moved to the survivor space. Note that this time referenced objects are moved to the second survivor space (S1). There is one more step; objects from the previous minor GC on the first survivor space (S0) have their age incremented and get moved to S1. Note that one survivor space is empty at any time.
- This process of clearing eden space, moving the referenced objects to one of the survivor spaces, incrementing the age for the surviving objects keeps on repeating with every minor GC. There is also a check for object’s age reaching a certain threshold. Objects that reach the threshold are moved from young generation to the old generation.
- With every minor GC aged objects will be moved from young to old generation space.
- When that movement of object fills up the tenured space that triggers a major GC in which the entire heap is collected. Major garbage collection lasts much longer than minor collections because a significantly larger number of objects are involved. Apart from garbage collecting the objects, major GC also compacts the space as it is defragmented from cleaning of the objects.

**☞** Refer [Garbage Collection in Java](https://www.netjstech.com/2017/11/garbage-collection-in-java.html) to know more about garbage collection process and available garbage collectors in Java

### VM Heap Size tuning options in Java

Heap memory in Java will have three things – live objects, dead objects and some portion of the memory which is still free. The JVM heap size determines the frequency of garbage collection and the time spent on collecting garbage.

If you set a large heap size frequency of garbage collection will be less but the time spent on collecting garbage will be more because of the large size (means having more objects to inspect). On the other hand if you do the opposite then time spent on collecting garbage will be less but frequency will increase as smaller heap will fill faster.

An acceptable heap size is application-specific and should be adjusted using the provided options after analyzing the actual time and frequency of garbage collections.

**Java heap size options**

1. Task:

    

   Setting initial heap size

   **Option:** -Xms

   **As example:** -Xms40m

2. Task:

    

   Setting maximum heap size

   **Option:** -Xms

   **As example:** -Xmx512m

   At initialization of the JVM, the entire space for the heap is reserved. The size of the space reserved can be specified with the -Xmx option. If the value provided with the -Xms parameter is smaller than the value provided with the -Xmx parameter, then all of the space that is reserved for the heap is not committed to the virtual machine. The different generationso of the heap (Young and tenured) can grow to that limit (provided with -Xmx) as and when needed.

   It is recommended that you set initial heap size (-Xms) equal to the maximum heap size (-Xmx) to minimize garbage collections.

3. If you have not given same values for parameters -Xms and -Xmx then the virtual machine will grow or shrink the heap at each garbage collection to try to keep the proportion of free space to live objects within a specific range. The options to set this target ranges are-

4. Task:

    

   To maintain minimum percentage heap free space

   **Option:** -XX:MinHeapFreeRatio=<minimum>

   **As example:** -XX:MinHeapFreeRatio=40

5. Task:

    

   To maintain maximum percentage heap free space

   **Option:** -XX:MaxHeapFreeRatio=<maximum>

   **As example:** -XX:MaxHeapFreeRatio=70

   With the parameters as used in the example let's try to understand these options better. If the percent of free space in a generation falls below 40%, then the generation will be expanded to maintain 40% free space, up to the maximum allowed size of the generation. Similarly, if the free space exceeds 70%, then the generation will be contracted so that only 70% of the space is free, subject to the minimum size of the generation.

6. After heap size setting parameters another option that affects GC performance is the proportion of the heap dedicated to the young generation. If you set the young generation to be bigger, minor collections will occur less often. But that would mean a smaller tenured generation, which will increase the frequency of major collections.

7. Three options for tuning the young generation are-

8. Task:

    

   Setting the Young generation heap size

   **Option:** -XX:NewSize

   It is recommended to set -XX:NewSize to be one-fourth the size of the heap size.

9. Task:

    

   Set the maximum size of the Young Generation heap size.

   Option: -XX:MaxNewSize

10. Task:

     

    Controlling the ratio between young and tenured generation

    **Option:** -XX:NewRatio

    **As example** if you set -XX:NewRatio=3 that would mean the ratio between the young and tenured generation is 1:3. The size of the eden + survivor spaces will be one-fourth of the total heap size.

11. You can also tune the size of the survivor spaces, for that you can use the parameter SurvivorRatio.

12. Task:

     

    Tune the size of the survivor spaces

    **Option:** -XX:SurvivorRatio

13. **As example** if you set -XX:SurvivorRatio=6 that would mean the ratio between eden and a survivor space is 1:6. Which means each survivor space will be one-sixth the size of eden, thus one-eighth the size of the young generation.

### Heap memory profiling

Though there are many tools available to profile the memory, I am mentioning one I have already used Java VisulalVM. It’s free and comes bundled with JDK itself. Another tool you will find in the same location is jconsole which is also a monitoring tool.

To launch you just need to go to bin directory of your installed JDK folder and launch **jvisualvm** from there.

On the left side pane it will show the running Java applications, just click on the one you want to inspect.

Here I am demonstrating its use with a very simple application where I have created a thread and in that thread I am creating 5000 objects of another class with some thread pause (by using [sleep method](https://www.netjstech.com/2015/07/difference-between-sleep-and-wait-java-threading.html)) in between. For this program I changed the **-Xms** and **-Xmx** so that the heap is small.

First image shows the heap when the program is just started, that’s why you see a very little variance in the used heap.

[![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:51:03-Heap%252Bmonitoring-2.png)](https://2.bp.blogspot.com/-abGIgg1dyRQ/WgCOfJHC2UI/AAAAAAAAAcI/KQnLJ0siUc0L_yfd3x394uhPOhqKH4szACPcBGAYYCw/s1600/Heap%2Bmonitoring-2.png)

Second image is a snapshot of the heap memory when garbage collection occurred that is why you can see a sudden dip in the used heap memory. If you see at 3:50 PM in the graph you will see a GC activity which has taken 1.3% of CPU time.

[![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:51:03-Heap%252Bmonitoring-3.png)](https://2.bp.blogspot.com/-VPB9vhCReBw/WgCOfTXiUTI/AAAAAAAAAcI/FOAoxsiXAGI1yBNpFv3ZyEGAsE6yl32fACPcBGAYYCw/s1600/Heap%2Bmonitoring-3.png)

Third image also shows a GC activity. At 3:52 PM you can see a barely visible blue mark showing GC activity and a corresponding dip in the used heap memory.

[![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:51:03-Heap%252Bmonitoring-4.png)](https://3.bp.blogspot.com/-a-cfN8wlaVY/WgCOfeIxwQI/AAAAAAAAAcI/KwlrHIQn7WQO23rqjAJF4BdhRgOoUz6rwCPcBGAYYCw/s1600/Heap%2Bmonitoring-4.png)

So using VisualVM GUI tool you can monitor your application’s memory usage, also analyze process threads and get a thread dump. Also profile the performance of your application by analyzing CPU and memory usage.

- Reference-

- https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html#sthref16
- http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html

That's all for this topic **Heap Memory Allocation in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!