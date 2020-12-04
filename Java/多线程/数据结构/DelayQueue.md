### Java DelayQueue With Examples

DelayQueue in Java is an implementation of [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) and is added in **Java 5** along with other concurrent utilities like [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html), [Exchanger](https://www.netjstech.com/2016/02/exchanger-in-java-concurrency.html), [ConcurentSkipListMap](https://www.netjstech.com/2016/03/concurrentskiplistmap-in-java.html), [CopyOnWriteArraySet](https://www.netjstech.com/2016/03/copyonwritearrayset-in-java-concurrency.html) etc.

DelayQueue in Java is an **unbounded** implementation of BlockingQueue, that's where it is different from the other implementations of BlockingQueue like [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) (Always bounded) and [LinkedBlockingQueue](https://www.netjstech.com/2016/03/linkedblockingqueue-in-java.html) (both bounded and unbounded options). Though it is similar to [PriorityBlockingQueue](https://www.netjstech.com/2016/03/priorityblockingqueue-in-java.html) in this behaviour as PriorityBlockingQueue is also unbounded.

### DelayQueue stores Delayed elements

**DelayQueue** in Java is a special implementation of BlockingQueue as it can only store elements of type **Delayed** and an *element can only be retrieved from DelayQueue when its delay has expired*.

Delayed [interface](https://www.netjstech.com/2015/05/interface-in-java.html) which defines the type for the elements in the DelayQueue has one method

- **getDelay(TimeUnit unit)**- Returns the remaining delay associated with this object, in the given time unit.

Delayed interface also extends [Comparable interface](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) so **compareTo(T o)** method should also be implemented. *This method implementation will decide whether you want to retrieve elements in ascending order of delay or descending*.

According to **JavaDocs** "*An implementation of this interface must define a compareTo method that provides an ordering consistent with its getDelay method.*"

So, just to sum it up; **DelayQueue** stores elements of type **Delayed**. When you implement **Delayed interface** two methods have to be implemented **getDelay(TimeUnit unit)** and **compareTo(T o)**.

### Ordering in Java DelayQueue

DelayQueue orders its elements (of type Delayed) based on the remaining delay associated with the element as returned by the **getDelay()** method. The head of the queue is the Delayed element whose delay expired furthest in the past where as the tail of the queue is the Delayed element with the longest remaining delay expiration time.

### Producer Consumer Java example using DelayQueue

Let's create a producer consumer using the DelayQueue. There is also a class DelayClass which implements Delayed interface and implements the required methods- getDelay() and compareTo(). DelayQueue will store objects of DelayClass.

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class DelayQDemo {
  public static void main(String[] args) {
    // delay of 2 seconds
    final long delay = 2000;
    BlockingQueue<DelayClass> delayQ = new DelayQueue<DelayClass>();
      
    // Producer thread
    new Thread(){
      @Override
      public void run() {
        for(int i = 0; i < 5; i++){
          try {
            // putting elements in delay queue
            delayQ.put(new DelayClass("item"+i, delay));
            Thread.sleep(50);
          } catch (InterruptedException e) {
            System.out.println("Error while putting values in the Queue "
             + e.getMessage());
          }
        }
      }
    }.start();
      
    // Consumer thread
    new Thread(){
      @Override
      public void run() {
        for(int i = 0; i < 5; i++){
          try {
            // retrieving elements from delay queue
            System.out.println(" Consumer got - " + delayQ.take());
            Thread.sleep(500);
          } catch (InterruptedException e) {
            System.out.println("Error while retrieving value from the Queue "
             + e.getMessage());
          }
        }
      }
    }.start();
  }
}

// Delayed interface implementing class
class DelayClass implements Delayed{
  private String item;
  private long expireTime;
  DelayClass(String item, long delay){
    this.item = item;
    // Expiretime is currenttime+delay, so if delay of 2 sec is required
    // expiration from queue will hppn after
    // currenttime + 2 sec
    this.expireTime = System.currentTimeMillis() + delay;
  }
    
  @Override
  public long getDelay(TimeUnit unit) {
    long diff = expireTime - System.currentTimeMillis();
    return unit.convert(diff, TimeUnit.MILLISECONDS);
  }
    
  @Override
  public int compareTo(Delayed o) {
    if(this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS)){
      return -1;
    }
    if(this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS)){
      return 1;
    }
    return 0;        
  }
    
  public String toString(){
    return "item = " + item + " expireTime = " + expireTime;
  } 
}
```

**Output**

```
Consumer got - item = item0 expireTime = 1458998017469
Consumer got - item = item1 expireTime = 1458998017531
Consumer got - item = item2 expireTime = 1458998017594
Consumer got - item = item3 expireTime = 1458998017656
Consumer got - item = item4 expireTime = 1458998017719
```

Here it can be seen elements are retrieved from the queue only after the delay expires.

**Points to remember**

1. DelayQueue in Java stores element of type Delayed.
2. Element is retrieved from DelayQueue only when its delay has expired.
3. The head of the queue is thatDelayed element whose delay expired furthest in the past.
4. If no delay has expired there is no head and poll will return null.
5. Expiration occurs when an element's getDelay(TimeUnit tu) method returns a value less than or equal to zero.

**Reference**: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/DelayQueue.html

That's all for this topic **Java DelayQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!