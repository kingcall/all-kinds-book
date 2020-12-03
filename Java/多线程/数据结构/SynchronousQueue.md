### Java SynchronousQueue With Examples

SynchronousQueue which is an implementation of the [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) interface was added in Java 5 along with other concurrent utilities like [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [ReentrantLock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html), [Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html), [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html) etc.

How SynchronousQueue differs from other implementations of BlockingQueue like [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) and [LinkedBlockingQueue](https://www.netjstech.com/2016/03/linkedblockingqueue-in-java.html) is that SynchronousQueue in Java *does not have any internal capacity, not even a capacity of one*. In SynchronousQueue each insert operation must wait for a corresponding remove operation by another thread, and vice versa.

What that means is, if you put an element in SynchronousQueue using **put()** method it will wait for another thread to receive it, you can't put any other element in the SynchronousQueue as it is blocked. Same way in case there is thread to remove an element (using **take()** method) but there is no element in the queue it will block and wait for an element in the queue.

### Java SynchronousQueue differs in functionality

Since SynchronousQueue has a very special functionality which differs from other BlockingQueue implementations so methods in Java SynchronusQueue behave a little differently. Actually calling it a Queue itself is a bit of wrong statement as there is never more than one element present. It's more of a point-to-point **handoff**.

As an example take peek() method, which in other [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) implementations work as follows-

**peek()** - Retrieves, but does not remove, the head of this queue, or returns null if this queue is empty.

Since in SynchronousQueue an element is only present when you try to remove it so peek() method in this class always returns null.

**Iterator** in SynchronousQueue returns an empty iterator in which hasNext always returns false.

For purposes of other Collection methods a SynchronousQueue acts as an empty collection, so methods like

- **contains**- Always returns false. A SynchronousQueue has no internal capacity.
- **remove**- Always returns false. A SynchronousQueue has no internal capacity.
- **isEmpty()**- Always returns true. A SynchronousQueue has no internal capacity.

### Nulls are not allowed in SynchronousQueue

SynchronousQueue in Java does not permit null elements. Attempt to add null to the queue results in NullPointerException.

```
public class SynchroQDemo {
  public static void main(String[] args) {
    BlockingQueue<String> sq = new SynchronousQueue<String>();
    sq.add(null);
  }
}
```

**Output**

```
Exception in thread "main" java.lang.NullPointerException
 at java.util.concurrent.SynchronousQueue.offer(SynchronousQueue.java:912)
 at java.util.AbstractQueue.add(AbstractQueue.java:95)
 at org.netjs.programs.SynchroQDemo.main(SynchroQDemo.java:11)
```

### Fairness policy in Java SynchronousQueue

The SynchronousQueue class in Java supports an optional fairness policy for ordering waiting producer and consumer threads. By default, this ordering is not guaranteed. However, a queue constructed with fairness set to true grants [threads](https://www.netjstech.com/2015/06/difference-between-thread-and-process-java.html) access in FIFO order.

Thus SynchronousQueue has two [constructors](https://www.netjstech.com/2015/04/constructor-chaining-in-java-calling-one-constructor-from-another.html)-

- **SynchronousQueue()**- Creates a SynchronousQueue with nonfair access policy.
- **SynchronousQueue(boolean fair)**- Creates a SynchronousQueue with the specified fairness policy.

### Usage of SynchronousQueue in Java

As already stated that calling it a Queue itself is a bit of wrong statement as there is never more than one element present. It's more of a point-to-point handoff. So SynchronousQueue is well suited for handoff designs, in which an object running in one [thread](https://www.netjstech.com/2015/06/thread-priorities-java-multithreading.html) must sync up with an object running in another thread in order to hand it some information, event, or task.

### Producer Consumer in Java using SynchronousQueue

Let's create a producer consumer using the SynchronousQueue. There is a **Producer class** which will put elements in the Queue, there is a **Consumer class** which will retrieve elements from the queue. Also a **SharedClass** which acts as a bounded buffer.

To see the functionality of SynchronousQueue in practice there are some changes in the code. You can see after every **get()** in the Consumer class some delay has been induced using **sleep()** method of the thread. What you can see here is that **put** will not add more than one element even if the consumer thread is sleeping. It will put one element and block.

Also in producer for loop runs 5 times though in consumer it is only 3 times. So here you can see SynchronousQueue will not add other elements for which there is no thread to retrieve and it will block. So you'll have to forcibly stop this program.

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.SynchronousQueue;

public class SyncQDemo {

  public static void main(String[] args) {
    SharedClass buffer = new SharedClass();
    // Starting two threads
    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(new Producer(buffer));
    executor.execute(new Consumer(buffer));
    executor.shutdown();
  }
}

class Producer implements Runnable{
  SharedClass buffer;
  Producer(SharedClass buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      System.out.println("Putting - " + i);
      buffer.put(i);
    }
  }
}

class Consumer implements Runnable{
  SharedClass buffer;
  Consumer(SharedClass buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 3; i++){
      buffer.get();
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
    }
  }    
}

//Shared class used by threads
class SharedClass{
  int i;
  // SynchronousQueue
  BlockingQueue<Integer> syncQ = new SynchronousQueue<Integer>();
  public void get(){
    // retrieve from synchronousQueue
    try {
      System.out.println("Consumer recd - " + syncQ.take());
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
    
  public void put(int i){
    this.i = i;
    try {        
      // putting in synchronousQueue
      syncQ.put(i);
      System.out.println("In Queue " + i);
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    
  }
}
```

**Output**

```
Putting - 0
Consumer recd - 0
In Queue 0
Putting - 1
Consumer recd - 1
In Queue 1
Putting - 2
Consumer recd - 2
In Queue 2
Putting - 3 
```

**Reference** : https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/SynchronousQueue.html

That's all for this topic **Java SynchronousQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!