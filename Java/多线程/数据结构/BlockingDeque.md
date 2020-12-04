### BlockingDeque in Java

BlockingDeque interface in Java (added in Java 6) is a **Deque** that provides additional support for **blocking operations**. Here blocking the operations means that the operation will wait for the deque to become non-empty (means deque has at least one element) when retrieving an element, and wait for space to become available in the deque when storing an element.

### Deque in Java

Deque is also an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) which was added in Java 6. Deque is short for "*double ended queue*". As the name suggests *Deque supports element insertion and removal at both ends*.

### Java BlockingDeque

Like any [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html), BlockingDeque in Java is thread safe, does not permit null elements, and may (or may not) be capacity-constrained.

[LinkedBlockingDeque class in Java](https://www.netjstech.com/2016/04/linkedblockingdeque-in-java.html) is the implementation of the BlockingDeque interface.

### BlockingDeque methods in Java

As already mentioned BlockingDeque in Java provides support for the **blocking operations**, but blocking methods come in **four forms**, categorized in the way these methods will handle operations that cannot be satisfied immediately, but may be satisfied at some point in the future:

- **Throw exception**- Methods falling in this category will throw exception if blocked.
- **Return special value**- This type of methods will return some value if need to wait, like false.
- **Blocks**- This type of methods will wait if necessary for space to become available.
- **Times out**- This type of methods will block for only a given maximum time limit before giving up.

These methods are summarized in the following table:

| **First Element (Head)** |                    |                 |                |                           |
| ------------------------ | ------------------ | --------------- | -------------- | ------------------------- |
|                          | *Throws exception* | *Special value* | *Blocks*       | *Times out*               |
| **Insert**               | addFirst(e)        | offerFirst(e)   | putFirst(e)    | offerFirst(e, time, unit) |
| **Remove**               | removeFirst()      | pollFirst()     | takeFirst()    | pollFirst(time, unit)     |
| **Examine**              | getFirst()         | peekFirst()     | not applicable | not applicable            |
| **Last Element (Tail)**  |                    |                 |                |                           |
|                          | *Throws exception* | *Special value* | *Blocks*       | *Times out*               |
| **Insert**               | addLast(e)         | offerLast(e)    | putLast(e)     | offerLast(e, time, unit)  |
| **Remove**               | removeLast()       | pollLast()      | takeLast()     | pollLast(time, unit)      |
| **Examine**              | getLast()          | peekLast()      | not applicable | not applicable            |

### BlockingDeque Java Example

Let's create a produce consumer bounded buffer using LinkedBlockingDeque which is an implmentation of BlockingDeque. Values will be inserted in the **LinkedBlockingDeque** using **put()** method, which will block if the space is full. Values will be retrieved from the **LinkedBlockingDeque** using **take()** method, which retrieves and removes the head of this queue, waiting if necessary until an element becomes available.

Here in the Produce class some delay is induced using **sleep()** method. You can see that in Consumer class, where it is taking elements out of the deque, no exception will be thrown but it will block. If you are using eclipse you can see that delay in the console when it is printing. Later we'll see the same example using deque which is not [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) safe there it will [throw](https://www.netjstech.com/2015/05/throw-statement-in-java-exception-handling.html) an exception.

```
import java.util.concurrent.BlockingDeque;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingDeque;

public class LinkedBlockingDQDemo {
    public static void main(String[] args) {
        SharedClass buffer = new SharedClass();
        // Starting two threads
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.execute(new Producer(buffer));
        executor.execute(new Consumer(buffer));
        executor.shutdown();
    }
}
/**
 * 
 */
class Producer implements Runnable{
    SharedClass buffer;
    Producer(SharedClass buffer){
        this.buffer = buffer;
    }
    @Override
    public void run() {
        for(int i = 0; i < 10; i++){
            buffer.put(i);
            if(i == 4){
                try {
                    // introducing some delay using sleep
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
    }
}

/**
 * 
 */
class Consumer implements Runnable{
    SharedClass buffer;
    Consumer(SharedClass buffer){
        this.buffer = buffer;
    }
    @Override
    public void run() {
        for(int i = 0; i < 10; i++){
            buffer.get();;
        }
    }    
}

//Shared class used by threads
class SharedClass{
    int i;
    // Bounded LinkedBlockingDeque of size 10
    BlockingDeque<Integer> linkedBlockingDeque = new LinkedBlockingDeque<Integer>(10);
    
    public void get(){
       try {
            // take method to get from blockingdeque
           System.out.println("Consumer recd - " + linkedBlockingDeque.take());
       } catch (InterruptedException e) {
            // TODO Auto-generated catch block
           e.printStackTrace();
       }
    }
    
    public void put(int i){
       this.i = i;
       try {
            // putting in blocking deque
            linkedBlockingDeque.put(i);
            System.out.println("Putting - " + i);
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
Putting - 1
Putting - 2
Putting - 3
Putting - 4
Consumer recd - 0
Consumer recd - 1
Consumer recd - 2
Consumer recd - 3
Consumer recd - 4
Putting - 5
Consumer recd - 5
Putting - 6
Consumer recd - 6
Putting - 7
Consumer recd - 7
Putting - 8
Consumer recd - 8
Putting - 9
Consumer recd - 9
```

Now let's write the same code as above using the **Deque** which is not thread safe. Here one change is instead of **LinkedBlockingDeque**, **ArrayDeque** is used which is implementation of **Deque** and also some delay is induced in consumer class too just to make sure that producer will start first. Then producer sleeps for 1000 MS, that is when exception is thrown.

```
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class DQDemo {
    public static void main(String[] args) {
        SharedClass buffer = new SharedClass();
        // Starting two threads
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.execute(new Producer(buffer));
        executor.execute(new Consumer(buffer));
        executor.shutdown();

    }
}

/**
 * 
 */
class Producer implements Runnable{
    SharedClass buffer;
    Producer(SharedClass buffer){
        this.buffer = buffer;
    }
    @Override
    public void run() {
        for(int i = 0; i < 10; i++){
            buffer.put(i);
            if(i == 4){
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        }
    }
}

/**
 * 
 */
class Consumer implements Runnable{
    SharedClass buffer;
    Consumer(SharedClass buffer){
        this.buffer = buffer;
    }
    @Override
    public void run() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        for(int i = 0; i < 10; i++){
            buffer.get();;
        }
    }    
}

//Shared class used by threads
class SharedClass{
    int i;
    // Bouded ArrayDeque of size 10
    Deque<Integer> arrayDq = new ArrayDeque<Integer>(10);
    public void get(){
        // retrieve from deque
        System.out.println("Consumer recd - " + arrayDq.pop());
    }
    
    public void put(int i){
        this.i = i;
        arrayDq.push(i);
        System.out.println("Putting - " + i);
    }
}
```

**Output**

```
Putting - 0
Putting - 1
Putting - 2
Putting - 3
Putting - 4
Consumer recd - 4
Consumer recd - 3
Consumer recd - 2
Consumer recd - 1
Consumer recd - 0
Exception in thread "pool-1-thread-2" java.util.NoSuchElementException
 at java.util.ArrayDeque.removeFirst(Unknown Source)
 at java.util.ArrayDeque.pop(Unknown Source)
 at org.netjs.prgrm.SharedClass.get(DQDemo.java:77)
 at org.netjs.prgrm.Consumer.run(DQDemo.java:65)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
 at java.lang.Thread.run(Unknown Source)
Putting - 5
Putting - 6
Putting - 7
Putting - 8
Putting - 9
```

**Reference:** https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/BlockingDeque.html

That's all for this topic **BlockingDeque in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!