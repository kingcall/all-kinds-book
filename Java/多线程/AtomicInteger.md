### AtomicInteger in Java With Examples

AtomicInteger class in Java provides an int value that may be updated atomically. This class resides in the **java.util.concurrent.atomic** package which has classes that support *lock-free, thread-safe programming on single variables*. Apart from AtomicInteger some of the other classes are [AtomicLong](https://www.netjstech.com/2016/09/atomiclong-in-java-concurrency.html), **AtomicReference**, **DoubleAccumulator**.

### How AtomicInteger class in Java works

These atomic variable classes in [Java concurrency](https://www.netjstech.com/p/java-concurrency-tutorial.html) like AtomicInteger, AtomicLong uses [non-blocking algorithm](https://www.netjstech.com/2016/06/non-blocking-algorithms-in-java.html). These non-blocking algorithms use low-level atomic machine instructions such as [compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap) instead of locks to ensure data integrity under concurrent access.

Classes in this package **java.util.concurrent.atomic** provides methods that can *get, set or compare value as an atomic operation*. Atomic operations are performed as a single unit of task where all the operations succeed or none.There is no need to explicitly use any locking or [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) as AtomicInteger supports lock-free, thread-safe programming on single variables.

### Java AtomicInteger Constructors

AtomicInteger class in Java has two [constructors](https://www.netjstech.com/2015/04/constructor-in-java.html)-

- **AtomicInteger()**- Creates a new AtomicInteger with initial value 0.
- **AtomicInteger(int initialValue)**- Creates a new AtomicInteger with the given initial value.

### Atomic operations

AtomicInteger in Java provides atomic methods for getting, setting, incrementing and decrementing variables like **getAndIncrement()**, **getAndDecrement()**, **decrementAndGet()**, **getAndSet()** etc.

From the method names itself you can easily deduce that these are atomic methods as example if you take getAndIncrement() method which is doing three operatios.

- Gets the value
- Increment the value by 1
- Sets the updated value back

But these 3 operations are done as a single unit i.e. atomic operation. Either all 3 succeed or none.

There is also a **compareAndSet(int expect, int update)** method which atomically sets the value to the given updated value if the current value is equal to the expected value.

### Java AtomicInteger Example

In banking applications where you need to process several documents there is a very common requirement to provide sequence numbers for the processed documents.

In a multi-threaded environment it can easily be achieved by using **AtomicInteger** and the atomic operation **getAndIncrement()**.

```
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntDemo {

 public static void main(String[] args) {
  AtomicInteger ai = new AtomicInteger(0);
  new Thread(new IncThread(ai)).start();
  new Thread(new IncThread(ai)).start();
  new Thread(new IncThread(ai)).start();
 }
}

class IncThread implements Runnable{
 AtomicInteger ai = null;
 IncThread(AtomicInteger ai){
  this.ai = ai;
 }
 
 @Override
 public void run() {
  System.out.println("Value - " + ai.getAndIncrement() + " for " 
    + Thread.currentThread().getName()); 
 }
}
```

**Output**

```
Value - 0 for Thread-0
Value - 2 for Thread-2
Value - 1 for Thread-1
```

Note that output may differ in different runs.

That's all for this topic **AtomicInteger in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!