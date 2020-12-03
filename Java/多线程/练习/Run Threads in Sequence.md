### How to Run Threads in Sequence in Java

How to ensure that threads run in sequence is a very popular [Java multi-threading interview question](https://www.netjstech.com/2015/08/java-multi-threading-interview-questions.html). Though it doesn’t make much sense practically to do that as you use threads so that processing can be done by many threads simultaneously. But you have to answer a question if asked in interview so this post tries to gives answer to that question "How to ensure threads run in sequence in Java".

So, if you are asked to answer the question “How can you make sure that Thread t1, t2 and t3 are executed in such a way that t2 starts after t1 finishes and t3 starts after executing t2”, you have to say, it can be done using [join() method in Java](https://www.netjstech.com/2015/06/isalive-and-join-method-in-java-multi.html).



### join() method in Java

join() method is used when you want to wait for the thread to finish. Its general form is–

```
public final void join() throws InterruptedException
```

This method waits until the thread on which it is called terminates.

As you see from the description of the **join()** method if it is called on any thread it will wait until the thread on which it is called terminates. Armed with this info let’s see the Java code to ensure that threads run in sequence.

### Executing threads in Sequence in Java

```
public class ThreadSequence {

 public static void main(String[] args) {
  SeqRun sr = new SeqRun();
  // Three threads
  Thread t1 = new Thread(sr);
  Thread t2 = new Thread(sr);
  Thread t3 = new Thread(sr);
  
  try {
   // First thread
   t1.start();
   t1.join();
   // Second thread
   t2.start();
   t2.join();
   // Third thread
   t3.start();
   t3.join();
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
}

class SeqRun implements Runnable{
 @Override
 public void run() {
  System.out.println("In run method " + Thread.currentThread().getName());
 } 
}
```

**Output**

```
In run method Thread-0
In run method Thread-1
In run method Thread-2
```

It can be seen that threads are executed in sequence here. Thing to do here is you [start the thread](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) and call the join() method on the same thread. This makes it to [wait](https://www.netjstech.com/2015/07/why-wait-notify-and-notifyall-methods-in-object-class-java-multi-threading.html) until the thread stops executing. That way order is ensured.

For testing you can also called sleep() method on the thread inside run(), you can observe that other threads don’t start their execution even if the current thread is sleeping.

```
@Override
public void run() {
 try {
  Thread.sleep(1000);
 } catch (InterruptedException e) {
  // TODO Auto-generated catch block
  e.printStackTrace();
 }
 System.out.println("In run method " + Thread.currentThread().getName()); 
} 
```