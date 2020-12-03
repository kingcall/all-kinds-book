### Java ReentrantLock With Examples

Part of the Concurrent util **java.util.concurrent.locks** package has added support for locks, which provides an alternative to using [Synchronized in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) in scenarios where we need to control access to a shared resource. In this post we'll talk about one of the concrete implementation of the lock interface called **ReentrantLock in Java**. There is another implementation [ReentrantReadWriteLock](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html) which is implementation of **ReadWriteLock** interface.



ReentrantLock was added in Java 5 along with other concurrent features like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) with in java.util.Concurrent package, to develop concurrent applications.



### What is ReentrantLock and why needed

ReentrantLock class in Java is a concrete implementation of the Lock interface which is present in **java.util.concurrent.locks** package. One question which comes to mind is why this separate functionality for locking is needed when there already is Synchronized keyword in Java which provides the same functionality.

As you must be knowing every object created in Java has one mutually exclusive lock associated with it. When you are using synchronized you are using that **lock implicitly** (with no other feature) whereas when you are using any of the lock implementation (like ReentrantLock) you are using that **lock explicitly**. Which means there are methods like **lock()** to acquire the lock and **unlock()** to release the lock. Along with that **ReentrantLock** in Java provides many other features like **fairness**, **ability to interrupt** and a thread waiting for a lock only for a **specified period**.

According to the **Java docs** "*ReentrantLock is a reentrant mutual exclusion Lock with the same basic behavior and semantics as the implicit monitor lock accessed using synchronized methods and statements, but with extended capabilities.*"

- Refer [Difference between ReentrantLock and Synchronized](https://www.netjstech.com/2016/02/difference-between-reentrantlock-and-synchronized-java.html) to know more about the difference between ReentrantLock and Synchronized keyword.

### Why is it called ReentrantLock

It is called ReentrantLock as there is an *acquisition count associated with the lock* which means when you use **lock()** method to acquire a lock and you get it then the acquisition count is 1.

A Reentrant lock will also *allow the lock holder to enter another block of code with the same lock [object](https://www.netjstech.com/2015/04/object-in-java.html)* as thread already owns it. In that case, if a thread that holds the lock acquires it again, the acquisition count is incremented and the lock then needs to be **released twice** to truly release the lock.

Let's see it with an **example** to make it clear-

Here two threads are created. In the [run method of the thread](https://www.netjstech.com/2015/06/what-if-run-method-called-directly-instead-of-start-java.html) class methodA() is called which uses the same lock object to control access.

So you will see two things here-

- Whichever thread acquires the lock will also be able to access methodA() critical section as it already holds the lock. Only thing is acquisition count will become 2.
- Since in the methodA(), unlock() method is not used to release the lock (remember we need to release it twice as acquisition count is 2). So another thread will never get a chance to acquire a lock.

```
public class ReentrantDemo {
  public static void main(String[] args) {
    ReentrantLock rLock = new ReentrantLock();
    Thread t1 = new Thread(new Display("Thread-1", rLock));
    Thread t2 = new Thread(new Display("Thread-2", rLock));
    System.out.println("starting threads ");
    t1.start();
    t2.start();
  }
}

class Display implements Runnable {
  private String threadName;
  ReentrantLock lock;
  Display(String threadName, ReentrantLock lock){
    this.threadName = threadName;
    this.lock = lock;
  }
  @Override
  public void run() {
    System.out.println("In Display run method, thread " + threadName + 
     " is waiting to get lock");
    //acquiring lock
    lock.lock();
    try {
      System.out.println("Thread " + threadName + "has got lock");
      methodA();
    } finally{
      lock.unlock();
    }        
  }
    
  public void methodA(){
    System.out.println("In Display methodA, thread " + threadName + 
      " is waiting to get lock");
    //try {        
      lock.lock();      
      System.out.println("Thread " + threadName + "has got lock");
      System.out.println("Count of locks held by thread " + threadName + 
       " - " + lock.getHoldCount());
      // Not calling unlock
      /*} finally{
      lock.unlock();
    }*/
  }    
}
```

**Output**

```
starting threads 
In Display run method, thread Thread-1 is waiting to get lock
In Display run method, thread Thread-2 is waiting to get lock
Thread Thread-1has got lock
In Display methodA, thread Thread-1 is waiting to get lock
Thread Thread-1has got lock
Count of locks held by thread Thread-1 - 2
```

Here it can be seen that both [thread starts](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) and **Thread-1** acquires a lock, Thread-1 will acquire the same lock again in **methodA()** but there it is **not released**. You can notice the method **lock.getHoldCount()** which gives the count of holds on this lock by the current thread. Since **unlock()** method is **not called** so lock is never released that is why **Thread-2** never gets a chance to acquire a lock. You can see it never goes beyond this message "*In Display run method, thread Thread-2 is waiting to get lock*".

Note that in different runs thread which acquires a lock may vary.

Now let's **correct the code** and use the **unlock()** method to release the lock and see what happens.

```
public class ReentrantDemo {
  public static void main(String[] args) {
    ReentrantLock rLock = new ReentrantLock();
    Thread t1 = new Thread(new Display("Thread-1", rLock));
    Thread t2 = new Thread(new Display("Thread-2", rLock));
    System.out.println("starting threads ");
    t1.start();
    t2.start();
  }
}

class Display implements Runnable {
  private String threadName;
  ReentrantLock lock;
  Display(String threadName, ReentrantLock lock){
    this.threadName = threadName;
    this.lock = lock;
  }
  @Override
  public void run() {
    System.out.println("In Display run method, thread " + threadName + 
     " is waiting to get lock");
    //acquiring lock
    lock.lock();
    try {
      System.out.println("Thread " + threadName + "has got lock");
      methodA();
    } finally{
      lock.unlock();
    }      
  }
    
  public void methodA(){
    System.out.println("In Display methodA, thread " + threadName 
     + " is waiting to get lock");
    //acquiring lock
    lock.lock();
    try {
      System.out.println("Thread " + threadName + "has got lock");
      System.out.println("Count of locks held by thread " + threadName 
       + " - " + lock.getHoldCount());
    } finally{
      lock.unlock();
    }
  }  
}
```

**Output**

```
starting threads 
In Display run method, thread Thread-1 is waiting to get lock
In Display run method, thread Thread-2 is waiting to get lock
Thread Thread-1has got lock
In Display methodA, thread Thread-1 is waiting to get lock
Thread Thread-1has got lock
Count of locks held by thread Thread-1 - 2
Thread Thread-2has got lock
In Display methodA, thread Thread-2 is waiting to get lock
Thread Thread-2has got lock
Count of locks held by thread Thread-2 - 2
```

Now both threads are able to run as the locks are properly release after acquiring.

### Convention while using ReentrantLock in Java

If you had noticed one thing in the above code **lock.lock()** method is always called before the try block. When you are using Reentrantlock in Java, it is a *recommended practice to always immediately follow a call to lock with a try block*.

If you will call lock() method with in the try block and some thing goes wrong while acquiring the lock [finally block](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html) will still be called and there you will have lock.unlock() method. So you will end up unlocking the lock which was never acquired and that will result in **IllegalMonitorStateException**, that’s why it is recommended to call lock() method before try block.

At the same time you do want to **unlock** the acquired lock if something goes wrong after acquiring the lock, that is why immediately follow a call to lock with try block.

### Features of ReentrantLock in Java

ReentrantLock provides many features like fairness, ability to interrupt and a thread waiting for a lock only for a specified period. Let's have a look at some of these features.

1. **Fairness**- ReentrantLock has one constructor which takes boolean value as an argument. That lets you choose whether you want a fair or an unfair lock depending upon whether the boolean value is true or false. A fair lock is one where the threads acquire the lock in the same order they asked for it; whereas in case of an unfair lock a thread can sometimes acquire a lock before another thread that asked for it first.

2. ```
   public ReentrantLock(boolean fair)
   ```

3. Lock interruptibly

   \- ReentrantLock provides a method lockInterruptibly, where the thread acquires a lock if it is not interrupted.

   ```
   public void lockInterruptibly() throws InterruptedException
    
   ```

4. **Ability to check if the lock is being held**- ReentrantLock in Java provides ability to check if the lock is already being held using tryLock() method.

   **tryLock()**- Acquires the lock only if it is not held by another thread at the time of invocation.

   tryLock(long timeout, TimeUnit unit) - Acquires the lock if it is not held by another thread within the given waiting time and the current thread has not been interrupted.

5. Some of the other methods in ReentrantLock class are as follows-

   - **getHoldCount()**- Queries the number of holds on this lock by the current thread.
   - **getWaitingThreads(Condition condition)** - Returns a collection containing those threads that may be waiting on the given condition associated with this lock.
   - **isHeldByCurrentThread()**- Queries if this lock is held by the current thread.
   - **isLocked()**- Queries if this lock is held by any thread.

### Drawbacks of ReentrantLock in Java

1. Need to wrap lock acquisitions in a [try/finally block](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html) and release the lock in finally block. Otherwise, if the critical section code threw an exception, the lock might never be released.
2. Need to call **unlock()** method explicitly. Forgetting to do that will result in lock never getting released which will create a lots of problem and make it very hard to detect performance problems.
   With [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html), the JVM ensures that locks are automatically released.

### ReentrantLock Java example code

Let us see one more example of Reentrant lock where a resource is shared between two threads and the access is controlled using locks.

Thread.sleep is used to induce some delay, in that case also another thread won't break in. Only when the unlock() method is called and the lock is released other thread gets a chance.

```
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {
  public static void main(String[] args) {
    ReentrantLock rLock = new ReentrantLock();
    Thread t1 = new Thread(new Counter("Thread-1", rLock));
    Thread t2 = new Thread(new Counter("Thread-2", rLock));
    System.out.println("starting threads ");
    t1.start();
    t2.start();
  }
}

// Shared class for threads
class SharedResource{
  static int count = 0;
}

class Counter implements Runnable {
  private String threadName;
  ReentrantLock lock;
  Counter(String threadName, ReentrantLock lock){
    this.threadName = threadName;
    this.lock = lock;
  }
  @Override
  public void run() {
    System.out.println("In Counter run method, thread " + threadName 
    + " is waiting to get lock");
    // acquiring the lock
    lock.lock();
    try {
      System.out.println("Thread " + threadName + " has got lock");
      SharedResource.count++;
      System.out.println("Thread " + threadName + 
       " Count " + SharedResource.count);
      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
    } finally{
      System.out.println("Thread " + threadName 
       + " releasing lock");
      // releasing the lock
      lock.unlock();
    }    
  }
}
```

**Output**

```
starting threads 
In Counter run method, thread Thread-1 is waiting to get lock
In Counter run method, thread Thread-2 is waiting to get lock
Thread Thread-1 has got lock
Thread Thread-1 Count 1
Thread Thread-1 releasing lock
Thread Thread-2 has got lock
Thread Thread-2 Count 2
Thread Thread-2 releasing lock
```

**Points to remember**

- ReentrantLock in Java is a reentrant mutual exclusion Lock with the same basic behavior and semantics as the implicit monitor lock accessed using synchronized methods with some extended features.
- Some of these features include fairness, ability to interrupt and a thread waiting for a lock only for a specified period.
- Thread which is currently holding a lock can repeatedly enter the same lock, acquisition count increments as many times current thread acquires the same lock.
- **lock** has to be released as many times as it has been acquired.
- Failure to call **unlock()** as many times as the lock is acquired will result is lock not being released and the thread will continue to hold it.
- **unlock()** method should be called in a [finally block](https://www.netjstech.com/2015/05/finally-block-in-java-exception-handling.html). Otherwise, if the critical section code threw an exception, the lock might never be released

That's all for this topic **Java ReentrantLock With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!