### Callable and Future in Java With Examples

In this post we'll see what are Callable and Future in Java and how you can use them to return a result from an asynchronous computation.

**Table of contents**

1. [Callable in Java](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html#JavaCallable)
2. [Running a Callable task using ExecutorService](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html#CallableExecutorService)
3. [Future interface in Java](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html#FutureInterface)
4. [Java Example using Callable and Future](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html#CallableFutureExp)
5. [Callable as lambda expression](https://www.netjstech.com/2016/04/callable-and-future-in-java-concurrency.html#CallableAsLambda)



### Callable in Java

Callable, an [interface](https://www.netjstech.com/2015/05/interface-in-java.html), was added in Java 5. It allows you to define a task to be completed by a thread asynchronously.

You must be wondering, there is already a **Runnable** interface, with its run() method to do the same thing then why Callable interface in Java is required? Problem with Runnable is that it can't return a value. It offers a single method **run()** that accepts no arguments and returns no values, nor can it throw any [checked exceptions](https://www.netjstech.com/2015/05/difference-between-checked-unchecked-exception-java.html).

If you have to use Runnable in scenario where you want to return some value, you'll have to write a method outside the interface to get a value back from the completed task, Also you need some kind of notification to know that the task is completed and now value can be retrieved.

**Callable** in Java provides the functionality out of the box to implement the scenario as stated above. A callable task can return a result and may throw an exception.

- Refer [Difference Between Runnable And Callable in Java](https://www.netjstech.com/2016/08/difference-between-runnable-and-callable-java.html) for more differences between Runnable and Callable in Java.

**Callable interface**

Callable interface in Java has a single method **call()**, since it is a generic interface so it can return any value (Object, String, Integer etc.) based on how it is initialized.

Note that Callable is a [functional interface](https://www.netjstech.com/2015/06/functional-interfaces-and-lambda-expression-in-java-8.html) and can therefore be used as the assignment target for a [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) or [method reference](https://www.netjstech.com/2015/06/method-reference-in-java-8.html).

Callable interface in Java is defined as follows-

```
public interface Callable<V> {
  V call() throws Exception;
}
```

### Running a Callable task using ExecutorService

Like Runnable **you cannot pass** a Callable into a [thread](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html) to execute it, you use the [ExecutorService](https://www.netjstech.com/2016/04/executor-and-executorservice-in-java-concurrency.html), which has a **submit()** method which takes Callable as parameter to execute the Callable task.

```
<T> Future<T> submit(Callable<T> task)
```

Here task is the Callable task to submit and it returns a **Future** representing pending completion of the task.

Here is a simple example showing Java Callable and Future in action.

```
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<String> future = executor.submit(new Callable<String>() {
  public String call() {
    return "callable example";
  }
});
System.out.println("Value- " + future.get());
executor.shutdown();
```

**Output**

```
Value- callable example
```

### Future interface in Java

A Future represents the result of an asynchronous computation. When you submit a callable task using the **submit()** method of the ExecutorService, *Future object is returned*.

Future interface provides methods to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation.

- **get()**- get() method retrieves the result of the computation, *blocking if necessary* for the computation to complete.
- **get(long timeout, TimeUnit unit)**- If you don't want to get blocked indefinitely, you can use the version of the *get() method with the timeout parameter*, then it will wait only for the given time.
- **cancel(boolean mayInterruptIfRunning)**-Attempts to cancel execution of this task. This attempt will fail if the task has already completed, has already been cancelled, or could not be cancelled for some other reason. Returns false if the task could not be cancelled, typically because it has already completed normally; true otherwise.
- **isDone()**- To check if task is completed.
- **isCancelled()**- To check whether the task was cancelled before it was completed normally.

### Java Example using Callable and Future

In this example code there is a MyCallable class that implements the Callable interface and the return value of the call() method is String and its length.

A **thread pool** of four threads is created and submit() method of ExecutorService is used to submit callable tasks. Total submitted tasks are 6.
Also one tempMethod() is called just to show the asynchronous nature of the callable. While getting the future values isDone() method is used for the fourth get() to check whether that particular task is completed or not.

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableDemo {
  public static void main(String args[]){
    // ExecutorService with pool of 4 threads
    ExecutorService es = Executors.newFixedThreadPool(4);
    // Submitting 6 tasks
    Future<String> f1 = es.submit(new MyCallable("callable"));
    Future<String> f2 = es.submit(new MyCallable("future"));
    Future<String> f3 = es.submit(new MyCallable("executor"));
    Future<String> f4 = es.submit(new MyCallable("executor service"));
    Future<String> f5 = es.submit(new MyCallable("executors"));
    Future<String> f6 = es.submit(new MyCallable("scheduled executor"));
    // calling some other methods
    tempMethod();
        
    try {
      // Calling get() method to get the future value
      System.out.println("1. " + f1.get());
      System.out.println("2. " + f2.get());
      System.out.println("3. " + f3.get());
      while(!f4.isDone()){
        System.out.println("waiting for task to finish");
        Thread.sleep(10);
      }           
      System.out.println("4. " + f4.get());
      System.out.println("5. " + f5.get());
      System.out.println("6. " + f6.get());
    } catch (InterruptedException | ExecutionException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    //shutting down the executor service
    es.shutdown();
  }
    
  //
  public static void tempMethod(){
    System.out.println("I am in temp method");
  }
}

/**
 * Callable implementation
 */
class MyCallable implements Callable<String> {
  String str;
  MyCallable(String str){
    this.str = str;
  }
  @Override
  public String call() throws Exception {
    //System.out.println("In call method of Callable " + str);
    StringBuffer sb = new StringBuffer();
    return (sb.append("Length of string ")
              .append(str)
              .append(" is ")
              .append(str.length())).toString();
  }
}
```

**Output**

```
I am in temp method
1. Length of string callable is 8
2. Length of string future is 6
3. Length of string executor is 8
waiting for task to finish
4. Length of string executor service is 16
5. Length of string executors is 9
6. Length of string scheduled executor is 18
```

It can be seen here that after 3 calls to callable, **tempMethod()** is called in between while the Callable tasks are executed asynchronously. The output may differ when you execute the code.

### Callable as lambda expression

As already stated above that Callable is a [functional interface](https://www.netjstech.com/2015/06/functional-interface-annotation-java-8.html) so it can be implemented as a [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-examples-in-java-8.html) too. If we have to write the code as above using the lambda expression then it can be done this way -

```
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableLambda {
  public static void main(String args[]){
    ExecutorService es = Executors.newFixedThreadPool(4);
    getLength(es, "executor");
    getLength(es, "executor service");
    getLength(es, "Scheduled executor service");
    getLength(es, "executors");
    getLength(es, "fork join");
    getLength(es, "callable");       
  }
    
  public static void getLength(ExecutorService es, String str){
    Future<String> f = es.submit(() -> { return str + str.length();});
    try {
      System.out.println("" + f.get());
    } catch (InterruptedException | ExecutionException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
}
```

**Output**

```
executor8
executor service16
Scheduled executor service26
executors9
fork join9
callable8
```

In the code Callable is implemented as a lambda expression with in the sumbit() method of the ExecutorService.

That's all for this topic **Callable and Future in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!