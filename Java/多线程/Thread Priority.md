### Thread Priority in Java Multi-Threading

When we talk about [thread](https://www.netjstech.com/2015/06/difference-between-thread-and-process-java.html) we use terms like [concurrent](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) threads or threads executing concurrently. But in reality threads also run one at a time (at least in a single CPU system), threads are given CPU cycle in a time shared manner to simulate concurrency. The order in which multiple threads will be executed is decided by thread scheduler and thread priority is used by the thread scheduler to make that decision. This article is about the thread priorities available in Java multi-threading and how to get and set Java thread priority.

### Java Thread Priority

When a [thread is created in Java](https://www.netjstech.com/2015/06/creating-thread-in-java.html), it inherits its priority from the thread that creates it. Thread's priority can be modified at any time after its creation using the **setPriority()** method which is a member of Thread class.

**General form of setPriority() method**

```
public final void setPriority(int newPriority)
```

Here newPriority specifies the new thread priority setting for the calling thread. *Thread priority in Java ranges from 1 (least important) to 10 (most important) and the default priority level is 5* .

In Java Thread class, **three constants are provided** to define **min**, **max** and **default** priority of a thread.

```
/**
* The minimum priority that a thread can have.
*/
public final static int MIN_PRIORITY = 1;

/**
* The default priority that is assigned to a thread.
*/
public final static int NORM_PRIORITY = 5;

/**
* The maximum priority that a thread can have.
*/
public final static int MAX_PRIORITY = 10;
```

Let's write some code to see Java thread priority in action. First we'll write a program where no priorities are set so each thread will have the default priority (i.e. 5).

### Getting thread priority in Java

If you want to check priority of a thread that can be done using getPriority() method.

- **int getPriority()**- Returns this thread's priority.

```
//This class' shared object will be accessed by threads
class LoopValues implements Runnable{

  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName() + 
          " Priority is " + Thread.currentThread().getPriority());
    for (int i = 1; i <= 10; i++) {
      System.out.println(Thread.currentThread().getName() + " : " + i);
    }      
  }
}

public class ThreadPriorityDemo {
  public static void main(String[] args) {
    Thread thread1 = new Thread(new LoopValues(), "Thread-1");
    Thread thread2 = new Thread(new LoopValues(), "Thread-2");            
    thread1.start();
    thread2.start();           
    try {        
      //Wait for the threads to finish
      thread1.join();
      thread2.join();            
    } catch (InterruptedException ex) {
      ex.printStackTrace();
    }            
    System.out.println("Done with looping values");
  }
}
```

**Output**

```
Thread-1 Priority is 5
Thread-2 Priority is 5
Thread-1 : 1
Thread-1 : 2
Thread-1 : 3
Thread-1 : 4
Thread-2 : 1
Thread-2 : 2
Thread-2 : 3
Thread-2 : 4
Thread-2 : 5
Thread-2 : 6
Thread-2 : 7
Thread-2 : 8
Thread-2 : 9
Thread-2 : 10
Thread-1 : 5
Thread-1 : 6
Thread-1 : 7
Thread-1 : 8
Thread-1 : 9
Thread-1 : 10
Done with looping values
```

Here it can be seen that output has a mix of both the threads. Please note that the output may vary with each run.

### Setting thread priority Java example

Now we'll set the thread priorities so that one thread has the **MAX_PRIORITY (10)** and another has the **MIN_PRIORITY (1)**.

```
//This class' shared object will be accessed by threads
class LoopValues implements Runnable{

  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName() + 
      " Priority is " + Thread.currentThread().getPriority());
    for (int i = 1; i <= 10; i++) {
      System.out.println(Thread.currentThread().getName() + " : " + i);
    }  
  }
}

public class ThreadPriorityDemo {
  public static void main(String[] args) {
    Thread thread1 = new Thread(new LoopValues(), "Thread-1");
    Thread thread2 = new Thread(new LoopValues(), "Thread-2");      
    thread1.setPriority(Thread.MAX_PRIORITY);
    thread2.setPriority(Thread.MIN_PRIORITY);
    thread1.start();
    thread2.start();           
    try {
      //Wait for the threads to finish
      thread1.join();
      thread2.join();        
    } catch (InterruptedException ex) {
      ex.printStackTrace();
    }       
    System.out.println("Done with looping values");
  }
}
```

**Output**

```
Thread-1 Priority is 10
Thread-1 : 1
Thread-1 : 2
Thread-1 : 3
Thread-1 : 4
Thread-1 : 5
Thread-1 : 6
Thread-1 : 7
Thread-1 : 8
Thread-1 : 9
Thread-1 : 10
Thread-2 Priority is 1
Thread-2 : 1
Thread-2 : 2
Thread-2 : 3
Thread-2 : 4
Thread-2 : 5
Thread-2 : 6
Thread-2 : 7
Thread-2 : 8
Thread-2 : 9
Thread-2 : 10
Done with looping values
```

It can be seen that the thread which has the highest priority finishes first then the second thread starts its execution. Please note that even with this priority setting there may be a mixed output or the thread with the min priority may execute first. That brings us to the next section *why we can't rely that much on thread priority in Java*.

### Caution with thread priority in Java

Thread priority rules are dependent on the host platform (native OS). Host platform may have different priorities for the threads which may be more than what priorities are defined in Java or may be less than that. Java run time system maps the thread priorities to the priority levels of the host platform. So JVM may schedule a thread according to the priority defined in Java multithreading but when that thread actually gets the CPU cycle also depends upon the priorities defined by the host platform.

That's all for this topic **Thread Priorities in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!