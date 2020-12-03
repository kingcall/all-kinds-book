### Thread States (Thread Life Cycle) in Java Multi-Threading

It is important to know the *lifecycle of a thread in Java and various states a Java thread can be in*. That will give an idea about what happens after [creating a thread in Java](https://www.netjstech.com/2015/06/creating-thread-in-java.html) and after calling the start() method on a thread. In this post we'll see various Thread states in Java multi-threading.

In order to begin execution of the thread's run() method you need to [call the start() method](https://www.netjstech.com/2015/06/what-if-run-method-called-directly-instead-of-start-java.html) on the thread. That is when Java Virtual Machine calls the run method of the thread.

Once scheduled to run by JVM a thread will run when its gets CPU cycle. A thread may be in waiting, blocked or running state after scheduled to run and later transitions to terminated state.

### Java thread states

Thread states in Java or the Java thread cycle is as follows.

**New state**

When a thread is created either by extending Thread class or implementing Runnable interface it is in "**New State**".

```
Thread thread1 = new ThreadDemoClass();
```

When a thread is in "New" state it is not yet scheduled to run.

**Runnable state**

When start() method is called on the thread object, that causes the thread to begin execution and it's the Java Virtual Machine that calls the **run()** method of the thread.

```
Thread thread1 = new ThreadDemoClass();
thread1.start();
```

This thread state is called **Runnable** as thread may not start running as soon as the start() method is called. It depends on the native OS when it schedules the thread to start running. A thread starts running when it gets the CPU cycle. A running thread may change state to **waiting**, **blocked**, **runnable** again and **terminated**.

**Blocked state**

When a resource is shared among various threads then a thread may go into blocked state as the resource may be used by another thread. In that case a thread has to suspend its execution because it is waiting to acquire a [lock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html).
**As example** in case of [synchronized block](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) where only one thread can enter that block of code.

```
synchronized (object reference) {   
  //code that uses shared resource 
}  
```

A thread in Java is also in blocked state when waiting for some IO to complete.

**Waiting state**

A thread that is waiting indefinitely for another thread to perform a particular action is in the waiting state.
**As example** A thread is waiting because of a call to [wait()](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) or join() method where no time of waiting is specified as parameter.

- Refer [isAlive() & join() methods in Java multithreading](https://www.netjstech.com/2015/06/isalive-and-join-method-in-java-multi.html) to know more about join() method in Java multi-threading.

**Timed_Waiting**

A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state.
**As exp.** A thread is waiting because of a call to **wait()** or **join()** method where time of waiting is specified as parameter. Calling **sleep(long millis)** will also result in a thread entering a TIMED_WAITING state.

**Example with Sleep**

```
Thread myThread = new MyThreadClass();
myThread.start();
try {
  myThread.sleep(10000);
} catch (InterruptedException e){
 
}
```

**Terminated**

A thread that has exited is in terminated state. This happens when the thread has completed executing the run() method. A thread may also be terminated any time by calling its **stop()** method (*note that Thread.stop() is deprecated and it is inherently unsafe*)

[![thread states in java](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/03/18:24:52-Thread%252Bstate.png)](https://2.bp.blogspot.com/-YVtJlB5Xs2E/VYRCLwVd3AI/AAAAAAAAALI/hnKKyDEoiBA/s1600/Thread%2Bstate.png)

**Various thread states**

### Getting thread's state in Java

Thread class has a method getState(), using that method you can get the current state of a thread. This method returns Thread.State enum which has constants for all the above mentioned thread states.

```
public class ThreadStates {

 public static void main(String[] args) {
  Thread thread = new Thread(new MyThread(), "MyThread");
  displayState("State after creation", thread);
  // Calling start method
  thread.start();
 
  try {
   // sleep method on Main thread 
   Thread.sleep(200);
   displayState("After starting ", thread);
   thread.join();
  } catch (InterruptedException e) {
   // TODO Auto-generated catch block
   e.printStackTrace();
  }
  displayState("State after running ", thread);
 }
 
 public static void displayState(String msg, Thread thread){
  System.out.println(msg + "- " + "Thread Name- " + thread.getName() 
     + " State- " +  thread.getState());
 }
}

class MyThread implements Runnable{
  @Override
  public void run() {
    ThreadStates.displayState("State in run method ", Thread.currentThread());
    System.out.println("In run method of MyThread --" + Thread.currentThread().getName());   
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
    ThreadStates.displayState("In run method after sleep", Thread.currentThread());  
  }    
}
```

**Output**

```
State after creation- Thread Name- MyThread State- NEW
State in run method - Thread Name- MyThread State- RUNNABLE
In run method of MyThread --MyThread
After starting - Thread Name- MyThread State- TIMED_WAITING
In run method after sleep- Thread Name- MyThread State- RUNNABLE
State after running - Thread Name- MyThread State- TERMINATED
```

That's all for this topic **Thread States (Thread Life Cycle) in Java Multi-Threading** . If you have any doubt or any suggestions to make please drop a comment. Thanks!