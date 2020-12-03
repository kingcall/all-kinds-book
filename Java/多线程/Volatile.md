### Volatile Keyword in Java With Examples

If volatile in Java has to be defined in simple terms, it can be defined as “*the volatile keyword in Java is an indication that the variable marked as volatile can change its value between different accesses*”. This post tries to explain this statement in more details along with Java volatile examples.

### Little background on memory

In order to understand volatile better we need to have a little background on how memory works, though its natural to think that a variable’s value will be stored some where in the memory (RAM) which is true also. The catch is, in order to boost performance processor **may store the value in its cache** (L1, L2, L3 cache, you might have read about it in your class or seen it in advertisements of laptops or desktops).

In that case any change to the value is written back to the main memory only when the [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) between the cache and the memory happens.

### Volatile keyword in Java

Now when we have some background on what actually happens with the variables and how main memory may be inconsistent with the current value of the variable in the processor’s cache, let’s see how volatile keyword in Java helps.

When a variable is declared as **volatile**, the compiler (even runtime) seeing the keyword volatile knows that this variable is shared. So the volatile variables are **not cached in registers** and read operation on a volatile variable always returns the most recent write by any [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html).

Let’s see it with an **example** - A multi-threaded application running on multi-processor may have different threads running on different processors. Also, note that shared field (a variable shared among many threads) will be stored in main memory, but as we know now, due to the caching by the processors even the shared field will have different copies in the cache of different processors. For example there is a shared int variable **var1** which is not declared as volatile, this scenario can be diagrammatically portrayed as below -

[![volatile in java](https://2.bp.blogspot.com/-8l8Mf8ZpqLk/WMrA2BYT5kI/AAAAAAAAAWg/Hb_ncCZGDdgr1pzrbxtjDJAvkalhM9rMACPcB/s320/volatile1.png)](https://2.bp.blogspot.com/-8l8Mf8ZpqLk/WMrA2BYT5kI/AAAAAAAAAWg/Hb_ncCZGDdgr1pzrbxtjDJAvkalhM9rMACPcB/s1600/volatile1.png)

Here you can see that when **thread-2** in **processor-2** tries to read the value of variable **var1** from main memory it gets a stale value, where as **thread-1** has already updated the value of variable **var1** but that updated value is still stored in the cache of **processor-1**.

In this type of scenario declaring variable var1 as **volatile** will ensure that any change to volatile variable var1 will be visible to other threads.

Thus using volatile variables reduces the risk of **memory consistency errors**, because any write to a volatile variable establishes a **happens-before relationship** with subsequent reads of that same variable. This means that changes to a volatile variable are always visible to other [threads](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html).

### What is Happens-before Order

Two actions can be ordered by a happens-before relationship. If one action happens-before another, then the first is visible to and ordered before the second.

### Java Volatile example code

The most common use for volatile variables in Java is to use it as a completion, interruption, or status flag. So let’s see an example where we have a status flag.

Here we have a class called **TestVolatile** which starts two threads, one of these threads prints “Printing Value“ five thousand times and then sets the flag as true. Where as the second thread is waiting for the flag to become true. When flag becomes true second thread prints “Job Done” once. Here note that the second thread loops continuously waiting for the first thread to complete its task and signal it to start, this is an example of [busy spinning in multi-threading](https://www.netjstech.com/2016/06/busy-spinning-in-multi-threading.html).

Also note that first time Runnable is implemented as an anonymous class, where as second time runnable is implemented as a [lambda expression](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html).

```
public class TestVolatile {
    private static  boolean flag = false;
    public static void main(String[] args) {
        // implemented as anonymous inner class
        new Thread(new Runnable(){
            
            @Override
            public void run() {
                for (int i = 1; i <= 5000; i++){
                    System.out.println("printing value " + i);
                }
                flag = true;
            }
            
        }).start();
        
        // Implemented as lambda expression
        new Thread(()-> {
            int i = 1;
            while (!flag) i++;
            System.out.println("Job Done " + i);    
        }).start();
    }
}
```

On executing this code you may get the first thread printing “printing value 1” to “printing value 5000” but after that second thread won’t start and the program will never terminate. Because the second thread never gets the updated value of flag and code execution goes into a [deadlock](https://www.netjstech.com/2015/07/deadlock-in-java-multi-threading.html). In this case having the boolean variable **flag** as volatile will help. That will guarantee that the change done to the shared variable by one thread is visible to other threads.

```
private static volatile boolean flag = false;
```

- Refer [Producer consumer Java Program using volatile](https://www.netjstech.com/2017/03/producer-consumer-java-program-using-volatile.html) to see how to write Producer consumer Java program using volatile.

### Happens-before extended guarantee

Since Java 5, happens-before does not only mean that a thread reading a volatile variable sees the latest change to the volatile variable, but also the side effects of the code that led up the change.

So if there are two threads **thread-1** and **thread-2** and **thread-1** changes any other variables (even non-volatile) before changing the **volatile variable**. Then, thread-2 when reading the volatile variable will also get the changed value of the other variables changed in thread-1 before updating the volatile variable.

**As example**, if there is a class **Shared** with 3 int variables

```
public class Shared {
    public int var1;
    public int var2;
    public volatile int var3;
}
```

Now let's say there is an object shared of the class Shared which is used by both thread-1 and thread-2, in **thread-1** you assign values to the variables.

```
shared.var1 = 1;
shared.var2 = 2;
shared.var3 = 3;
```

And in **thread-2** you print those variables

```
System.out.println("printing value " + shared.getVar1());
System.out.println("printing value " + shared.getVar2());
System.out.println("printing value " + shared.getVar3());
```

Here var3 being volatile ensures that all the actions of thread-1 prior to the write to the volatile variable var3 are guaranteed to be visible to the [thread](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) (thread-2 in this case) that has read the updated value of var3. Here it means that it is guaranteed by the Java memory model that the updated values of variables **var1** and **var2** will also be visible in thread-2 as they were updated before updating the value of volatile variable var3.

### Volatile won’t stop race condition

One of the important point you should remember about volatile in Java is that it **just guarantees visibility**, atomicity is **not guaranteed** by the volatile. Thus a code where shared resource is used by many threads and each thread is updating/reading the shared resource is not an ideal place to rely on volatile as it may lead to [race condition](https://www.netjstech.com/2015/06/race-condition-in-java-multi-threading.html).

**Example code**

Here we have an example where a shared class is used to increment and get a counter variable, counter variable is declared as volatile too. Then **6 threads** are spawned that increment and use the counter. Some delay is also introduced to simulate a production environment as you generally have many users accessing an application where one request is pre-empted to cater to other request.

```
public class VolatileDemo {

 public static void main(String[] args) {
  Data data = new Data();
  // Starting 6 threads
  ExecutorService ex = Executors.newFixedThreadPool(6);
  ex.execute(new VTask(data));
  ex.execute(new VTask(data));
  ex.execute(new VTask(data));
  ex.execute(new VTask(data));
  ex.execute(new VTask(data));
  ex.execute(new VTask(data));
  //shutting down the executor service
  ex.shutdown();
 }
}

// shared class
class Data{
 public volatile int counter = 0;

 public int getCounter() {
  return counter;
 }

 public void incrementCounter() {
  try {
       Thread.sleep(10);
  } catch (InterruptedException e) {
       // TODO Auto-generated catch block
       e.printStackTrace();
  }
  ++counter;
 }
}

// Thread 
class VTask implements Runnable{
 private Data data;
 public VTask(Data data){
  this.data = data;
 }
 
 @Override
 public void run() {
  System.out.println("Value for Thread " + Thread.currentThread().getName() + 
    " Before increment " + data.getCounter());
  data.incrementCounter();
  System.out.println("Value for Thread " + Thread.currentThread().getName() + 
    " After increment " + data.getCounter());
  
 }
}
```

**Output**

```
Value for Thread pool-1-thread-1 Before increment 0
Value for Thread pool-1-thread-3 Before increment 0
Value for Thread pool-1-thread-2 Before increment 0
Value for Thread pool-1-thread-4 Before increment 0
Value for Thread pool-1-thread-5 Before increment 0
Value for Thread pool-1-thread-6 Before increment 0
Value for Thread pool-1-thread-2 After increment 1
Value for Thread pool-1-thread-5 After increment 2
Value for Thread pool-1-thread-3 After increment 1
Value for Thread pool-1-thread-6 After increment 3
Value for Thread pool-1-thread-1 After increment 5
Value for Thread pool-1-thread-4 After increment 4 
```

As you see values are not as expected because every thread is updating the value not only reading it.

**Points to remember**

1. volatile keyword in Java can only be used with variables, not with methods and classes.
2. volatile variables are not cached and read operation on a volatile variable always returns the most recent write by any thread.
3. volatile variable reduces the risk of memory consistency errors, because any write to a volatile variable establishes a **happens-before relationship** with subsequent reads of that same variable.
4. From Java 5 Happens before guarantees that all the other variables updated before writing to the volatile variable by one thread are visible to other threads when reading the volatile variable.
5. volatile keyword in Java just guarantees visibility, atomicity is not guaranteed by the volatile.
6. volatile won't block threads as [synchronized](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) does.

That's all for this topic **Volatile Keyword in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!