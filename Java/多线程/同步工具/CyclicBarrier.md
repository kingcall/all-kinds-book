### Java CyclicBarrier With Examples

There are scenarios in concurrent programming when you want set of threads to wait for each other at a common point until all threads in the set have reached that common point, concurrent util provides a synchronization aid **CyclicBarrier** in Java to handle such scenarios where you want set of threads to wait for each other to reach a common barrier point.

*The barrier is called cyclic because it can be re-used after the waiting threads are released.*

Note that CyclicBarrier was introduced in Java 5 along with other concurrent classes like [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html), [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html), [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) with in java.util.Concurrent package.



### CyclicBarrier class constructors

CyclicBarrier class in Java has following two [constructors](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html)-

```
CyclicBarrier(int parties)
```

Creates a new **CyclicBarrier** that will trip when the given number of **parties** (threads) are waiting upon it, and does not perform a predefined action when the *barrier is tripped*.

```
CyclicBarrier(int parties, Runnable barrierAction)
```

Creates a new **CyclicBarrier** that will trip when the given number of parties (threads) are waiting upon it, and which will execute the given **barrier action** when the barrier is tripped, performed by the last thread entering the barrier.

Here parties parameter signifies the number of threads that must invoke **await()** before the barrier is tripped.

**barrierAction** specifies a thread that will be executed when the barrier is reached.

### How CyclicBarrier in Java is used

First thing is to create a **CyclicBarrier** [object](https://www.netjstech.com/2015/04/object-in-java.html) using any of the two [constructors](https://www.netjstech.com/2015/04/constructor-overloading-in-java.html), specifying the number of threads that should wait for each other. When each [thread](https://www.netjstech.com/2015/06/thread-priorities-java-multithreading.html) reaches the barrier (common point) call **await()** method on the CyclicBarrier object. This will suspend the thread until all the threads call the await() method on the same CyclicBarrier object. Once all the specified threads have called await() method that will trip the barrier and all threads can resume operation.

If the current thread is the last thread to arrive, and a non-null barrier action was supplied in the constructor, then the current thread runs the action before allowing the other threads to continue.

### await() method in CyclicBarrier

await() method has following two forms-

1. public int await() throws InterruptedException, BrokenBarrierException

2. public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException

   

In the second form it Waits until all parties have invoked await on this barrier, or the specified waiting time elapses.

If the current thread is not the last to arrive then it is disabled for [thread scheduling](https://www.netjstech.com/2015/06/thread-priorities-java-multithreading.html) purposes and lies dormant until one of the following things happens:

- The last thread arrives; or
- The specified timeout elapses; (In case of second form) or
- Some other thread interrupts the current thread; or
- Some other thread interrupts one of the other waiting threads; or
- Some other thread times out while waiting for barrier; or
- Some other thread invokes reset() on this barrier.

Await() method returns int which is the arrival index of the current thread, where index (Number of specified threads - 1) indicates the first to arrive and zero indicates the last to arrive.

### CyclicBarrier Java example

Now is the time to see an example of CyclicBarrier in Java. Let's take a scenario where your application needs to read 3 files using 3 threads, parse the read lines and only after reading and parsing all the three files the application should call another thread for further processing. In this scenario we can use CyclicBarrier and provide a runnable action to execute thread once all the threads reach the barrier.

```
public class CyclicBarrierDemo {
  public static void main(String[] args) {
    CyclicBarrier cb = new CyclicBarrier(3, new AfterAction());
    // Initializing three threads to read 3 different files.
    Thread t1 = new Thread(new TxtReader("thread-1", "file-1", cb));
    Thread t2 = new Thread(new TxtReader("thread-2", "file-2", cb));
    Thread t3 = new Thread(new TxtReader("thread-3", "file-3", cb));
    t1.start();
    t2.start();
    t3.start();
    
    System.out.println("Done ");
  }
}

class TxtReader implements Runnable {
  private String threadName;
  private String fileName;
  private CyclicBarrier cb;
  TxtReader(String threadName, String fileName, CyclicBarrier cb){
    this.threadName = threadName;
    this.fileName = fileName;
    this.cb = cb;        
  }
  @Override
  public void run() {
    System.out.println("Reading file " + fileName + " thread " + threadName);    
    try{
      // calling await so the current thread suspends
      cb.await();           
    } catch (InterruptedException e) {
      System.out.println(e);
    } catch (BrokenBarrierException e) {
      System.out.println(e);
    }
  }
}

class AfterAction implements Runnable {
  @Override
  public void run() {
    System.out.println("In after action class, start further processing as all files are read");
  }
}
```

**Output**

```
Done 
Reading file file-2 thread thread-2
Reading file file-1 thread thread-1
Reading file file-3 thread thread-3
In after action class, start further processing as all files are read
```

In the code CyclicBarrier instance is created with 3 parties so the barrier will trip when 3 threads are waiting upon it.

One thing to note here is that [main thread](https://www.netjstech.com/2018/04/main-thread-in-java.html) doesn't block as can be seen from the "Done" printed even before the [threads start](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html). Also it can be seen the AfterAction class is executed once all the three threads call the await() method and the barrier is tripped.

Now if you want to block the main thread then you have to call the await() on the main thread too. Let's take another CyclicBarrier example where two services are started using two separate threads and main thread should start process only after both the services are executed.

```
public class CBExample {
  public static void main(String[] args) {
    CyclicBarrier cb = new CyclicBarrier(3);
    // Creating two threads with CyclicBarrier obj as param
    Thread t1 = new Thread(new FirstService(cb));
    Thread t2 = new Thread(new SecondService(cb));
    System.out.println("starting threads ");
    t1.start();
    t2.start();
        
    try {
      // Calling await for main thread
      cb.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    // once await is called for all the three threads, execution starts again
    System.out.println("In main thread, processing starts again ..... ");
    }
}

class FirstService implements Runnable {
  CyclicBarrier cb;
  FirstService(CyclicBarrier cb){
    this.cb = cb;
  }
  @Override
  public void run() {
    System.out.println("In First service, thread " + Thread.currentThread().getName());
    try {
      // Calling await for Thread-0
      cb.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }      
  }   
}

class SecondService implements Runnable {
  CyclicBarrier cb;
  SecondService(CyclicBarrier cb){
    this.cb = cb;
  }
  @Override
  public void run() {
    System.out.println("In Second service, thread " + Thread.currentThread().getName());
    try {
      // Calling await for Thread-1
      cb.await();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }    
  }    
}
```

**Output**

```
starting threads 
In First service, thread Thread-0
In Second service, thread Thread-1
In main thread, processing starts again .....
```

Here it can be seen that main thread starts only after both the services are executed.

### CyclicBarrier can be reused

Unlike [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html), CyclicBarrier in Java can be reused after the waiting threads are released.

Let's reuse the same example as above where three threads were used to read 3 files. Now three more threads are added to read 3 more files and the same CyclicBarrier object is used with initial count as 3.

```
public class CyclicBarrierDemo {
  public static void main(String[] args) {
    CyclicBarrier cb = new CyclicBarrier(3, new AfterAction());
    // Initializing three threads to read 3 different files.
    Thread t1 = new Thread(new TxtReader("thread-1", "file-1", cb));
    Thread t2 = new Thread(new TxtReader("thread-2", "file-2", cb));
    Thread t3 = new Thread(new TxtReader("thread-3", "file-3", cb));
    
    t1.start();
    t2.start();
    t3.start();
        
    System.out.println("Start another set of threads ");
    
    Thread t4 = new Thread(new TxtReader("thread-4", "file-4", cb));
    Thread t5 = new Thread(new TxtReader("thread-5", "file-5", cb));
    Thread t6 = new Thread(new TxtReader("thread-6", "file-6", cb));
    t4.start();
    t5.start();
    t6.start();           
  }
}

class TxtReader implements Runnable {
  private String threadName;
  private String fileName;
  private CyclicBarrier cb;
  TxtReader(String threadName, String fileName, CyclicBarrier cb){
    this.threadName = threadName;
    this.fileName = fileName;
    this.cb = cb;        
  }
  @Override
  public void run() {
    System.out.println("Reading file " + fileName + " thread " + threadName);    
    try{
      // calling await so the current thread suspends
      cb.await();
        
    } catch (InterruptedException e) {
      System.out.println(e);
    } catch (BrokenBarrierException e) {
      System.out.println(e);
    }
  }
}

class AfterAction implements Runnable {
  @Override
  public void run() {
    System.out.println("In after action class, start further processing 
     as all files are read");
  }
}
```

**Output**

```
Start another set of threads 
Reading file file-1 thread thread-1
Reading file file-2 thread thread-2
Reading file file-3 thread thread-3
In after action class, start further processing as all files are read
Reading file file-4 thread thread-4
Reading file file-5 thread thread-5
Reading file file-6 thread thread-6
In after action class, start further processing as all files are read
```

Here it can be seen that specified runnableAction class is called twice as the CyclicBarrier is reused here. Note that the thread order may be different while executing the code.

**Points to note**

- A CyclicBarrier initialized to N, using its constructor, can be used to make N threads wait using await() and the barrier will be broken once all the N threads call **await()** method.
- A barrierAction can also be provided while creating CyclicBarrier object. This barrierAction will be executed once the barrier is tripped. This barrier action is useful for updating shared-state before any of the parties continue.
- If the current thread is not the last to arrive then it is paused after calling await() and lies dormant until the last thread arrives, current thread or some other waiting thread is interrupted by any other thread, specified timeout elapses (as provided in await()) or some thread calls reset() method.
- **reset()** method resets the barrier to its initial state. If any parties are currently waiting at the barrier, they will return with a **BrokenBarrierException**.
- CyclicBarrier in Java uses an all-or-none breakage model for failed synchronization attempts: If a thread leaves a barrier point prematurely because of interruption, failure, or timeout, all other threads waiting at that barrier point will also leave abnormally via BrokenBarrierException (orInterruptedException if they too were interrupted at about the same time).

That's all for this topic **Java CyclicBarrier With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!