### Java Semaphore With Examples

Semaphore is one of the synchronization aid provided by Java concurrency util in Java 5 along with other synchronization aids like [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html), [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html) and [Exchanger](https://www.netjstech.com/2016/02/exchanger-in-java-concurrency.html).

The **Semaphore** class present in **java.util.concurrent** package is a *counting semaphore* in which a semaphore, conceptually, maintains a set of permits.

Semaphore class in Java has two methods that make use of permits-

- **acquire()**- Acquires a permit from this semaphore, blocking until one is available, or the thread is interrupted. It has another overloaded version **acquire(int permits)**.
- **release()**- Releases a permit, returning it to the semaphore. It has another [overloaded method](https://www.netjstech.com/2015/04/method-overloading-in-java.html) **release(int permits)**.

Here be informed that *no actual permit objects are used*; the **Semaphore** just keeps a count of the number available and acts accordingly thus the name **Counting Semaphore**.

**Table of contents**

1. [How Semaphore works in Java](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#SemaphoreJava)
2. [Java Semaphore constructors](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#SemaphoreJavaConstructor)
3. [Binary Semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#BinarySemaphore)
4. [Semaphore Usage in Java](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#SemaphoreUsage)
5. [Java Semaphore example](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#SemaphoreExp)
6. [Inter-thread communication using Java semaphore](https://www.netjstech.com/2016/02/semaphore-in-java-concurrency.html#SemaphoreInterThread)



### How Semaphore works in Java

Thread that wants to access the shared resource tries to acquire a permit using **acquire()** method. At that time if the Semaphore's count is greater than zero *thread will acquire a permit* and Semaphore's count will be decremented by one.

If Semaphore's count is zero and thread calls acquire() method, then the thread will be blocked until a permit is available.

When thread is done with the shared resource access, it can call the **release()** method to release the permit. That results in the *Semaphore's count incremented by one*.

### Java Semaphore constructors

1. Semaphore(int permits)

   \- Creates a Semaphore with the given number of permits and

    

   nonfair

    

   fairness setting.

   If a Semaphore is initialized with 5 permits that means atmost 5 threads can call the acquire method without Calling release method.

2. Semaphore(int permits, boolean fair)

   \- Creates a Semaphore in Java with the given number of permits and the given fairness setting.

   When fairness is set to true, the semaphore guarantees that threads invoking any of the acquire methods are selected to obtain permits in the order in which their invocation of those methods was processed (first-in-first-out; FIFO).

### Binary Semaphore

A semaphore initialized to one, and which is used such that it only has at most one permit available, can serve as a mutual exclusion lock. This is more commonly known as a **binary semaphore**, because it only has **two states**: *one permit available, or zero permits available*. When used in this way, the binary semaphore has the property (unlike many Lock implementations), that the "lock" can be released by a thread other than the owner (as semaphores have no notion of ownership). This can be useful in some specialized contexts, such as deadlock recovery.

### Semaphore Usage in Java

- Semaphores are often used to restrict the number of threads than can access some (physical or logical) resource.
- Semaphore can also be used to facilitate inter-thread communication like in producer-consumer kind of scenario.

### Java Semaphore example

Let's see one example where Semaphore is used to control shared access. Here we have a shared counter and three threads using the same shared counter and trying to increment and then again decrement the count. So every [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html) should first increment the count to 1 and then again decrement it to 0.

```
import java.util.concurrent.Semaphore;

public class SemaphoreDemo {
  public static void main(String[] args) {
    Semaphore s = new Semaphore(1);
    SharedCounter counter = new SharedCounter(s);
    // Creating three threads
    Thread t1 = new Thread(counter, "Thread-A");
    Thread t2 = new Thread(counter, "Thread-B");
    Thread t3 = new Thread(counter, "Thread-C");
    t1.start();
    t2.start();
    t3.start();
  }
}

class SharedCounter implements Runnable{
  private int c = 0;
  private Semaphore s;
  SharedCounter(Semaphore s){
    this.s = s;
  }
  // incrementing the value
  public void increment() {
    try {
      // used sleep for context switching
      Thread.sleep(10);
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    c++;
  }
  // decrementing the value
  public void decrement() {    
    c--;
  }

  public int getValue() {
    return c;
  }
    
  @Override
  public void run() {
    try {
      // acquire method to get one permit
      s.acquire();
      this.increment();
      System.out.println("Value for Thread After increment - " + Thread.currentThread().getName() + " " + this.getValue());
      this.decrement();
      System.out.println("Value for Thread at last " + Thread.currentThread().getName() + " " + this.getValue());
      // releasing permit
      s.release();
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
Value for Thread After increment - Thread-A 1
Value for Thread at last Thread-A 0
Value for Thread After increment - Thread-B 1
Value for Thread at last Thread-B 0
Value for Thread After increment - Thread-C 1
Value for Thread at last Thread-C 0
```

In the code you can see that the access is controlled using Semaphore. Semaphore acquires and releases a permit after it is done manipulating the shared resource.

### Inter-thread communication using Java semaphore

Let's see another **example** where **producer-consumer** is implemented using semaphores. Idea is to have 2 semaphores when first is acquired release second, when second is acquired release first. That way shared resource has controlled access and there is [inter-thread communication](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) between the threads.

Remember unlike [RentrantLock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html), Semaphore *can be released by a thread other than the owner (as semaphores have no notion of ownership)*.

```
public class SemConProdDemo {
  public static void main(String[] args) {    
    Shared s = new Shared();
    // Producer and Consumer threads
    Thread t1 = new Thread(new SemProducer(s), "Producer");
    Thread t2 = new Thread(new SemConsumer(s), "Consumer");
    t1.start();
    t2.start();    
  }
}

// Shared class used by threads
class Shared{
  int i;
  // 2 semaphores 
  Semaphore sc = new Semaphore(0);
  Semaphore sp = new Semaphore(1);

  public void get(){
    try {
      // acquiring consumer semaphore
      sc.acquire();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    System.out.println("Got - " + i);
    // releasing producer semaphore
    sp.release();
  }

  public void put(int i){
    try {
      // acquiring producer semaphore
      sp.acquire();
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    this.i = i;
    System.out.println("Putting - " + i);
    // releasing consumer semaphore
    sc.release();
  }
}

// Producer thread
class SemProducer implements Runnable{
  Shared s;
  SemProducer(Shared s){
    this.s = s;
  }
  @Override
  public void run() {
    for(int i = 0; i < 5; i++){
      s.put(i);
    }
  }            
}

// Consumer thread
class SemConsumer implements Runnable{
  Shared s;
  SemConsumer(Shared s){
    this.s = s;
  }
  
  @Override
  public void run() {    
    for(int i = 0; i < 5; i++){
      s.get();                
    }
  }
}
```

**output**

```
Putting - 0
Got - 0
Putting - 1
Got - 1
Putting - 2
Got - 2
Putting - 3
Got - 3
Putting - 4
Got - 4
```

Check another program [Print odd-even numbers using threads and semaphore](https://www.netjstech.com/2016/04/print-odd-even-numbers-using-threads-semaphore.html) to see how to print odd-even numbers using threads and semaphore.

**Source**: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/Semaphore.html

That's all for this topic **Java Semaphore With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!