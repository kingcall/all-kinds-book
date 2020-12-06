https://mp.weixin.qq.com/s/TWG9OIOMEdbtKrE_abVe5w

### Fail-Fast Vs Fail-Safe Iterator in Java

Difference between fail-fast and fail-safe iterator in Java, apart from being an important [Java Collections interview questions](https://www.netjstech.com/2015/11/java-collections-interview-questions.html), is a very important concept to know. The collections which are there from Java 1.2 (or even legacy) like [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html), [Vector](https://www.netjstech.com/2015/12/difference-between-arraylist-and-vector-java.html), [HashSet](https://www.netjstech.com/2015/09/how-hashset-works-internally-in-java.html) all have **fail-fast iterator** whereas Concurrent collections added in Java 1.5 like [ConcurrrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html), [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html), [CopyOnWriteArraySet](https://www.netjstech.com/2016/03/copyonwritearrayset-in-java-concurrency.html) have **fail-safe iterator**.

So first let us see the *key differences between the fail-fast and fail-safe iterator* in Java and then we'll see some Java programs to explain those features.

### Differences between fail-fast and fail-safe iterator

1. **fail-fast** iterator throws **ConcurrentModificationException** if the underlying collection is structurally modified in any way except through the iterator's own remove or add methods.
   **fail-safe** iterator doesn't throw ConcurrentModificationException.
2. **fail-fast** iterator in Java works on the original collection.
   **fail-safe** iterator in Java makes a copy of the underlying structure and iteration is done over that snapshot. Drawback of using a copy of the collection rather than original collection is that the iterator will not reflect additions, removals, or changes to the collection since the iterator was created.
3. **fail-fast** iterator provides remove, set, and add operations. Note that not all the iterators support all these methods. As exp. [ListIterator](https://www.netjstech.com/2015/08/list-iterator-in-java.html) supports **add()** method but the general iterator doesn't.
   With **fail-safe** iterator element-changing operations on iterators themselves (remove, set, and add) are not supported. These methods throw **UnsupportedOperationException**.

Now let us see some detailed explanation and supporting programs to see these features of both fail-fast and fail-safe iterators.

### fail-fast iterator in Java

An iterator is considered **fail-fast** if it throws a **ConcurrentModificationException** under either of the following two conditions:

- In **multi-threaded environment**, if one thread is trying to modify a Collection while another thread is iterating over it.
- Even **with single thread**, if a thread modifies a collection directly while it is iterating over the collection with a fail-fast iterator, the iterator will throw this exception.

**fail-fast iterator** fails if the underlying collection is **structurally modified** at any time after the iterator is created. The iterator will **throw a ConcurrentModificationException** if the underlying collection is structurally modified in any way *except through the iterator's own remove or add* (if applicable as in ListIterator) methods.

### What is termed as structural modification

Note that structural modification is any operation that adds or deletes one or more elements; merely setting the value of an element (in case of list) or changing the value associated with an existing key (in case of map) is not a structural modification.

Mostly iterators from **java.util** [package](https://www.netjstech.com/2016/07/package-in-java.html) throw **ConcurrentModificationException** if collection was modified by collection's methods (add / remove) while iterating

Also note that according to [Oracle Docs](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html) *fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw ConcurrentModificationException on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: the fail-fast behavior of iterators should be used only to detect bugs*.

### Java fail-fast iterator example

Here is a Java example of fail-fast iterator with an attempt to add new value to a map while the map is being iterated

```
public class FailFastModification {
  public static void main(String[] args) {
    // creating map
    Map <String,String> cityMap = new HashMap<String,String>();
    cityMap.put("1","New York City" );
    cityMap.put("2", "New Delhi");
    cityMap.put("3", "Newark");
    cityMap.put("4", "Newport");
    // getting iterator
    Iterator <String> itr = cityMap.keySet().iterator();
    while (itr.hasNext()){
      System.out.println(cityMap.get(itr.next()));
      // trying to add new value to a map while iterating it
      cityMap.put("5", "New City");
    }        
  }
}
```

**Output**

```
New York City
Exception in thread "main" java.util.ConcurrentModificationException
 at java.util.HashMap$HashIterator.nextNode(Unknown Source)
 at java.util.HashMap$KeyIterator.next(Unknown Source)
 at org.netjs.examples.FailFastModification.main(FailFastModification.java:20)
```

Though **we can update the underlying collection** as that is not structural modification, let's see **an example** with the same hash map used above-

```
public class FailFastModification {
  public static void main(String[] args) {
    // creating map
    Map <String,String> cityMap = new HashMap<String,String>();
    cityMap.put("1","New York City" );
    cityMap.put("2", "New Delhi");
    cityMap.put("3", "Newark");
    cityMap.put("4", "Newport");
    // getting iterator
    Iterator <String> itr = cityMap.keySet().iterator();
    while (itr.hasNext()){
      System.out.println(cityMap.get(itr.next()));
      // updating existing value while iterating
      cityMap.put("3", "New City");
    }        
  }
}
```

Here I have changed the value for the key "3", which is reflected in the output and no exception is thrown.

**Output**

```
New York City
New Delhi
New City
Newport
```

Using **iterator remove method** you can remove the values, that is permitted.

```
public class FailFastModification {
  public static void main(String[] args) {
    Map <String,String> cityMap = new HashMap<String,String>();
    cityMap.put("1","New York City" );
    cityMap.put("2", "New Delhi");
    cityMap.put("3", "Newark");
    cityMap.put("4", "Newport");
    System.out.println("size before iteration " + cityMap.size());
    Iterator <String> itr = cityMap.keySet().iterator();
    while (itr.hasNext()){
      System.out.println(cityMap.get(itr.next()));
      // removing value using iterator remove method
      itr.remove();
    }
    System.out.println("size after iteration " + cityMap.size());        
  }
}
```

**Output**

```
size before iteration 4
New York City
New Delhi
Newark
Newport
size after iteration 0
```

Here after iteration the value is removed using the remove method of the iteartor, thus the size becomes zero after the iteration is done.

### Fail-fast iterator Java example with multiple threads

Let’s see a multi-threaded example, where [concurrency](https://www.netjstech.com/2016/05/java-concurrency-interview-questions.html) is simulated using [sleep](https://www.netjstech.com/2015/07/difference-between-sleep-and-wait-java-threading.html) method.

In this example there are two threads one [thread](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) will [iterate the map](https://www.netjstech.com/2015/05/how-to-loop-iterate-hash-map-in-java.html) and print the values where as the second thread will try to remove the element from the same map.

```
public class FailFastTest {
 public static void main(String[] args) { 
  final Map<String,String> cityMap = new HashMap<String,String>();
  cityMap.put("1","New York City" );
  cityMap.put("2", "New Delhi");
  cityMap.put("3", "Newark");
  cityMap.put("4", "Newport");
  //This thread will print the map values
  // Thread1 starts 
  Thread thread1 = new Thread(){ 
   public void run(){ 
    try{ 
     Iterator<String> i = cityMap.keySet().iterator(); 
     while (i.hasNext()){ 
      System.out.println(i.next()); 
      // Using sleep to simulate concurrency
      Thread.sleep(1000); 
     }  
    }catch(ConcurrentModificationException e){ 
     System.out.println("thread1 : Concurrent modification detected on this map"); 
    }catch(InterruptedException e){
     
    } 
   } 
  }; 
  thread1.start(); 
  // Thread1 ends
   // This thread will try to remove value from the collection,
  // while the collection is iterated by another thread.
  // thread2 starts
  Thread thread2 = new Thread(){ 
   public void run(){ 
     try{ 
    // Using sleep to simulate concurrency
      Thread.sleep(2000);
      // removing value from the map
      cityMap.remove("2"); 
      System.out.println("city with key 2 removed successfully"); 
     }catch(ConcurrentModificationException e){ 
      System.out.println("thread2 : Concurrent modification detected on this map"); 
     } catch(InterruptedException e){}
    } 
  }; 
  thread2.start(); 
// thread2 end
 } // main end
} // class end
```

**Output**

```
1
2
city with key 2 removed successfully
thread1 : Concurrent modification detected on this map
```

It can be seen that in thread 1 which is iterating the map, Concurrent modification exception is thrown.

### Fail Safe iterator in Java

In case of **fail-safe iterator, ConcurrentModificationException** is not thrown as the fail-safe iterator makes a copy of the underlying structure and iteration is done over that snapshot.
Since iteration is **done over a copy of the collection so interference is impossible** and the iterator is guaranteed not to throw ConcurrentModificationException.

Drawback of using a copy of the collection rather than original collection is that the iterator may not reflect additions, removals, or changes to the collection since the iterator was created. Element-changing operations on iterators themselves (remove, set, and add) are not supported. These methods throw **UnsupportedOperationException**.

Iterator of [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) is an example of fail-safe Iterator in Java, also iterator provided by [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) keySet is **fail-safe** and never throws ConcurrentModificationException.

### Java fail-safe iterator example

```
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class FailSafeTest {
  public static void main(String[] args) {
    List  cityList = new CopyOnWriteArrayList();
    cityList.add("New York City");
    cityList.add("New Delhi");
    cityList.add("Newark");
    cityList.add("Newport");  
    Iterator itr = cityList.iterator();
    boolean flag = false;
    while (itr.hasNext()){         
      System.out.println(itr.next());
      // add a new value into the list
      if(!flag){
        cityList.add("NewCity");
        flag = true;
      }
      //itr.remove();
    }
    System.out.println("After addition -- ");
    itr = cityList.iterator();
    while (itr.hasNext()){         
      System.out.println(itr.next());
    }
  }
}
```

**Output**

```
New York City
New Delhi
Newark
Newport
After addition -- 
New York City
New Delhi
Newark
Newport
NewCity
```

This program **won't throw ConcurrentModificationException** as iterator used with CopyOnWriteArrayList is fail-safe iterator.

If we uncomment the line **//itr.remove()**; this program will throw UnsupportedOperationException as **fail-safe iterator does not support remove, set, and add operations**.

### Java fail-safe iterator with multiple threads example

In this example there are two threads one thread will iterate the ConcurrentHashMap and print the values where as the second thread will try to remove the element from the same ConcurrentHashMap. Since the iterator returned by ConcurrentHashMap is fail-safe so ConcurrentModificationException is not thrown.

```
public class FailSafeTest {
  public static void main(String[] args) { 
    final Map<String,String> cityMap = new ConcurrentHashMap<String,String>();
    cityMap.put("1","New York City" );
    cityMap.put("2", "New Delhi");
    cityMap.put("3", "Newark");
    cityMap.put("4", "Newport");
    //This thread will print the map values
    // Thread1 starts 
    Thread thread1 = new Thread(){ 
      public void run(){ 
        Iterator<String> i = cityMap.keySet().iterator(); 
        while (i.hasNext()){ 
          System.out.println(i.next()); 
          // Using sleep to simulate concurrency
          try {
            Thread.sleep(1000);
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          } 
        }  
      } 
    }; 
    thread1.start(); 
    // Thread1 ends
     // This thread will try to remove value from the collection,
    // while the collection is iterated by another thread.
    // thread2 starts
    Thread thread2 = new Thread(){ 
      public void run(){ 
        // Using sleep to simulate concurrency
        try {
          Thread.sleep(2000);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
        // removing value from the map
        cityMap.remove("2"); 
        System.out.println("city with key 2 removed successfully"); 
      } 
    }; 
    thread2.start(); 
    // thread2 end
  } // main end
}
```

**Output**

```
1
2
city with key 2 removed successfully
3
4
```

Note that iterators returned by concurrent Collection like ConcurrentHashMap provide weakly consistent traversal rather than fast-fail traversal which are guaranteed to traverse elements as they existed upon construction exactly once, and may (but are not guaranteed to) reflect any modifications subsequent to construction.

**Points to note-**

- An iterator is considered fail-fast if it throws a ConcurrentModificationException in case the underlying collection's structure is modified.
- While iterating a list or a map values can be updated, only if an attempt is made to add or remove from the collection ConcurrentModificationException will be thrown by fail-fast iterator.
- Fail-fast iterators throw ConcurrentModificationException on a best-effort basis and fail-fast behavior of an iterator cannot be guaranteed.
- Fail-safe iterator works with a copy of the collection rather than the original collection thus interference is impossible and the iterator is guaranteed not to throw ConcurrentModificationException.
- remove, set, and add operations are not supported in fail-safe iterator.

That's all for this topic **Fail-Fast Vs Fail-Safe Iterator in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!