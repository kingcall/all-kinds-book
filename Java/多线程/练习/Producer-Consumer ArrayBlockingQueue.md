### Producer-Consumer Java Program Using ArrayBlockingQueue

This Java program solves the Producer-Consumer problem using threads and [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) which is an implementation of the [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html).

- Refer [Producer-Consumer Java Program Using wait notify](https://www.netjstech.com/2016/04/producer-consumer-java-program-using-wait-notify-multithreading.html) to see how to solve producer-consumer problem using wait notify.
- Refer [Producer-Consumer Java Program Using volatile](https://www.netjstech.com/2017/03/producer-consumer-java-program-using-volatile.html) to see how to solve producer-consumer problem using volatile keyword.

Initial capacity of the **ArrayBlockingQueue** will be kept one so that producer and consumer both get a chance alternatively.

Values will be inserted in the ArrayBlockingQueue using **put()** method, which will block if the space is full.

Values will be retrieved from the ArrayBlockingQueue using **take()** method, which retrieves and removes the head of this queue, waiting if necessary until an element becomes available.

In the program there is a class **Buffer** which is shared by both threads. In comparison to produce-consumer using wait notify this version using blocking queue is much simpler as you don't need to write the logic for making the thread wait or notifying the waiting thread.

### Producer-Consumer program in Java Using ArrayBlockingQueue

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

That's all for this topic **Producer-Consumer Java Program Using ArrayBlockingQueue**. If you have any doubt or any suggestions to make please drop a comment. Thanks!