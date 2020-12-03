### Java PriorityBlockingQueue With Examples

PriorityBlockingQueue class in Java is an implementation of [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html). PriorityBlockingQueue class uses the same ordering rules as the PriorityQueue class, in fact PriorityBlockingQueue can be termed as the thread-safe alternative of the **PriorityQueue** as it has blocking retrieval operations.

### PriorityBlockingQueue is logically unbounded

While PriorityBlockingQueue is **logically unbounded**, attempted additions may fail due to resource exhaustion (causing **OutOfMemoryError**). That's where it differs from the other implementations of [BlockingQueue](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) like [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) (Always bounded) and [LinkedBlockingQueue](https://www.netjstech.com/2016/03/linkedblockingqueue-in-java.html) (both bounded and unbounded options).

As Example **put(E e)** method in ArrayBlockingQueue or LinkedBlockingQueue will wait if necessary for space to become available.

Whereas **put(E e)** method in PriorityBlockingQueue will never block as it is unbounded.

### Elements are sorted in Java PriorityBlockingQueue

The elements of the **PriorityBlockingQueue** are ordered according to their natural ordering, or by a [Comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html) provided at queue construction time, depending on which [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html) is used.

Note that a priority queue relying on natural ordering also does not permit insertion of non-comparable objects (doing so results in ClassCastException).

### Java PriorityBlockingQueue Constructors

- **PriorityBlockingQueue()**- Creates a PriorityBlockingQueue with the **default initial capacity** (11) that orders its elements according to their natural ordering.
- **PriorityBlockingQueue(Collection<? extends E> c)**- Creates a PriorityBlockingQueue containing the elements in the specified collection.
- **PriorityBlockingQueue(int initialCapacity)**- Creates a PriorityBlockingQueue with the specified initial capacity that orders its elements according to their natural ordering.
- **PriorityBlockingQueue(int initialCapacity, Comparator<? super E> comparator)**- Creates a PriorityBlockingQueue with the specified initial capacity that orders its elements according to the specified comparator.

### Nulls are not allowed in PriorityBlockingQueue

PriorityBlockingQueue in Java does not permit null elements. Trying to add null to the queue results in NullPointerException.

```
public class PriorityQDemo {
  public static void main(String[] args) {
    BlockingQueue<String> priortyBQ = new PriorityBlockingQueue<String>();
    priortyBQ.add("E");
    priortyBQ.add("B");
    priortyBQ.add("N");
    priortyBQ.add(null);
  }
}
```

**Output**

```
Exception in thread "main" java.lang.NullPointerException
 at java.util.concurrent.PriorityBlockingQueue.offer(PriorityBlockingQueue.java:479)
 at java.util.concurrent.PriorityBlockingQueue.add(PriorityBlockingQueue.java:463)
 at org.netjs.programs.PriorityQDemo.main(PriorityQDemo.java:13)
```

### PriorityBlockingQueue Java example

```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityQDemo {

  public static void main(String[] args) {
    String[] cityNames = {"Delhi", "Mumbai", "Chennai", "Bangalore", 
            "Hyderabad", "Lucknow"};
    // initializing PriortiyBlockingQueue
    BlockingQueue<String> priortyBQ = new PriorityBlockingQueue<String>();
    
    // Producer thread
    new Thread(){
      @Override
      public void run() {
        for(int i = 0; i < cityNames.length; i++){
          try {
            priortyBQ.put(cityNames[i]);
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
        for(int i = 0; i < cityNames.length; i++){
          try {
            System.out.println(" Consumer got - " + priortyBQ.take());
          } catch (InterruptedException e) {
            System.out.println("Error while retrieving value from the Queue " 
              + e.getMessage());
          }
        }
      }
    }.start();
  }
}
```

**Output**

```
Consumer got - Bangalore
Consumer got - Chennai
Consumer got - Delhi
Consumer got - Hyderabad
Consumer got - Lucknow
Consumer got - Mumbai
```

It can be seen here that the city names are ordered.

### PriorityBlockingQueue with Comparator Java Example

You can also use a [comparator](https://www.netjstech.com/2015/10/difference-between-comparable-and-comparator-java.html), if you want different ordering than the natural ordering. Here, if you want the cities in reverse order you can use the constructor that takes comparator as parameter.

```
import java.util.Comparator;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.PriorityBlockingQueue;

public class PriorityQDemo {

  public static void main(String[] args) {
    String[] cityNames = {"Delhi", "Mumbai", "Chennai", "Bangalore", 
              "Hyderabad", "Lucknow"};
    // initializing PriortiyBlockingQueue
    BlockingQueue<String> priortyBQ = new PriorityBlockingQueue<String>(10, new CityComparator());
    
    // Producer thread
    new Thread(){
      @Override
      public void run() {
        for(int i = 0; i < cityNames.length; i++){
          try {
            priortyBQ.put(cityNames[i]);
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
        for(int i = 0; i < cityNames.length; i++){
          try {
            System.out.println(" Consumer got - " + priortyBQ.take());
          } catch (InterruptedException e) {
            System.out.println("Error while retrieving value from the Queue " 
                + e.getMessage());
          }
        }
      }
    }.start();
  }
}

// Comparator class
class CityComparator implements Comparator<String>{
  @Override
  public int compare(String o1, String o2) {
    return o2.compareTo(o1);
  }    
}
```

**Output**

```
Consumer got - Mumbai
Consumer got - Lucknow
Consumer got - Hyderabad
Consumer got - Delhi
Consumer got - Chennai
Consumer got - Bangalore 
```

### PriorityBlockingQueue Iterator

Note that though elements are stored using natural ordering, [Iterator](https://www.netjstech.com/2015/08/list-iterator-in-java.html) provided in method iterator() is not guaranteed to traverse the elements of the PriorityBlockingQueue in any particular order. If you need ordered traversal, consider using Arrays.sort(pq.toArray()).

That's all for this topic **Java PriorityBlockingQueue With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!