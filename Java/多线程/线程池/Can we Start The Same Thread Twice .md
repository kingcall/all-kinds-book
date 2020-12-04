### Can we Start The Same Thread Twice in Java

What if we start the same thread again in Java is one question you will be confronted with in many [Java Multi-threading interview questions](https://www.netjstech.com/2015/08/java-multi-threading-interview-questions.html). This post tries to answer this question whether you can start the same thread twice or not.

According to Java Docs- https://docs.oracle.com/javase/10/docs/api/java/lang/Thread.html#start() description of start method in the Thread class says -

- **public void start()**- *Causes this thread to begin execution; the Java Virtual Machine calls the run method of this thread.
  The result is that two threads are running concurrently: the current thread (which returns from the call to the start method) and the other thread (which executes its run method).
  It is never legal to start a thread more than once. In particular, a thread may not be restarted once it has completed execution.
  **Throws: IllegalThreadStateException**- If the thread was already started.*

Thus a Thread can only be started once and any attempt to start the same thread twice in Java will throw **IllegalThreadStateException**.

### Thread's transition to terminated State

As we saw in [Thread States in Java Multi-Threading](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) once a Java thread finishes execution it transitions to terminated (dead) state. Calling start() method on a terminated thread should not be permitted and that's why IllegalThreadStateException is thrown.

### Starting a thread twice in Java example

```
public class ThreadTwiceStart implements Runnable {
    public static void main(String[] args) {
        // creating a thread 
        Thread t = new Thread(new ThreadTwiceStart(), "MyThread");
        t.start();
        // starting the same thread again
        t.start();
    }

    @Override
    public void run() {
        System.out.println("Thread started running " + Thread.currentThread().getName());    
    }
}
```

**Output**

```
Thread started running MyThread
Exception in thread "main" java.lang.IllegalThreadStateException
 at java.lang.Thread.start(Unknown Source)
 at org.netjs.example.ThreadTwiceStart.main(ThreadTwiceStart.java:9)
```

It can be seen how it has thrown IllegalThreadStateException if an attempt is made to start the same thread again.

If we have to **start the same thread again** we have to create a new thread object. This code will run fine-

```
public class ThreadTwiceStart implements Runnable {
    public static void main(String[] args) throws InterruptedException{
        // creating a thread 
        Thread t = new Thread(new ThreadTwiceStart(), "MyThread");
        t.start();
        t.join();       
        t = new Thread(new ThreadTwiceStart(), "MyThread");        
        t.start();
    }

    @Override
    public void run() {
        System.out.println("Thread started running " + Thread.currentThread().getName());    
    }
}
```

Here a new thread object is created before calling the start method again thus this code will run without throwing any [exception](https://www.netjstech.com/2015/05/overview-of-java-exception-handling.html).

That's all for this topic **Can we Start The Same Thread Twice in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!