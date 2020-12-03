### Java ArrayBlockingQueue With Examples

Java ArrayBlockingQueue which is an implementation of the [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) interface was added in Java 5 along with other concurrent utilities like [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html), [ReentrantReadWriteLock](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html), [Exchanger](https://www.netjstech.com/2016/02/exchanger-in-java-concurrency.html), [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html) etc.

**ArrayBlockingQueue** in Java is a *bounded blocking queue which internally uses an array to store elements*. ArrayBlockingQueue orders elements in **FIFO (first-in-first-out) order**. When new elements are inserted, those are inserted at the tail of the queue. At the time of retrieval, elements are retrieved from the head of the queue.

Since **ArrayBlockingQueue** is bounded it means you can't add unlimited number of elements in it. ArrayBlockingQueue has to be created with some **initial capacity** and that capacity cannot be changed later. Attempts to put an element into a full queue will result in the operation blocking; attempts to take an element from an empty queue will similarly block.

### Java ArrayBlockingQueue Constructors

1. **ArrayBlockingQueue(int capacity)**- Creates an **ArrayBlockingQueue** with the given (fixed) capacity and default access policy.
2. **ArrayBlockingQueue(int capacity, boolean fair)**- Creates an ArrayBlockingQueue with the given (fixed) capacity and the specified access policy. A queue created with fairness set to true grants access to waiting producer and consumer threads in FIFO order.
3. **ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c)**- Creates an ArrayBlockingQueue with the given (fixed) capacity, the specified access policy and initially containing the elements of the given collection, added in traversal order of the collection's [iterator](https://www.netjstech.com/2015/08/list-iterator-in-java.html).

### ArrayBlockingQueue Java example

Let's create a producer consumer using the ArrayBlockingQueue. Initial capacity of the ArrayBlockingQueue will be kept one so that producer and consumer both get a chance alternatively.

Values will be inserted in the ArrayBlockingQueue using **put()** method, which will block if the space is full.

Values will be retrieved from the ArrayBlockingQueue using **take()** method, which retrieves and removes the head of this queue, waiting if necessary until an element becomes available.

```
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ArrayBQDemo {
  public static void main(String[] args) {
    Buffer buffer = new Buffer();
    // Starting two threads
    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(new ProdTask(buffer));
    executor.execute(new ConTask(buffer));
    executor.shutdown();
  }
}

class ProdTask implements Runnable{
  Buffer buffer;
  ProdTask(Buffer buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      buffer.put(i);
    }
  }
}

class ConTask implements Runnable{
  Buffer buffer;
  ConTask(Buffer buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      buffer.get();;
    }
  }    
}

//Shared class used by threads
class Buffer{
  int i;
  // Bouded ArrayBlockingQueue of size 1
  BlockingQueue<Integer> arrayBlockingQ = new ArrayBlockingQueue<Integer>(1);
  public void get(){
    try {
      // take method to get from blockingqueue
      System.out.println("Consumer recd - " + arrayBlockingQ.take());
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
    
  public void put(int i){
    this.i = i;
    try {
      // putting in blocking queue
      arrayBlockingQ.put(i);
      System.out.println("Putting - " + i);
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
Putting - 1
Consumer recd - 1
Putting - 2
Consumer recd - 2
Putting - 3
Consumer recd - 3
Putting - 4
Consumer recd - 4
```

Here Buffer is the shared class used by both the producer and consumer threads. Two [threads are created](https://www.netjstech.com/2015/06/creating-thread-in-java.html) here one as producer thread and another as consumer thread.

Compare this producer-consumer implementation with the [producer-consumer implementation using semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html) or with the [producer-consumer implementation using wait-notify](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) and you will see how much simplified it is using a BlockingQueue implementation.

That's all for this topic **Java ArrayBlockingQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!