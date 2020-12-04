### Java BlockingQueue With Examples

BlockingQueue [interface](https://www.netjstech.com/2015/05/interface-in-java.html) in Java is added in Java 5 with in the **java.util.concurrent** package along with other concurrent utilities like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html), [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [ReentranctLock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html) etc.

### Java BlockingQueue

**BlockingQueue** in Java, as the name suggests *is a queue that can block the operations*. Which means BlockingQueue supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available in the queue when storing an element.

If we take the Producer-Consumer problem as example where we have two threads, one inserting the elements in the queue and another thread retrieving the elements from the queue then using the BlockingQueue can block the operation in the following scenarios-

- If Producer thread is trying to insert an element when the queue is already full.
- If Consumer thread is trying to retrieve an element when the queue is empty.

For that BlockingQueue interface has two specific methods-

1. **put(E e)**- Inserts the specified element into this queue, waiting if necessary for space to become available.
2. **take()**- Retrieves and removes the head of this queue, waiting if necessary until an element becomes available.

### Java BlockingQueue Methods

BlockingQueue methods for inserting, removing and examining elements come in four forms. These set of methods have different functionality if the operationcan not be carried out when requested.

|             | Throws exception | Special value | Blocks         | Times out            |
| ----------- | ---------------- | ------------- | -------------- | -------------------- |
| **Insert**  | add(e)           | offer(e)      | put(e)         | offer(e, time, unit) |
| **Remove**  | remove()         | poll()        | take()         | poll(time, unit)     |
| **Examine** | element()        | peek()        | not applicable | not applicable       |

1. Methods in first column throw exception if the operation cannot be executed immediately i.e. these methods won't block.
2. Methods in second column return a special value (either null or false, depending on the operation) if operation cannot be performed immediately.
3. Methods in third column will block the current thread indefinitely until the operation can succeed.
4. Methods in fourth column block for only a given maximum time limit before giving up.

### BlockingQueue Superinterfaces

BlockingQueue extends Collection, Queue and Iterable interfaces so it inherits all Collection and Queue methods.

As example **add(E e)**, **remove(Object o)** from the Collection interface which are different from the other two methods **put()** and **take()** in the way that add() and remove() don't block, they throw exception if the operation cannot be executed immediately.

**poll()** and **peek(**) operations from Queue interface where

- **poll()**- Retrieves and removes the head of this queue, or returns null if this queue is empty.
- **peek()**- Retrieves, but does not remove, the head of this queue, or returns null if this queue is empty.

### BlockingQueue Implementations in Java

Classes implementing BlockingQueue interface in Java are-

- [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html)
- [DelayQueue](https://www.netjstech.com/2016/03/delayqueue-in-java.html)
- [LinkedBlockingDeque](https://www.netjstech.com/2016/04/linkedblockingdeque-in-java.html)
- [LinkedBlockingQueue](https://www.netjstech.com/2016/03/linkedblockingqueue-in-java.html)
- [LinkedTransferQueue](https://www.netjstech.com/2016/04/linkedtransferqueue-in-java.html)
- [PriorityBlockingQueue](https://www.netjstech.com/2016/03/priorityblockingqueue-in-java.html)
- [SynchronousQueue](https://www.netjstech.com/2016/03/synchronousqueue-in-java.html)



### Features of the Java BlockingQueue

Some of the important features of the BlockingQueue in Java are as follows which are shared by all the implementations.

**1. No nulls in BlockingQueue**

A **BlockingQueue** *does not accept null elements*. Implementations (like LinkedBlockingQueue or ArrayBlockingQueue) [throw](https://www.netjstech.com/2015/05/throw-statement-in-java-exception-handling.html) **NullPointerException** on attempts to add, put or offer a null.

```
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BQDemo {
  public static void main(String[] args) {
    BlockingQueue<String> arrayBlockingQ = new ArrayBlockingQueue<String>(2);
    try {
      arrayBlockingQ.put(null);
    } catch (InterruptedException e) {
      System.out.println("Exception occurred" + e);
    }
  }
}
```

**Output**

```
Exception in thread "main" java.lang.NullPointerException
 at java.util.concurrent.ArrayBlockingQueue.checkNotNull(Unknown Source)
 at java.util.concurrent.ArrayBlockingQueue.put(Unknown Source)
 at org.netjs.prgrm.BQDemo.main(BQDemo.java:10)
```

**2. Java BlockingQueue implementations are thread-safe**

BlockingQueue implementations like ArrayBlockingQueue, LinkedBlockingQueue are thread-safe. All queuing methods use internal [locks](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html) or other forms of concurrency control to achieve their effects atomically.

Since BlockingQueue interface also extends Collection interface so it inherits operations from Collection interface also. However, the bulk Collection operations **addAll**, **containsAll**, **retainAll** and **removeAll** are not necessarily performed atomically unless specified otherwise in an implementation. So it is possible, for example, for addAll(c) to fail (throwing an exception) after adding only some of the elements in c.

**Java BlockingQueue capacity**

BlockingQueue implementations are of two types-

- **Bounded**- A bounded queue is a queue that has a fixed capacity.
- **Unbounded**- An unbounded queue is a queue that can grow indefinitely.

Note that there are BlockingQueue implementations that can be created as both bounded or unbounded.

For a bounded BlockingQueue implementation we have to create the BlockingQueue with the given (fixed) capacity.

**For example,** ArrayBlockingQueue for which capacity has to be specified.

```
BlockingQueue<String> arrayBlockingQ = new ArrayBlockingQueue<String>(2);
```

In case of LinkedBlockingQueue and PriorityBlockingQueue both can be bounded or unbounded.

```
BlockingQueue<String> linkedBlockingQ = new LinkedBlockingQueue<String>(2);
        
Queue<String> linkedBlockingQ = new LinkedBlockingQueue<String>();
```

Note that a **BlockingQueue** without any intrinsic capacity constraints always reports a remaining capacity of **Integer.MAX_VALUE**.

### BlockingQueue Usage in Java

BlockingQueue implementations are designed to be used primarily for producer-consumer queues because of the blocking methods put() and take() which facilitates inter-thread communication.

It can also be used as a bounded buffer. Let's say you have a ArrayBlockingQueue of capacity 10. So one thread can keep putting values in it and another thread can read from it once the buffer is full thus creating a bounded buffer.

At any time if all 10 slots are filled put() will block and same way for take() if there are no elements to read it will block.

- Refer [Exchanger in Java](https://www.netjstech.com/2016/02/exchanger-in-java-concurrency.html) to see an elegant and simple way to create bounded buffer.

### BlockingQueue Java example

```
public static void main(String[] args) {
  BlockingQueue<String> arrayBlockingQ = new ArrayBlockingQueue<String>(2);
  try {
    arrayBlockingQ.put("A");
    arrayBlockingQ.put("B");
    System.out.println("------ 1 -------");
    arrayBlockingQ.forEach(a->System.out.println(a));
    arrayBlockingQ.take();
    arrayBlockingQ.put("C");
    System.out.println("------ 2 -------");
    
    arrayBlockingQ.forEach(a->System.out.println(a));
  } catch (InterruptedException e) {
    System.out.println("Exception occurred" + e);
  }
}
```

**Output**

```
------ 1 -------
A
B
------ 2 -------
B
C
```

Here it can be seen how elements are added at the end, while taking it is retrieved from the head of the queue.

**Reference:** https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/BlockingQueue.html

That's all for this topic **Java BlockingQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!