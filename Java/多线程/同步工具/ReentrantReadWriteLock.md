### Java ReentrantReadWriteLock With Examples

This post gives an introduction to ReadWriteLock interface and it's implementing class ReentrantReadWriteLock in Java with usage examples.

**Table of contents**

1. [ReadWriteLock in Java](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReadWriteLockJava)
2. [ReentrantReadWriteLock class in Java](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReentrantReadWriteLockClass)
3. [Java ReentrantReadWriteLock constructors](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReentrantReadWriteLockConstructor)
4. [Fair mode in ReentrantReadWriteLock](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReentrantReadWriteLockFairMode)
5. [Lock downgrading in ReentrantReadWriteLock](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReentrantReadWriteLockDowngrade)
6. [ReentrantReadWriteLock Java example](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html#ReentrantReadWriteLockExp)



### ReadWriteLock in Java

Even in a multi-threading application multiple reads can occur simultaneously for a shared resource. It is only when multiple writes happen simultaneously or intermix of read and write that there is a chance of writing the wrong value or reading the wrong value.

ReadWriteLock in Java uses the same idea in order to boost the performance by having separate pair of locks.

A ReadWriteLock maintains a pair of associated locks-

- One for read-only operations; and
- one for writing.

The **read lock** may be held simultaneously by multiple reader [threads](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html), so long as there are no writers. The **write lock** is exclusive.

Having a pair of **read-write** lock allows for a greater level of concurrency in accessing shared data than that permitted by a mutual exclusion lock. It exploits the fact that while only a single thread at a time (a writer thread) can modify the shared data, in many cases any number of threads can concurrently read the data (hence reader threads).

A read-write lock will improve performance over the use of a mutual exclusion lock *if the frequency of reads is more than writes, duration of the read operations is more than the duration of the writes. It also depends on the contention for the data - that is, the number of threads that will try to read or write the data at the same time*.

**For example**, a collection that is initially populated with data and thereafter infrequently modified, while being frequently searched (such as a directory of some kind) is an ideal candidate for the use of a read-write lock. However, if updates become frequent then the data spends most of its time being exclusively locked and there is little, if any increase in concurrency.

### ReentrantReadWriteLock class in Java

As already mentioned **ReentrantReadWriteLock** is an implementation of the **ReadWriteLock** [interface](https://www.netjstech.com/2015/05/interface-in-java.html) which provides a pair of read-write lock. ReentrantReadWriteLock has similar semantics to [ReentrantLock in Java](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html).

ReentrantReadWriteLock class in Java **does not impose** a reader or writer preference ordering for lock access which means there is no acquisition preference. Though there is an **optional fairness policy**. A ReentrantReadWriteLock is fair or not is specified in its constructor.

ReentrantReadWriteLock in Java allows both read and write locks to reacquire read and write locks in the same fashion as done in Reentrant lock. See an example [here](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html).

### Java ReentrantReadWriteLock constructors

- **ReentrantReadWriteLock()**- Creates a new ReentrantReadWriteLock with default (nonfair) ordering properties.
- **ReentrantReadWriteLock(boolean fair)**- Creates a new ReentrantReadWriteLock with the given fairness policy.

### Fair mode in ReentrantReadWriteLock

When constructed as fair, [threads](https://www.netjstech.com/2015/06/creating-thread-in-java.html) contend for entry using an approximately **arrival-order policy**. When the currently held lock is released, either the longest-waiting single writer thread will be assigned the write lock, or if there is a group of reader threads waiting longer than all waiting writer threads, that group will be assigned the read lock.

### Lock downgrading in ReentrantReadWriteLock

ReentrantReadWriteLock also allows downgrading from the write lock to a read lock. You can first acquire a write lock, then the read lock and then release the write lock. So you are effectively left with a read lock. However, upgrading from a read lock to the write lock is not possible.

**Example of lock downgrading**

If you have a scenario where you want to read from a cache only if it is still valid, using a read lock. If cache is dirty then you need to acquire a write lock and put data in the cache again.

```
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReentrantDowngrade {
  Object data;
  volatile boolean cacheValid;
  ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

  void processCacheData(){
    // first acquire a read lock
    rwl.readLock().lock();
    // check if cache is still valid
    if (!cacheValid) {
      // Must release read lock before acquiring 
      // write lock, as upgrading not possible
      rwl.readLock().unlock();
      rwl.writeLock().lock();
      try {
        // Recheck state because another thread might have
        // acquired write lock and changed state before we did.
        if (!cacheValid) {
          // get fresh data for the cache
          data = ...
          cacheValid = true;
        }
        // Downgrade by acquiring read lock before 
        // releasing write lock
        rwl.readLock().lock();
      } finally {
        // Unlock write, still hold read
        rwl.writeLock().unlock(); 
      }
    }
    try {
      // use cache data
      use(data);
    } finally {
      // Finally release the read lock
      rwl.readLock().unlock();
    }
  }
}
```

### ReentrantReadWriteLock Java example

Let us see **another example** where two threads are using the read lock and one write lock. In class **ReentrantRWDemo** there are two methods, **get()** is used to get data from the [TreeMap](https://www.netjstech.com/2015/11/treemap-in-java.html), so read lock is used. Another method **put()** is used to add value to a map and uses the write lock.

There are 2 classes **ReadThread** which is used for reader threads and another class **WriterThread** is used for write threads. In the program two reader thread and one writer thread are spawned.

```
public class ReentrantRWDemo {
  private final Map<String, String> m = new TreeMap<String, String>();
  private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    
  // get method for getting values from map
  // it can be used by many read threads simultaneously
  public String get(String key) {
    System.out.println("In get method waiting to acquire lock");
    rwl.readLock().lock();
    System.out.println("In get method acquired read lock");
    try { 
      try {
        Thread.sleep(1500);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
      return m.get(key); 
    }
    finally { 
      rwl.readLock().unlock(); 
      System.out.println("In get method released read lock");
    }
  }
    
  // Put method to store  key, value in a map
  // it acquires a write lock so only one thread at a time
  public String put(String key, String value) {
    System.out.println("In put method waiting to acquire lock");
    rwl.writeLock().lock();
    System.out.println("In put method acquired write lock");
    try { 
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
      return m.put(key, value); 
    }
    finally { 
      rwl.writeLock().unlock(); 
      System.out.println("In put method released write lock");
    }
  }
    
  public void display(){
    m.entrySet().forEach(System.out::println);
      
  }
    
  public static void main(String... args) {
    ReentrantRWDemo rwDemo = new ReentrantRWDemo();
    // Putting some values in the map
    rwDemo.put("1", "One");
    rwDemo.put("2", "Two");
    rwDemo.put("3", "Three");
    
    // Starting two read threads and one write thread
    Thread rThread1 = new Thread(new ReadThread(rwDemo));
    Thread wThread = new Thread(new WriterThread(rwDemo));
    Thread rThread2 = new Thread(new ReadThread(rwDemo));
    rThread1.start();
    wThread.start();
    rThread2.start();
    // Wait for the threads to finish, then only go for display method
    try {
      rThread1.join();
      wThread.join();
      rThread2.join();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }    
    rwDemo.display();        
  }
}

class ReadThread implements Runnable {
  ReentrantRWDemo rwDemo;
  ReadThread(ReentrantRWDemo rwDemo){
    this.rwDemo = rwDemo;
  }
  public void run() {
    System.out.println("Value - " + rwDemo.get("1"));
  }
}

class WriterThread implements Runnable {
  ReentrantRWDemo rwDemo;
  WriterThread(ReentrantRWDemo rwDemo){
    this.rwDemo = rwDemo;
  }
  public void run() {
    rwDemo.put("4", "Four");
  }
}
```

**Output**

```
In put method waiting to acquire lock
In put method acquired write lock
In put method released write lock
In put method waiting to acquire lock
In put method acquired write lock
In put method released write lock
In put method waiting to acquire lock
In put method acquired write lock
In put method released write lock
In get method waiting to acquire lock
In put method waiting to acquire lock
In put method acquired write lock
In get method waiting to acquire lock
In put method released write lock
In get method acquired read lock
In get method acquired read lock
In get method released read lock
Value - One
In get method released read lock
Value - One
1=One
2=Two
3=Three
4=Four
```

Here you can ignore the first three set of put prints as these are the messages for the first 3 puts that are used to add values to the TreeMap. As mentioned two reader threads and one writer thread are spawned. In the display I got (for you it may vary) it can be seen that write thread first locks the shared object rwDemo, though Thread.sleep is used to introduce some delay but the reader threads will wait until the write lock is released.

But both read locks can acquire lock simultaneously as confirmed by two consecutive "*In get method acquired read lock*" statement.

Also note that in display() method, [method reference](https://www.netjstech.com/2015/06/method-reference-in-java-8.html) with [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) is used to display the map values. These features are available from Java 8.

[Thread's join method](https://www.netjstech.com/2015/06/isalive-and-join-method-in-java-multi.html) is used so that values are displayed once all the threads have finished.

That's all for this topic **Java ReentrantReadWriteLock With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!