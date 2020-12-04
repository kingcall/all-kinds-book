### Java Exchanger With Examples

Exchanger in Java is one of the Synchronization class added along with other synchronization classes like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html), [Semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html) and [Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html) in **java.util.concurrent package**.

### How does Exchanger in Java work

Exchanger makes it easy for two threads to exchange data between themselves.

Exchanger provides a synchronization point at which two threads can pair and swap elements. **Exchanger** waits until two separate threads call its **exchange()** method. When two threads have called the **exchange()** method, Exchanger will swap the objects presented by the threads.

[![Exchanger in Java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:17:09-Exchanger%252Bin%252BJava.png)](https://2.bp.blogspot.com/-HCAdTvVmzvE/VrIgzjPdzNI/AAAAAAAAAP0/sOmo168cH6s/s1600/Exchanger%2Bin%2BJava.png)



**Exchanger in Java**



### Exchanger class in Java

Exchanger in Java is a generic class and is declared as

```
public class Exchanger<V>
```

Here **V** specifies the type of objects that may be exchanged.

### Java Exchanger constructor

Exchanger class has only one constructor.

- **Exchanger()**- Creates a new Exchanger.

### Exchange() class methods

Exchanger class has only one method **exchange()** overloaded to have two forms -

1. public V exchange(V x)throws InterruptedException

   Here

   **x**- the object to exchange.

   This method returns the object provided by the other thread.

   **exchange()** method [waits for another thread](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) to arrive at this exchange point (unless the current thread is interrupted), and then transfers the given object to it, receiving its object in return. If another thread is already waiting at the exchange point then it is resumed for thread scheduling purposes and receives the object passed in by the current thread. The current thread returns immediately, receiving the object passed to the exchange by that other thread.

   If no other thread is already waiting at the exchange then the current thread is disabled for thread scheduling purposes and lies dormant until one of two things happens:

   - Some other thread enters the exchange; or
   - Some other thread interrupts the current thread.

   If the current thread:

   - has its interrupted status set on entry to this method; or
   - is interrupted while waiting for the exchange,

   then InterruptedException is thrown and the current thread's interrupted status is cleared.

   

2. public V exchange(V x, long timeout, TimeUnit unit) throws InterruptedException, TimeoutException

   In the second form of exchange() method timeout period is specified so the thread will wait for the time specified. If the specified waiting time elapses then **TimeoutException** is thrown. If the time is less than or equal to zero, the method will not wait at all.

### Usage of Exchanger in Java

**Exchanger** can be used in a **Producer-Consumer** scenarios where one [thread](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) will produce the data and exchange it with the consumer thread. Consumer thread in turn will pass the empty buffer to the producer thread.

### Exchanger Java example code

Let's see an example where exchange() method will be called by two separate threads on the same Exchanger object. In this example there will be two threads, one thread will fill the buffer and another thread will exchange that with an empty buffer.

```
import java.util.concurrent.Exchanger;

public class ExchangerDemo {
  public static void main(String[] args) {
    Exchanger<String> ex = new Exchanger<String>();
    // Starting two threads
    new Thread(new ProducerThread(ex)).start();
    new Thread(new ConsumerThread(ex)).start();
  }
}

class ProducerThread implements Runnable {
  String str;
  Exchanger<String> ex;
  ProducerThread(Exchanger<String> ex){
    this.ex = ex;
    str = new String();
  }
  @Override
  public void run() {
    for(int i = 0; i < 3; i ++){
      str = "Producer" + i;
      try {
        // exchanging with an empty String 
        str = ex.exchange(str);
      } catch (InterruptedException e) {
        System.out.println(e);
      }
    }       
  }   
}

class ConsumerThread implements Runnable {
  String str;
  Exchanger<String> ex;
  ConsumerThread(Exchanger<String> ex){
    this.ex = ex;
  }
  @Override
  public void run() {
    for(int i = 0; i < 3; i ++){
      try {
        // Getting string from producer thread
        // giving empty string in return
        str = ex.exchange(new String());
        System.out.println("Got from Producer " + str);
      } catch (InterruptedException e) {
        System.out.println(e);
      }
    }        
  }   
}
```

**Output**

```
Got from Producer Producer0
Got from Producer Producer1
Got from Producer Producer2
```

Here we have two threads one each for **ProducerThread** and **ConsumerThread** class. Note that both of these threads are sharing the same Exchanger object.

ProducerThread is producing string objects in a loop which runs thrice. Here it is just a hardcoded string along with the loop count. So it should produce three strings Producer0, Producer1 and Producer2.
In the exchange() method of the Consumer thread, empty String is passed in exchange to the ProducerThread.

That's all for this topic **Java Exchanger With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!