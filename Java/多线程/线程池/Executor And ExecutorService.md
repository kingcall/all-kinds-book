### Executor And ExecutorService in Java With Examples

This post gives an overview of Java Executors framework which comprises-

1. **Executor interfaces**- Executor, ExecutorService and ScheduledExecutorService interfaces which define the three executor object types.
2. **Thread pools**- Executor implementation classes like [ThreadPoolExecutor](https://www.netjstech.com/2018/11/threadpoolexecutor-java-thread-pooling-example-executorservice.html) and [ScheduledThreadPoolExecutor](https://www.netjstech.com/2018/11/java-scheduledthreadpoolexecutor-task-scheduling-executor.html) which use thread pools.
3. **Fork/Join**- A framework (from JDK 7) for taking advantage of multiple processors.





### Thread management through executors in Java

In large-scale applications, its good to separate thread management and creation from the rest of the application. The concurrent API in Java has a feature called **executors** that provides an alternative to managing threads through the Thread class.

### Executor interface in Java

At the core of the Java executors framework is the **Executor interface**, which also has two sub interfaces **ExecutorService** and **ScheduledExecutorService**.

An object of type Executor can execute runnable tasks. An Executor is normally used instead of explicitly creating threads. For example If r is a Runnable object, and e is an Executor object you can replace

```
(new Thread(r)).start();
```

with

```
e.execute(r);
```

### Methods in Java Executor interface

The Executor interface in Java provides a single method execute().

**void execute(Runnable command)**- Executes the given command at some time in the future. The command may execute in a new thread, in a pooled thread, or in the calling thread, at the discretion of the Executor implementation.

### Java ExecutorService interface

ExecutorService interface in Java extends Executor interface and provides methods to **manage termination** (through shutdown() method) and methods that can **produce a Future** (using submit() with a Callable) for tracking progress of one or more asynchronous tasks.

**ExecutorService** interface has more versatile **submit** method. Like execute, submit accepts Runnable objects, but *also accepts Callable objects*, which allows the task to return a value. The *submit method returns a Future object*, which is used to retrieve the Callable return value and to manage the status of both Callable and Runnable tasks.

### ExecutorService Implementing classes

In the Java concurrency there are three pre defined executor classes that implement the Executor and ExecutorService interfaces.

1. **ThreadPoolExecutor**- Implements the Executor and ExecutorService interfaces and executes the submitted task using one of the pooled thread.
2. **ScheduledThreadPoolExecutor**- It extends ThreadPoolExecutor and also implements the ScheduledExecutorService interface. This class schedule commands to run after a given delay, or to execute periodically.
3. **ForkJoinPool** implements the Executor and ExecutorService interfaces and is used by the Fork/Join Framework.

Before writing any examples for Executor or ExecutorService, two things are worth knowing **Executors class** and **ThreadPools**.

### ThreadPools in Java

In a large scale application if each task uses its own thread then allocating and deallocating many thread objects creates a significant memory management overhead.

Thread pool as the name suggests provides a set of threads, any task which has to be executed get a thread from this pool.

### Executors class in Java

Executors class provides factory and utility methods for Executors framework classes like Executor, ExecutorService, ScheduledExecutorService, ThreadFactory, and Callable.

Though you can use **ThreadPoolExecutor** and **ScheduledThreadPoolExecutor** directly, but the best way to get an executor is to use one of the static factory methods provided by the Executors utility class.

Some of the factory methods provided by Java Executors class are-

1. **static ExecutorService newCachedThreadPool()**- Creates a thread pool that creates new threads as needed, but will reuse previously constructed threads when they are available.
2. **static ExecutorService newFixedThreadPool(int numThreads)**- Creates a thread pool that reuses a fixed number of threads.
3. **static ScheduledExecutorService newScheduledThreadPool(int numThreads)**- Creates a thread pool that can schedule commands to run after a given delay, or to execute periodically.
4. **newSingleThreadExecutor()**- Creates an Executor that uses a single worker thread operating off an unbounded queue.

### Java ExecutorService Examples

As already mentioned apart from **execute()** method, ExecutorService also has submit() method which is overloaded to take either Runnable or Callable as parameter. So let's see examples for these methods.

**Using execute method**

In this ExecutorService example a thread pool of two threads is created, and 6 runnable tasks are executed using execute() method. These 6 tasks will be executed using only these 2 threads from the thread pool, new thread won't be created for each of the 6 tasks. It can be verified from the thread name in the output.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorDemo {
 public static void main(String args[]) {
  // creating executor with pool of 2 threads
  ExecutorService ex = Executors.newFixedThreadPool(2);
  // running 6 tasks
  ex.execute(new Task());
  ex.execute(new Task());
  ex.execute(new Task());
  ex.execute(new Task());
  ex.execute(new Task());
  ex.execute(new Task());
  //shutting down the executor service
  ex.shutdown();
 }
}
 
/**
 *
 */
class Task implements Runnable{

 @Override
 public void run() {
  System.out.println("in run task for thread - " + Thread.currentThread().getName());
  // Introducing some delay for switching
  try {
   Thread.sleep(500);
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
 }
 
}
```

**Output**

```
in run task for thread - pool-1-thread-2
in run task for thread - pool-1-thread-1
in run task for thread - pool-1-thread-1
in run task for thread - pool-1-thread-2
in run task for thread - pool-1-thread-2
in run task for thread - pool-1-thread-1
```

**Submit with Runnable parameter example**

In this ExecutorService Java example submit method is used to submit a **runnable task** for execution which returns a Future representing that task. The Future's get() method will return null upon successful completion of a runnable task.

In the example 2 runnable tasks are submitted which return future objects, in case of Runnable, Future's get method will return null.

```
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ExecutorServiceDemo {
 public static void main(String args[]) {
  // creating executor with pool of 2 threads
  ExecutorService ex = Executors.newFixedThreadPool(2);
  // running tasks
  Future f1 = ex.submit(new Task());
  Future f2 = ex.submit(new Task());
  try {
   // getting the future value
   System.out.println("Future f1 " + f1.get());
   System.out.println("Future f1 " + f1.get());
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  } catch (ExecutionException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  ex.shutdown();
  
 }
}

/**
 * Runnable 
 */
class Task implements Runnable{

 @Override
 public void run(){
  System.out.println("in run task for thread - " + Thread.currentThread().getName());
  // Introducing some delay for switching
  try {
   Thread.sleep(500);
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   
   e.printStackTrace();
      
  }
 }
}
```

**Output**

```
in run task for thread - pool-1-thread-2
in run task for thread - pool-1-thread-1
Future f1 null
Future f1 null
```

**Submit with Callable as parameter example**

In this ExecutorService Java example callable task is submitted using submit() method.

Callable interface has call method which can return value too, so in this case when Future's get method is called it'll return a value. Note that here callable is implemented as a lambda expression.

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ExecutorServiceDemo {
  public static void main(String args[]) {
    // creating executor with pool of 2 threads
    ExecutorService ex = Executors.newFixedThreadPool(2);
    // Callable implemented as lambda
    Callable<String> c = ()->"Callable lambda is called";
    // running tasks with callable as param
    Future f1 = ex.submit(c);
    try {
      // getting the future value
      System.out.println("Future f1 " + f1.get());
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (ExecutionException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    ex.shutdown();      
  }
}
```

**Output**

```
Future f1 Callable lambda is called
```

- Refer [Callable and Future in Java concurrency](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html) to know more about Callable and Future in Java concurrency.

### ExecutorService shutdown

An ExecutorService can be shut down, which will cause it to reject new tasks. Two different methods are provided for shutting down an ExecutorService. The **shutdown()** method will allow previously submitted tasks to execute before terminating, while the **shutdownNow()** method prevents waiting tasks from starting and attempts to stop currently executing tasks. Upon termination, an executor has no tasks actively executing, no tasks awaiting execution, and no new tasks can be submitted. An unused ExecutorService should be shut down to allow reclamation of its resources.

### ScheduledExecutorService interface in Java

ScheduledExecutorService extends ExecutorService and provides methods that can schedule commands to run after a given delay, or to execute periodically.

It has methods that execute a Runnable or Callable task after a specified delay.

- **schedule(Callable<V> callable, long delay, TimeUnit unit)** - Creates and executes a ScheduledFuture that becomes enabled after the given delay.
- **schedule(Runnable command, long delay, TimeUnit unit)** - Creates and executes a one-shot action that becomes enabled after the given delay.

In addition, the interface defines scheduleAtFixedRate and scheduleWithFixedDelay, which executes specified tasks repeatedly, at defined intervals.

- **scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)** - Creates and executes a periodic action that becomes enabled first after the given initial delay, and subsequently with the given period; that is executions will commence after initialDelay then initialDelay+period, then initialDelay + 2 * period, and so on.
- **scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)** - Creates and executes a periodic action that becomes enabled first after the given initial delay, and subsequently with the given delay between the termination of one execution and the commencement of the next.

### ScheduledThreadPoolExecutor Java Example code

In this ScheduledExecutorService Java example callable task is submitted using schedule method which will be executed after a delay of 2 Sec.

```
import java.util.Date;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledFuture;
import java.util.concurrent.TimeUnit;

public class STEDemo {
  public static void main(String[] args) {
    ScheduledExecutorService scheduledExecutor = Executors.newScheduledThreadPool(2);
    // Callable implemented as lambda
    Callable<String> c = ()->{
      System.out.println("Time of execution- " + new Date());
      return "Callable lambda is called";
    };
    System.out.println("Time before execution- " + new Date());
    // scheduling tasks with callable as param
    // it will execute after a delay of 2 Secs
    ScheduledFuture<String> sf = scheduledExecutor.schedule(c, 2, TimeUnit.SECONDS); 
    try {
      System.out.println("Value- " + sf.get());
    } catch (InterruptedException | ExecutionException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
      scheduledExecutor.shutdown();
  }
}
```

**Output**

```
Time before execution- Wed Nov 21 11:37:15 IST 2018
Time of execution- Wed Nov 21 11:37:17 IST 2018
Value- Callable lambda is called
```

That's all for this topic **Executor And ExecutorService in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!