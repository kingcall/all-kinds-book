### CopyOnWriteArrayList in Java With Examples

This post talks about CopyOnWriteArrayList in Java which resides in **java.util.concurrent** package. CopyOnWriteArrayList is a thread safe implementation of the List interface.



### Synchronized List options in Java

Though we have an option to synchronize the collections like List or Set using **synchronizedList** or **synchronizedSet** methods respectively of the **Collections** class but there is a drawback to this [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html); very poor performance as the whole collection is locked and only a single thread can access it at a given time.

Java also has a [Vector](https://www.netjstech.com/2015/12/difference-between-arraylist-and-vector-java.html) class as a thread-safe alternative to List but that thread safety is achieved by synchronizing all the methods of the Vector class, which again results in poor performance.

### CopyOnWriteArrayList in Java

From Java 5 **CopyOnWriteArrayList** is introduced as a thread-safe variant of ArrayList. It is designed for concurrent access from multiple threads. CopyOnWriteArrayList in Java provides a thread-safe alternative for [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html), same way [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) provides a thread-safe alternative for [HashMap](https://www.netjstech.com/2016/02/difference-between-hashmap-and-concurrenthashmap-java.html) and [CopyOnWriteArraySet](https://www.netjstech.com/2016/03/copyonwritearrayset-in-java-concurrency.html) for [HashSet](https://www.netjstech.com/2015/11/how-to-sort-hashset-in-java.html).

### Thread safety in CopyOnWriteArrayList

CopyOnWriteArrayList in Java is *also an implementation* of the List interface but it is a **thread safe variant**. This **thread safety** is achieved by *making a fresh copy of the underlying array with every mutative operations* (add, set, and so on). It is evident from the name also "Copy on write"; *whenever value is changed create a copy*.

You may argue that this way of creating a fresh copy whenever any mutative operation is performed must be very costly. Yes it is, that is why using CopyOnWriteArrayList provides **better performance** in scenarios where there are more iterations of the list than mutations.

That brings us to the second point "**snapshot style**" iterator in CopyOnWriteArrayList.

### CopyOnWriteArrayList has a fail-safe iterator

Iterator returned by CopyOnWriteArrayList in Java is [fail-safe](https://www.netjstech.com/2015/05/fail-fast-vs-fail-safe-iterator-in-java.html), it uses a *reference to the state of the array at the point that the iterator was created*. You know by now any mutation will result in a fresh copy of the underlying array. Thus the array that the iterator has a reference to never changes during the lifetime of the iterator, so interference is impossible and the iterator is guaranteed not to throw **ConcurrentModificationException**.

The iterator will not **reflect additions, removals, or changes** to the list since the iterator was created thus it is also known as "snapshot style" iterator.

Element-changing operations on iterators themselves (remove, set, and add) are not supported. These methods throw **UnsupportedOperationException**.

Since iterator is not affected by the mutations thus multiple threads can iterate the collection without interference from one another or from threads wanting to modify the collection.

### Java CopyOnWriteArrayList constructors

- **CopyOnWriteArrayList()**- This constructor Creates an empty list.
- **CopyOnWriteArrayList(E[] toCopyIn)**- Creates a list holding a copy of the given array.
- **CopyOnWriteArrayList(Collection<? extends E> c)**- Creates a list containing the elements of the specified collection, in the order they are returned by the collection's iterator.

### CopyOnWriteArrayList Java Example

Let's see a simple Java example creating CopyOnWriteArrayList and adding elements to it.

```
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteALDemo {
  public static void main(String[] args) {
    List<String> numList = new CopyOnWriteArrayList<String>();
    numList.add("1");
    numList.add("2");
    numList.add("3");
    numList.add("4");
    // Displaying CopyOnWriteArrayList elements
    for(String num : numList){
      System.out.println("Number- " + num);
    }
  }
}
```

**Output**

```
Number- 1
Number- 2
Number- 3
Number- 4
```

### Java CopyOnWriteArrayList iterator Example

Let's see "snapshot style" iterator concept of CopyOnWriteArrayList in Java with an example.

First let's use ArrayList with 2 threads accessing it concurrently. One of the thread tries to structurally modified the ArrayList while second thread is iterating it. This should result in a ConcurrentModificationException.

```
public class FailFastDemo {
  public static void main(String[] args) {
    List<String> numList = new ArrayList<String>();
    numList.add("1");
    numList.add("2");
    numList.add("3");
    numList.add("4");
        
    //This thread will iterate the list
    Thread thread1 = new Thread(){ 
      public void run(){ 
        try{ 
          Iterator<String> i = numList.iterator(); 
          while (i.hasNext()){ 
            System.out.println(i.next()); 
            // Using sleep to simulate concurrency
            Thread.sleep(1000); 
          }     
        }catch(ConcurrentModificationException e){ 
          System.out.println("thread1 : Concurrent modification detected 
            on this list"); 
          e.printStackTrace();
        }catch(InterruptedException e){
          
        } 
      } 
    }; 
    thread1.start(); 
        
    // This thread will try to add to the collection,
    // while the collection is iterated by another thread.
    Thread thread2 = new Thread(){ 
      public void run(){ 
        try{ 
          // Using sleep to simulate concurrency
          Thread.sleep(2000);
          // adding new value to the shared list
          numList.add("5"); 
          System.out.println("new value added to the list"); 
        }catch(ConcurrentModificationException e){ 
          System.out.println("thread2 : Concurrent modification detected 
                  on the List"); 
        } catch(InterruptedException e){}
      } 
    }; 
    thread2.start(); 
  }
}
```

**Output**

```
1
2
new value added to the list
thread1 : Concurrent modification detected on this list
java.util.ConcurrentModificationException
 at java.util.ArrayList$Itr.checkForComodification(Unknown Source)
 at java.util.ArrayList$Itr.next(Unknown Source)
 at org.netjs.prog.FailFastDemo$1.run(FailFastDemo.java:24)
```

Here it can be seen that the ConcurrentModificationException is thrown because the list is changed by a thread while it has been iterated by another thread.

Now in the same code change the ArrayList to **CopyOnWriteArrayList**. Also added one sysout after adding new element to the list.

```
public class FailFastDemo {
  public static void main(String[] args) {
    List<String> numList = new CopyOnWriteArrayList<String>();
    numList.add("1");
    numList.add("2");
    numList.add("3");
    numList.add("4");
        
    //This thread will iterate the list
    Thread thread1 = new Thread(){ 
      public void run(){ 
        try{ 
          Iterator<String> i = numList.iterator(); 
          while (i.hasNext()){ 
            System.out.println(i.next()); 
            // Using sleep to simulate concurrency
            Thread.sleep(1000); 
          }     
        }catch(ConcurrentModificationException e){ 
          System.out.println("thread1 : Concurrent modification detected 
            on this list"); 
          e.printStackTrace();
        }catch(InterruptedException e){
                    
        } 
      } 
    }; 
    thread1.start(); 
        
    // This thread will try to add to the collection,
    // while the collection is iterated by another thread.
    Thread thread2 = new Thread(){ 
      public void run(){ 
        try{ 
          // Using sleep to simulate concurrency
          Thread.sleep(2000);
          // adding new value to the shared list
          numList.add("5"); 
          System.out.println("new value added to the list"); 
          System.out.println("List " + numList);
        }catch(ConcurrentModificationException e){ 
          System.out.println("thread2 : Concurrent modification detected 
           on the List"); 
        } catch(InterruptedException e){}
      } 
    }; 
    thread2.start();    
  }
}
```

**Output**

```
1
2
3
new value added to the list
List [1, 2, 3, 4, 5]
4
```

Here ConcurrentModificationException is not thrown as CopyOnWriteArrayList is used now. Also note that, though one of the thread adds a new element and at that time the list prints all the elements from 1-5. But the iterator has the reference to the old copy of the list and it prints from 1-4.

**Points to note**

- CopyOnWriteArrayList in Java provides a thread-safe alternative to the normal ArrayList.
- In CopyOnWriteArrayList thread safety is achieved in a different way from a thread safe collection like Vector. In CopyOnWriteArrayList fresh copy of the underlying array is created with every mutative operations (add, set, and so on).
- Because of this approach CopyOnWriteArrayList gives better performance in case there are more threads iterating the list than mutating it. Several threads can iterate CopyOnWriteArrayList concurrently.
- CopyOnWriteArrayList's iterator is fail-safe and guaranteed not to throw ConcurrentModificationException.
- CopyOnWriteArrayList's iterator uses a reference to the state of the array at the point that the iterator was created.
- The iterator will not reflect additions, removals, or changes to the list since the iterator was created thus it is also known as "snapshot style" iterator.
- All elements are permitted, including null.

That's all for this topic **CopyOnWriteArrayList in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!