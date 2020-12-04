### Java LinkedTransferQueue With Examples

LinkedTransferQueue is added in **Java 7** and it is an implementation of **TransferQueue** interface.

### TransferQueue interface in Java

It will be worthwhile to spend some time knowing the TransferQueue [interface](https://www.netjstech.com/2015/05/marker-interface-in-java.html) here.

**TransferQueue** interface, also added in Java 7, extends [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html). The **extra functionality** provided by TransferQueue interface is that it *provides blocking method which will wait until other thread receives your element*.

That's how it differs from **BlockingQueue** where you can only put element into queue or retrieve element from queue and block if queue is full (while you are putting elements) or block if queue is empty (while you are retrieving elements). So BlockingQueue won't block until consumer thread removes any particular element (Though [SynchronousQueue](https://www.netjstech.com/2016/03/synchronousqueue-in-java.html) provided that kind of functionality but more as one-to-one handoff, not as a queue.)

**TransferQueue** has a blocking method **transfer(E e)** which will ensure that the element is transferred to the consumer, it will wait if required to do so. More precisely, transfers the specified element immediately if there exists a consumer already waiting to receive it (in BlockingQueue.take() or timed poll), else waits until the element is received by a consumer.

**TransferQueue** also has **put()** method, just like BlockingQueue which will just enqueue the element without waiting for consumer to retrieve the element.

It also has non-blocking and time-out tryTransfer() method-

- **tryTransfer(E e)**- Transfers the element to a waiting consumer immediately, if possible.
- **tryTransfer(E e, long timeout, TimeUnit unit)**- Transfers the element to a consumer if it is possible to do so before the timeout elapses.

TransferQueue also has querying methods like **getWaitingConsumerCount()** and **hasWaitingConsumer()**

- **getWaitingConsumerCount()**- Returns an estimate of the number of consumers waiting to receive elements via BlockingQueue.take() or timed poll.
- **hasWaitingConsumer()**- Returns true if there is at least one consumer waiting to receive an element via BlockingQueue.take() or timed poll.

### Java LinkedTransferQueue

LinkedTransferQueue, as already mentioned is an implementation of the TransferQueue. LinkedTransferQueue in Java is an unbounded queue and stores elements as linked nodes.

LinkedTransferQueue orders elements FIFO (first-in-first-out) with respect to any given producer. The head of the queue is that element that has been on the queue the longest time for some producer. The tail of the queue is that element that has been on the queue the shortest time for some producer.

### Java LinkedTransferQueue constructors

- **LinkedTransferQueue()**- Creates an initially empty LinkedTransferQueue.
- **LinkedTransferQueue(Collection<? extends E> c)**- Creates a LinkedTransferQueue initially containing the elements of the given collection, added in traversal order of the collection's iterator.

### Producer Consumer Java example using LinkedTransferQueue

Let's create a producer consumer using LinkedTransferQueue. There is a producer [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) which will put elements into the queue and a consumer thread which will retrieve elements from the queue. Also a shared class which will be used by both producer and consumer threads.

To show the functionality of transfer() method delay is induced in consumer thread, since elements are stored using transfer() method so the producer will wait unless until consumer [thread](https://www.netjstech.com/2015/06/thread-priorities-java-multithreading.html) retrieves the element. Producer won't keep on adding the element to the queue even if consumer thread is sleeping it gets blocked.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedTransferQueue;
import java.util.concurrent.TransferQueue;

public class LinkedTQDemo {
  public static void main(String[] args) {
    // buffer class used by both threads
    SharedTQ buffer = new SharedTQ();
    // Starting two threads
    ExecutorService executor = Executors.newFixedThreadPool(2);
    // Executing producer
    executor.execute(new TProd(buffer));
    // Executing consumer
    executor.execute(new TCon(buffer));
    executor.shutdown();
  }
}


/**
 * Producer class
 */
class TProd implements Runnable{
  SharedTQ buffer;
  TProd(SharedTQ buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      buffer.put(i);
    }
  }
}

/**
 * Consumer class
 */
class TCon implements Runnable{
  SharedTQ buffer;
  TCon(SharedTQ buffer){
    this.buffer = buffer;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      try {
        // introducing some delay using sleep
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        System.out.println("Error while putting in the queue " + e.getMessage());
      }
      buffer.get();
    }
  }    
}

//Shared class used by threads
class SharedTQ{
  int i;
  // TransferQueue
  TransferQueue<Integer> ltQueue = new LinkedTransferQueue<Integer>();
  
  public void get(){
    try {
      // take method to get from TransferQueue
      System.out.println("Consumer recd - " + ltQueue.take());
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
    
  public void put(int i){
    this.i = i;
    try {
      System.out.println("Putting - " + i);
      // putting in TransferQueue
      ltQueue.transfer(i);        
    }
    catch (InterruptedException e) {
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

That's all for this topic **Java LinkedTransferQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!