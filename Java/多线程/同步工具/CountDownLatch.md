### Java CountDownLatch With Examples

There are scenarios in an application when you want one or more [threads](https://www.netjstech.com/2015/06/creating-thread-in-java.html) to wait until one or more events being performed in other threads complete. **CountDownLatch** in Java concurrent API helps in handling such scenarios.

Note that CountDownLatch was introduced in Java 5 along with other concurrent classes like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html), [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) with in **java.util.Concurrent** package.

### How CountDownLatch is used

CountDownLatch in Java, as the name suggests, can be visualized as a latch that is released only after the given number of events occur. When an instance of CountDownLatch is created it is initialized with a count. This count denotes the number of times event must occur before waiting threads can pass through the latch.

Note that a CountDownLatch initialized to N can be used either ways-

- To make one thread wait until N threads have completed some action, or
- Some action has been completed N times (may be by a single thread).

Each time one of these events occur count is decremented using the **countdown()** method of the CountDownLatch class. Waiting threads are released when the count reaches zero.

Thread(s) that are waiting for the latch to release are blocked using **await()** method.

### Java CountDownLatch Constructor

```
CountDownLatch(int count)
```

Constructs a **CountDownLatch** initialized with the given count. Here count specifies the number of events that must happen in order for the latch to open.

### await() and countdown() methods in CountDownLatch class

await() and countdown() are two main methods in CountDownLatch class which control the working of the latch.

**await() method**- A thread that waits on the latch to open calls **await()** method, await() method has two forms.

1. **public void await() throws InterruptedException**

Causes the current thread to wait until the latch has counted down to zero, unless the thread is interrupted. If the current count is zero then this method returns immediately.

If the current count is greater than zero then the current thread becomes disabled for thread scheduling purposes and lies dormant until one of two things happen:

- The count reaches zero due to invocations of the **countDown()** method
- Some other thread interrupts the current thread.

2. **public boolean await(long timeout, TimeUnit unit) throws InterruptedException**

Causes the current thread to wait until the latch has counted down to zero, unless the thread is interrupted, or the specified waiting time elapses, the waiting time is specified by an object of TimeUnit enumeration.

If the current count is zero then this method returns immediately with the value true. If the current count is greater than zero then the current thread becomes disabled for thread scheduling purposes and lies dormant until one of three things happen:

- The count reaches zero due to invocations of the **countDown()** method.
- Some other thread interrupts the current thread.
- The specified waiting time elapses.

**countdown() method**- Threads which are executing the events signal the completion of the event by calling countDown() method.

```
public void countDown()
```

Decrements the count of the latch, releasing all waiting threads if the count reaches zero.

### CountDownLatch Java Example program

That's a lot of theory so let's see an example to make it clearer and see how **await()**, **countdown()** and the [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) to provide count are actually used.

Let's take a scenario where your application needs to read 3 files, parse the read lines and only after reading and parsing all the three files the application should move ahead to do some processing with the parsed objects.
So here we'll have three separate threads reading three separate files and the main thread awaits until all the three threads finish and call countdown().

```
public class CountdownlatchDemo {
  public static void main(String[] args) {
    CountDownLatch cdl = new CountDownLatch(3);
    // Initializing three threads to read 3 different files.
    Thread t1 = new Thread(new FileReader("thread-1", "file-1", cdl));
    Thread t2 = new Thread(new FileReader("thread-2", "file-2", cdl));
    Thread t3 = new Thread(new FileReader("thread-3", "file-3", cdl));
    t1.start();
    t2.start();
    t3.start();
    try {
      // main thread waiting till all the files are read
      cdl.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    System.out.println("Files are read ... Start further processing");
  }
}

class FileReader implements Runnable {
  private String threadName;
  private String fileName;
  private CountDownLatch cdl;
  FileReader(String threadName, String fileName, CountDownLatch cdl){
    this.threadName = threadName;
    this.fileName = fileName;
    this.cdl = cdl;        
  }
  @Override
  public void run() {
    System.out.println("Reading file " + fileName + " thread " + threadName);
    // do countdown here
    cdl.countDown();
  } 
}
```

**Output**

```
Reading file file-1 thread thread-1
Reading file file-3 thread thread-3
Reading file file-2 thread thread-2
Files are read ... Start further processing
```

Here it can be seen that inside [main() method](https://www.netjstech.com/2015/04/why-main-method-is-static-in-java.html), **CountDownLatch** instance cdl is created with an initial count of 3. Then three instances of **FileReader** are created that start three new threads. Then the main thread calls **await()** on cdl, which causes the main [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) to wait until cdl count has been decremented three times. Notice that cdl instance is passed as a parameter to the FileReader constructor that cdl instance is used to call countdown() method in order to decrement the count. Once the countdown reaches zero, the latch opens allowing the main thread to resume.

You can comment the code where **await()** is called, then main thread will resume even before all the 3 files are read, so you see in these type of scenarios where you want the thread to resume only after certain events occur then CountDownLatch is a powerful [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) aid that allows one or more threads to wait for certain events to finish in other threads.

From the above example if you got the feeling that whatever count you have given in the CountDownLatch, you should spawn the same number of threads for countdown then that is a **wrong understanding**. As I have mentioned it **depends on the number of events**, so you can very well have a **single thread** with a loop and decrementing the count there.

Let's change the **example** used above to have single thread and use for loop to countdown.

```
public class CountdownlatchDemo {
  public static void main(String[] args) {
    CountDownLatch cdl = new CountDownLatch(3);
    // Initializing three threads to read 3 different files.
    Thread t1 = new Thread(new FileReader("thread-1", "file-1", cdl));
    /*Thread t2 = new Thread(new FileReader("thread-2", "file-2", cdl));
    Thread t3 = new Thread(new FileReader("thread-3", "file-3", cdl));*/
    t1.start();
    /*t2.start();
    t3.start();*/
    try {
      // main thread waiting till all the files are read
      cdl.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    System.out.println("Files are read ... Start further processing");
  }
}

class FileReader implements Runnable {
  private String threadName;
  private String fileName;
  private CountDownLatch cdl;
  FileReader(String threadName, String fileName, CountDownLatch cdl){
    this.threadName = threadName;
    this.fileName = fileName;
    this.cdl = cdl;        
  }
  @Override
  public void run() {
    for(int i = 0; i < 3; i++){
      System.out.println("Reading file " + fileName + " thread " + threadName);
      // do countdown here
      cdl.countDown();
    }
  }
}
```

**Output**

```
Reading file file-1 thread thread-1
Reading file file-1 thread thread-1
Reading file file-1 thread thread-1
Files are read ... Start further processing
```

Here you can see that only a single thread is used and countdown is done on the number of events. So it is true both ways. *A CountDownLatch initialized to N can be used to make one thread wait until N threads have completed some action, or some action has been completed N times*.

### Usage of CountDownLatch in Java

As you have seen in the example you can use CountDownLatch when you want to break your code in such a way that more than one thread can process the part of the code but you can start further processing only when all the threads which are working on some part of the code have finished. Once all the threads have finished main thread can come out of the await (as the latch is released) and start further processing.

You can also use CountDownLatch to test concurrency by giving a certain count in the CountDownLatch Constructor and start that many threads. Also there may be more than one waiting thread, so that scenario how waiting threads behave once the countdown reaches zero (as all of them will be released at once) can also be tested.

If you have some external dependencies and once all the dependencies are up and running then only you should start processing in your application. That kind of scenario can also be handled with CountDownLatch.

### CountDownLatch in Java can not be reused

One point to remember is **CountDownLatch** cannot be reused. *Once the countdown reaches zero any further call to await() method won't block any thread*. It won't throw any exception either.

Let's see an **example**. We'll use the same example as above and spawn 3 more threads once the first three set of threads are done.

```
public class CountdownlatchDemo {
  public static void main(String[] args) {
    CountDownLatch cdl = new CountDownLatch(3);
    // Initializing three threads to read 3 different files.
    Thread t1 = new Thread(new FileReader("thread-1", "file-1", cdl));
    Thread t2 = new Thread(new FileReader("thread-2", "file-2", cdl));
    Thread t3 = new Thread(new FileReader("thread-3", "file-3", cdl));
    t1.start();
    t2.start();
    t3.start();
    try {
      // main thread waiting till all the files are read
      cdl.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    System.out.println("Files are read ... Start further processing");
    Thread t4 = new Thread(new FileReader("thread-4", "file-4", cdl));
    Thread t5 = new Thread(new FileReader("thread-5", "file-5", cdl));
    Thread t6 = new Thread(new FileReader("thread-6", "file-6", cdl));
    t4.start();
    t5.start();
    t6.start();
    try {
      // main thread waiting till all the files are read
      cdl.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
      System.out.println("Files are read again ... Start further processing");
  }
}

class FileReader implements Runnable {
  private String threadName;
  private String fileName;
  private CountDownLatch cdl;
  FileReader(String threadName, String fileName, CountDownLatch cdl){
    this.threadName = threadName;
    this.fileName = fileName;
    this.cdl = cdl;        
  }
  @Override
  public void run() {
    System.out.println("Reading file " + fileName + " thread " + threadName);
    // do countdown here
    cdl.countDown();
  }
}
```

**Output**

```
Reading file file-2 thread thread-2
Reading file file-3 thread thread-3
Reading file file-1 thread thread-1
Files are read ... Start further processing
Files are read again ... Start further processing
Reading file file-4 thread thread-4
Reading file file-6 thread thread-6
Reading file file-5 thread thread-5
```

Here note that await() is called again after starting thread4, thread5 and thread6 but it doesn't block main thread as it did for the first three threads. "**Files are read again ... Start further processing**" is printed even before the next three threads are processed. Another concurrent utility CyclicBarrier can be resued infact that is one of the [difference between CountDownLatch and CyclicBarrier](https://www.netjstech.com/2016/01/difference-between-countdownlatch-and-cyclicbarrier-in-java.html).

**Points to note**

- A **CountDownLatch** initialized to N, using its constructor, can be used to make one (or more) thread wait until N threads have completed some action, or some action has been completed N times.
- **countDown()** method is used to decrement the count, once the count reaches zero the latch is released.
- **await()** method is used to block the thread(s) waiting for the latch to release.
- **CountDownLatch** cannot be reused. Once the countdown reaches zero any further call to await() method won't block any thread.

That's all for this topic **Java CountDownLatch With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!