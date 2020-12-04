### CopyOnWriteArraySet in Java With Examples

In Java 5 many concurrent collection classes are added as a thread safe alternative for their normal collection counterparts which are not thread safe. Like [ConcurrentHashMap](https://www.netjstech.com/2016/01/concurrenthashmap-in-java.html) as a thread safe alternative for [HashMap](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html), [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) as a thread safe alternative for [ArrayList](https://www.netjstech.com/2015/09/arraylist-in-java.html). Same way **CopyOnWriteArraySet in Java** is added as a thread safe alternative for [HashSet](https://www.netjstech.com/2015/09/how-hashset-works-internally-in-java.html) in Java.

### CopyOnWriteArraySet class in Java

CopyOnWriteArraySet implements the **Set** interface (Actually it extends the **AbstractSet** class which in turn implements the set interface). Since **CopyOnWriteArraySet** implements the Set [interface](https://www.netjstech.com/2015/05/interface-in-java.html) so basic functionality of the Set that only *unique elements can be added applies to CopyOnWriteArraySet too*. **CopyOnWriteArraySet** also has an add() method-

**add(E e)**- Adds the specified element to this set if it is not already present.

### CopyOnWriteArraySet is thread-safe

As already mentioned CopyOnWriteArraySet in Java is thread-safe. One thing to note here is that it is **backed by CopyOnWriteArrayList** which means *CopyOnWriteArraySet internally uses CopyOnWriteArrayList for all of its operations*.

Since it uses CopyOnWriteArrayList internally so thread-safety is achieved in the same way in CopyOnwriteArraySet as in CopyOnWriteArrayList; all *mutative operations* (add, set, and so on) are implemented by making a fresh copy of the underlying array.

Since fresh copy is made every time for a mutative operation so **CopyOnWriteArraySet** is more useful in the scenarios where set sizes generally stay small, read-only operations vastly outnumber mutative operations, and you need to prevent interference among threads during traversal.

As **example** when you have a unique set of values, which are read more by [threads](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) and operations like add or set are very rarely used, **CopyOnWriteArraySet** would be a good choice.

### Java CopyOnWriteArraySet constructors

CopyOnWriteArraySet class has two constructors.

- **CopyOnWriteArraySet()**- Creates an empty set.
- **CopyOnWriteArraySet(Collection<? extends E> c)**- Creates a set containing all of the elements of the specified collection.

### CopyOnWriteArraySet Java example

Here is a simple Java example showing the creation of CopyOnWriteArraySet and adding elements to it. This set is later iterated using for-each loop to display the set elements.

```
public class CWSetDemo {
  public static void main(String[] args) {
    Set<String> citySet = new CopyOnWriteArraySet<>();
    citySet.add("NYC");
    citySet.add("London");
    citySet.add("Prague");
    citySet.add("Berlin");
    citySet.add("London");
    // Iterating CopyOnWriteArraySet
    for(String city : citySet) {
      System.out.println("City name- " + city);
    }
  }
}
```

**Output**

```
City name- NYC
City name- London
City name- Prague
City name- Berlin
```

In the code you can see that the London is added twice but it is not added again as Set doesn't allow duplicates. If CopyOnWriteArraySet already contains the element, the call leaves the set unchanged and returns false.

### CopyOnWriteArraySet iterator

The iterator returned by CopyOnWriteArraySet is [fail-safe](https://www.netjstech.com/2015/05/fail-fast-vs-fail-safe-iterator-in-java.html) which means any **structural modification** (Structural modifications are those that change the size of the collection or otherwise perturb it in such a fashion that iterations in progress may yield incorrect results) made to the CopyOnwriteArraySet won't throw **ConcurrentModificationException**.

The iterator returned by concurrent collections like [CopyOnWriteArrayList](https://www.netjstech.com/2016/01/copyonwritearraylist-in-java.html) or CopyOnWriteArraySet uses the "**snap-shot**" style iterator in Java. The "snapshot" style iterator method uses a reference to the state of the array at the point that the iterator was created. This array never changes during the lifetime of the iterator, so interference is impossible and the iterator is guaranteed not to throw **ConcurrentModificationException**. The iterator will not reflect additions, removals, or changes to the list since the iterator was created. Note that in case of **CopyOnWriteArraySet** Iterators do not support the mutative remove operation. Any attempt to use iterator's remove() method will result in **UnsupportedOperationException**.

### Iterate a CopyOnWriteArraySet Java example

In this Java program we'll try to iterate a CopyOnWriteArraySet and also create a thread which tries to add new element to the set simultaneously.

```
public class CWSDemo {
  public static void main(String[] args) {
    Set<Integer> numSet = new CopyOnWriteArraySet<Integer>();
    // Adding 5 elements to the set
    for(int i=1;i<=5;i++) {
      numSet.add(i);
    }
    // Creating new thread
    new Thread(new Runnable(){
      @Override
      public void run() {
        try {
          // introducing some delay
          Thread.sleep(150);
        } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
        }
        // add new element to the set
        numSet.add(6);
        System.out.println("" + numSet);
      }    
    }).start();
        
    // get an iterator
    Iterator<Integer> itr = numSet.iterator();
    while(itr.hasNext()){
      Integer i = itr.next();
      try {
        Thread.sleep(30);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
      //itr.remove();
      System.out.println("" + i);
    }
  }
}
```

**Output**

```
1
2
3
4
[1, 2, 3, 4, 5, 6]
5
```

Here it can be seen that the thread adds a new element at the same time when the CopyOnWriteArraySet is iterated, still **ConcurrentModificationException** is not thrown. Since iterator get its own copy to iterate so the change is not reflected in that copy and the set still display elements 1-5 only.

If you uncomment the line itr.remove(); in the code it will throw **UnsupportedOperationException** when executed.

**Output**

```
Exception in thread "main" java.lang.UnsupportedOperationException at 
java.util.concurrent.CopyOnWriteArrayList$COWIterator.remove(Unknown Source) 
at org.netjs.prgrm.CWSDemo.main(CWSDemo.java:42)
[1, 2, 3, 4, 5, 6]
```

**Points to remember**

- CopyOnWriteArraySet is thread-safe.
- CopyOnWriteArraySet internally uses CopyOnWriteArrayList for all of its operations.
- CopyOnWriteArraySet is best suited for applications in which set sizes generally stay small and read-only operations outnumber the mutative operations.
- For mutative operations (add, set, remove, etc.) copy of an underlying array is made thus making mutative operations expensive.
- Iterator returned by CopyOnWriteArraySet is fail-safe and cannot encounter interference from other threads because iteration happens on a separate copy.

That's all for this topic **CopyOnWriteArraySet in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!