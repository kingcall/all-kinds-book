### Garbage Collection in Java

In the post [Heap Memory Allocation in Java](https://www.netjstech.com/2017/11/heap-memory-allocation-in-java.html) I have already explained why heap is divided into generations and how it helps in garbage collection by adopting an approach known as “**generational collection**”.

In this post we’ll see more details about garbage collection in Java, basic process of garbage collection, garbage collection tuning and types of garbage collectors available in Java.

### What is Garbage Collection

Garbage collection in Java is an automatic process of inspecting the heap memory and identifying which objects are in use and which are not and deleting the unused objects after identification.

Here [object](https://www.netjstech.com/2015/04/object-in-java.html) which is in use (referenced object) means that a part of your program is still holding a reference to the object. Unused object (unreferenced object) means not referenced by any part of your program and can be deleted safely.

### Basic Garbage Collection Process

Basic garbage collection process in Java has three steps -

1. Marking
2. Sweeping
3. Compacting

**Marking**

The first step in the garbage collection process is called marking. In this step GC iterates through the memory and identifies which objects are still in use and which objects are unused.

**Sweeping**

In this step of sweeping (normal deletion) objects which are already marked as unused in the “marking” step are removed in order to free space.

**Compacting**

Deleting the unused objects fragments the memory which results in further memory allocation which are not contiguous. That is why there is another step which moves the reference objects together which results in having a contiguous free space thus making the new memory allocation much easier and faster.

[![garbage collection compacting process](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:48:27-Fragmented%252Bmemory.png)](https://1.bp.blogspot.com/-8X6IwkHlVYE/Wgr8ZFrZDII/AAAAAAAAAcs/ES4QeIG31tAKswhiD1b5VwQb2DZ3gNupQCPcBGAYYCw/s1600/Fragmented%2Bmemory.png)

[![garbage collection in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:48:27-Compacting.png)](https://4.bp.blogspot.com/-CZMqyEPyqlg/Wgr8V-TV9MI/AAAAAAAAAcs/eq21So-bjLE5CNO44PXyKB0Z4f-9nqW6ACPcBGAYYCw/s1600/Compacting.png)

### Performance parameters for GC

There are two goals for any application with respect to garbage collection –

- Maximum pause time goal
- Application throughput goal

**Maximum Pause Time Goal**

All minor garbage collections and major garbage collections are "**Stop the World**" events which means all application threads are stopped until the garbage collection completes. So, the goal here is to minimize this pause time or restrict it by putting an upper limit.

That is what this parameter does; maximum pause time goal is to limit the longest of these pauses.

Note here that only parallel collector provides a command line option to specify a maximum pause time goal.

**Command line option**

```
-XX:MaxGCPauseMillis=<nnn> 
```

This option is a hint to the garbage collector that pause times of <nnn> milliseconds or less are desired.

**Throughput Goal**

Since garbage collection is "Stop the world event" stopping all the application threads so we can divide the total time into-

- The time spent collecting garbage
- The application time

The throughput goal is measured in terms of the time spent collecting garbage and the time spent outside of garbage collection.

Note here that only parallel collector provides a command line option to specify a throughput goal.

**Command line option**

```
-XX:GCTimeRatio=<nnn>
```

The ratio of garbage collection time to application time is 1 / (1 + <nnn>). **For example**, -XX:GCTimeRatio=9 sets a goal of 1/10th or 10% of the total time for garbage collection.

### Garbage collectors available in Java

The Java HotSpot VM has three different types of collectors-

1. Serial GC
2. Parallel GC also known as Throughput Collector.
3. Mostly Concurrent Collector – Java HotSpot offers two types of mostly concurrent collector.
   - Concurrent Mark Sweep (CMS) Collector
   - Garbage First, Garbage collector (G1 Collector)

**Serial Collector**

The serial collector uses a single [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html) to do both minor and major collections. The serial collector is best-suited to single processor machines, because it cannot take advantage of multiprocessor hardware. Since only a single thread performs garbage collection so Serial GC is most suited for applications that do not have low pause time requirements.

**Command Line Switches**

The serial collector is selected by default on certain hardware (client machines) and operating system configurations. Serial collector can be explicitly enabled with the option

```
-XX:+UseSerialGC.
```

**Parallel Collector**

In parallel collector (also known as the throughput collector) multiple threads are used for garbage collection.

The command line option to enable parallel collector is **-XX:+UseParallelGC**.

By default, with this option, both minor and major collections are executed in parallel to further reduce garbage collection overhead.

The feature that enables the parallel collector to perform major collections in parallel is known as **parallel compaction**. If parallel compaction is not enabled then major collections are performed using a single thread.

Command line option to turn off parallel compaction is : **-XX:+UseParallelOldGC**.

Only parallel collector provides command line options to tune the performance parameters as stated above.

**Command Line Switches**

- Maximum Garbage Collection Pause Time: The maximum pause time goal is specified with the command-line option **-XX:MaxGCPauseMillis=<N>**.
- Throughput goal: The throughput goal is specified by the command-line option -XX:GCTimeRatio=<N>

**Concurrent Mark Sweep (CMS) Collector**

Concurrent Mark Sweep (CMS) Collector, as the name suggests, performs garbage collection concurrently while the application is running. Since application also keep running that results in low pause time but the application throughput is affected because processor resources are shared.

This collector should be considered for any application with a low pause time requirement.

Like other available collectors the CMS collector is generational; thus both minor and major collections occur. The CMS collector attempts to reduce pause times due to major collections by using separate garbage collector threads to trace the reachable objects concurrently with the execution of the application threads. CMS (Concurrent Mark Sweep) garbage collection does not do compaction.

**Pauses in CMS collector**

The CMS collector pauses an application twice during a concurrent collection cycle. The first pause marks those objects as live which are directly reachable from the roots and from elsewhere in the heap. This first pause is referred to as the initial mark pause.

The second pause comes at the end of the concurrent tracing phase and finds objects that were missed by the concurrent tracing due to updates by the application threads of references in an object after the CMS collector had finished tracing that object. This second pause is referred to as the remark pause.

**Command Line Switches**

- The command line option to enable CMS collector is **-XX:+UseConcMarkSweepGC**.
- Command line option to set the number of threads **-XX:ParallelCMSThreads=<n>**

**Garbage-First Garbage Collector**

The Garbage-First (G1) garbage collector is a server-style garbage collector which is suited for multiprocessor machines with large memories. G1 garbage collector minimizes the garbage collection (GC) pause time while achieving high throughput at the same time.

It minimizes the garbage collection (GC) pause time by trying to adhere to pause time goals which is set using the flag **MaxGCPauseMillis**.

**Technique used by G1 collector**

Technique used by G1 collector to achieve high performance and pause time goals is explained below-

G1 Collector partitions the heap into a set of equally sized heap regions.

[![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:48:27-G1%252BCollector.png)](https://4.bp.blogspot.com/-niw6no59S8k/XFGWoo1L6eI/AAAAAAAABK8/4mbTie_Pg2o4ZLDL7EAx01S3yQP5eSKjQCLcBGAs/s1600/G1%2BCollector.png)

Image credit-[ https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)

Initially G1 performs a concurrent global marking throughout the heap to determine which objects are still referenced and which are not (unreferenced). Once the marking is done G1 knows which regions are mostly empty. It collects these mostly empty regions first thus the name Garbage-First. By using this method of garbage collection G1 frees the large amount of free space by sweeping only a small region of heap.

G1 tries to adhere to the specified pause time target (defined by using flag MaxGCPauseMillis) by using a pause prediction model. It calculates how many regions can be collected within the given pause time limit and collects only those regions.

**G1 is generational in a logical sense**

As already stated heap is partitioned into a set of equally sized heap regions. A set of empty regions is designated as the logical young generation. Objects are allocated from that logical young generation and that young generation (those regions of heap) is garbage collected when it is full. In some cases, regions outside the set of young regions (regions designated as tenured generation) can be garbage collected at the same time. This is referred to as a **mixed collection**.

G1 collector also compacts the memory by copying the live objects to selected, initially empty regions.

### G1 collector changes in Java Versions

G1 collector is the default garbage collector Java 9 onward. Till Java 8 default garbage collector was Parallel GC.

Though G1 garbage collector is designed to avoid full collections but a full GC occurs if memory is not reclaimed fast enough with in the pause time target. In Java 9 implementation of the full GC for G1 used a single threaded mark-sweep-compact algorithm which meant an increased pause time.

Java 10 onward G1 collector uses the parallelized mark-sweep-compact algorithm to minimize the impact for users experiencing full GCs.

**Command Line Switches**

The command line option to enable G1 collector is **-XX:+UseG1GC**.

Reference -

http://www.oracle.com/technetwork/articles/java/architect-evans-pt1-2266278.html
https://docs.oracle.com/cd/E15289_01/doc.40/e15058/underst_jit.htm

That's all for this topic **Garbage Collection in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!