### Java LinkedBlockingQueue With Examples

LinkedBlockingQueue in Java is an implementation of [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html). It is added in **Java 5** along with other concurrent utilities like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html), [ConcurentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [CopyOnWriteArraySet](https://www.netjstech.com/2016/03/copyonwritearrayset-in-java-concurrency.html) etc.

**LinkedBlockingQueue** in Java internally uses **linked nodes** to store elements. It is **optionally bounded** and that's where it differs from another implementation of BlockingQueue, [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) which is a **bounded queue**, another difference between the two is how elements are stored internally **ArrayBlockingQueue** uses [array](https://www.netjstech.com/2015/09/how-to-convert-array-to-arraylist-in-java.html) internally whereas **LinkedBlockingQueue** uses linked nodes. Since LinkedBlockingQueue is **optionally bounded** so it has both types of [constructors](https://www.netjstech.com/2015/04/constructor-in-java.html)

- one where initial capacity can be passed thus making it **bounded**.
- Or

- without any capacity thus making it **unbounded**. Note that in case no initial capacity is defined capacity of LinkedBlockingQueue is **Integer.MAX_VALUE**.

### Java LinkedBlockingQueue constructors

- **LinkedBlockingQueue(int capacity)**- Creates a LinkedBlockingQueue with the given (fixed) capacity.
- **LinkedBlockingQueue()**- Creates a LinkedBlockingQueue with a capacity of Integer.MAX_VALUE.

### How elements are stored in LinkedBlockingQueue

**LinkedBlockingQueue** orders elements in first-in-first-out (**FIFO**) order. The head of the queue is that element that has been on the queue the longest time. The tail of the queue is that element that has been on the queue the shortest time. New elements are inserted at the tail of the queue, and the queue retrieval operations obtain elements at the head of the queue.

### Java LinkedBlockingQueue methods

Since LinkedBlockingQueue also implements Iterable, Collection and Queue interfaces apart from BlockingQueue interface so this class and its iterator implement all of the optional methods of the Collection and Iterator interfaces.

For the same type of functionality like **adding an element** or **removing an element** LinkedBlockingQueue provides different methods which will [throw exception](https://www.netjstech.com/2015/05/throw-statement-in-java-exception-handling.html), block or timeout based on the method used. Refer [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) to see the table of methods.

### LinkedBlockingQueue Java example

Let's create a **producer consumer** using the **LinkedBlockingQueue**. Initial capacity of the LinkedBlockingQueue will be kept one so that producer and consumer both get a chance alternatively.

Values will be inserted in the LinkedBlockingQueue using **put()** method, which will block if the space is full. Values will be retrieved from the LinkedBlockingQueue using **take()** method, which retrieves and removes the head of this queue, waiting if necessary until an element becomes available.

```
public class LinkedBQDemo {
  public static void main(String[] args) {
    Buffer buffer = new Buffer();
    // Starting two threads
    ExecutorService executor = Executors.newFixedThreadPool(2);
    executor.execute(new ProdTask(buffer));
    executor.execute(new ConTask(buffer));
    executor.shutdown();
  }
}

/**
 * Producer class
*/
class LinkProdTask implements Runnable{
  Buffer buffer;
  LinkProdTask(Buffer buffer){
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
 * Consumer Class
*/
class LinkConTask implements Runnable{
  Buffer buffer;
  LinkConTask(Buffer buffer){
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
class LinkBuffer{
  int i;
  // Bounded LinkedBlockingQueue of size 1
  BlockingQueue<Integer> linkedBlockingQ = new LinkedBlockingQueue<Integer>(1);
  public void get(){
    try {
      // take method to get from blockingqueue
      int i = linkedBlockingQ.take();
      System.out.println("Consumer recd - " + i);
    } catch (InterruptedException e) {
      System.out.println("Error " + e.getMessage());
    }
  }
    
  public void put(int i){
    this.i = i;
    try {
      // putting in blocking queue
      linkedBlockingQ.put(i);
      System.out.println("Putting - " + i);
    } catch (InterruptedException e) {
      System.out.println("Error " + e.getMessage());
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

Here it can be seen how put and take methods are working simultaneously and the respective method blocks, in the case of put() it blocks if space is full (size is 1 only) and take() method blocks unless there is any element to retrieve.

Compare this implementation with [producer-counsumer implementation using wait, notify, notifyall](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) where you need to do a lot of work yourself for inter-thread communication or with [producer-counsumer implementation using semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html) and you will see how much simplified it is using a BlockingQueue implementation.

That's all for this topic **Java LinkedBlockingQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!