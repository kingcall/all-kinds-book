### Producer-Consumer Java Program Using volatile

Producer-consumer problem using multiple threads in Java is a frequently asked [Java multi-threading interview question](https://www.netjstech.com/2015/08/java-multi-threading-interview-questions.html). Though there are many ways to do it like-

- [Producer-Consumer Java program using wait notify](https://www.netjstech.com/2016/04/producer-consumer-java-program-using-wait-notify-multithreading.html)
- [Producer-Consumer Java program using ArrayBlockingQueue](https://www.netjstech.com/2016/04/producer-consumer-java-program-using-arrayblockingqueue.html)

But a simple way to write Producer-consumer Java program, if you are using one **writer thread** and one or more **reader thread**, is to use **volatile keyword**.

- Refer [Volatile in Java](https://www.netjstech.com/2017/03/volatile-in-java_20.html) to know more about volatile keyword.

### Producer consumer program in Java using volatile

Here logic is to use a **volatile boolean flag** which steers the logic and makes sure that value is put in the queue only after the previous value is consumed. Here two threads are created one calls the **produce method** and another **consume**. In consume method flag is assigned the value false, that is the trigger for the thread calling the produce method to come out of sleep and put another value.

In consume method [busy spinning thread](https://www.netjstech.com/2016/06/busy-spinning-in-multi-threading.html) is used which loops continuously until the condition is met.

First let’s see what will happen if **volatile keyword** is not used with the boolean variable flag. In that case it may happen that producer thread sets the flag as true but that value is **cached locally** and current value of the flag variable is not visible to another thread. That will result in consumer thread getting in an endless busy spinning causing [deadlock](https://www.netjstech.com/2015/07/deadlock-in-java-multi-threading.html).

**ProducerConsumer class**

```
public class ProducerConsumer {
  private int value = 0;
  private boolean flag = false;
  public void produce(Queue<Integer> sharedListObj) {
    // while flag is true put thread to sleep
    while (flag) {
      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
     
    sharedListObj.add(++value);
    System.out.println("Thread " + Thread.currentThread().getName() + 
    " putting " + value);
    flag = true;
  }
  public int consume(Queue<Integer> sharedListObj) {
    int j = 0;
    while (!flag) j++;    
    System.out.println("Getting from queue ");
    int value = sharedListObj.remove();
    flag = false;
    System.out.println("Thread " + Thread.currentThread().getName() + 
    " Consuming " + value);
    return value;
  }
}
```

**ProducerConsumerDemo**

```
import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumerDemo {
  public static void main(String[] args) {
    Queue<Integer> sharedListObj = new LinkedList<Integer>();
    ProducerConsumer producerConsumer = new ProducerConsumer();
    new Thread(new Runnable() {         
      @Override
      public void run() {
         for(int i = 0; i < 5; i++){
             producerConsumer.produce(sharedListObj);
         }
      }
    }, "ProducerThread").start();
         
    new Thread(()-> {
       for(int i = 0; i < 5; i++){
           producerConsumer.consume(sharedListObj);
       }

    }, "ConsumerThread").start();        
  }
}
```

**Output**

```
Thread ProducerThread putting 1
Getting from queue 
Thread ConsumerThread Consuming 1
Thread ProducerThread putting 2
```

When I execute it, I get a deadlock after producing and consuming the first value.

**Changing to volatile**

Changing flag variable to volatile does the trick, now it is ensured that value of flag won’t be cached locally. It is always read from the main memory.

```
public class ProducerConsumer {
  private int value = 0;
  private volatile boolean flag = false;
  public void produce(Queue<Integer> sharedListObj) {
    // while flag is true put thread to sleep
    while (flag) {
      try {
        Thread.sleep(500);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
         
    sharedListObj.add(++value);
    System.out.println("Thread " + Thread.currentThread().getName() + 
    " putting " + value);
    flag = true;
  }
  public int consume(Queue<Integer> sharedListObj) {
    int j = 0;
    while (!flag) j++;

    System.out.println("Getting from queue ");
    int value = sharedListObj.remove();
    flag = false;
    System.out.println("Thread " + Thread.currentThread().getName() + 
    " Consuming " + value);
    return value;
  }
}
```

**Output**

```
Thread ProducerThread putting 1
Getting from queue 
Thread ConsumerThread Consuming 1
Thread ProducerThread putting 2
Getting from queue 
Thread ConsumerThread Consuming 2
Thread ProducerThread putting 3
Getting from queue 
Thread ConsumerThread Consuming 3
Thread ProducerThread putting 4
Getting from queue 
Thread ConsumerThread Consuming 4
Thread ProducerThread putting 5
Getting from queue 
Thread ConsumerThread Consuming 5
```

That's all for this topic **Producer-Consumer Java Program Using volatile**. If you have any doubt or any suggestions to make please drop a comment. Thanks!