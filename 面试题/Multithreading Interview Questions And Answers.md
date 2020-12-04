### Java Multithreading Interview Questions And Answers

In this post Java multithreading interview questions and answers are listed. This compilation will help the Java developers in preparing for their interviews.

1. **What is thread in Java?**

   According to JavaDoc A thread is a thread of execution in a program. The Java Virtual Machine allows an application to have multiple threads of execution running concurrently.

2. ------

3. **How do we create thread in Java?**

   In Java there are two ways to create thread.

   - By implementing the Runnable interface.
   - By extending the Thread class.

   **Example code snippet using Runnable interface**

   ```
   class MyThread implements Runnable{
     MyThread(){
       Thread t = new Thread(this, "My Thread");
       t.start();
     }
     ..
     ..
   }
   ```

   **Example code snippet using Thread class**

   ```
   class TestThread extends Thread{
   ```

   and then

   ```
   TestThread t = new TestThread();
   // starting the thread
   t.start();
   ```

   After the new thread is created, it will not start running until you call the start( ) method.

   

   Read more about How to create thread in Java

    

   here

   .

4. ------

5. **What is the difference between thread and process?**

   

   - A process has a self-contained execution environment, Threads exist within a process - every process has at least one.
   - Process are heavyweight tasks whereas threads are referred as lightweight processes as creating a new thread requires fewer resources than creating a new process.
   - Each Process has its own *separate address spaces*, threads with in the same process *share the process' resources*, including memory and open files. This means that it's very easy to share data amongst threads, but it's also easy for the threads to bump on each other, which can lead to unpredictable scenarios.
   - **Inter process communication** is expensive whereas **inter thread communication** is inexpensive and in Java can be achieved easily using wait and notify.
   - Context switching from one process to another *is expensive*; context switching between threads is *generally less expensive than in processes*.
   - Threads are *easier to create* than processes as separate address space is not required for a thread.

   

   Read more about difference between thread and process in Java

    

   here

   .

6. ------

7. **What are the different thread states?**

   

   - **New**- When a thread is created either by extending Thread class or implementing Runnable interface it is in "New State".
   - **Runnable**- When we call start() method on the thread object that causes the thread to begin execution and it's the Java Virtual Machine that calls the run method of the thread.
   - **Blocked**- When a resource is shared among various threads then a thread may go into blocked state as the resource may be used by another thread.
   - **Waiting**- A thread that is waiting indefinitely for another thread to perform a particular action is in the waiting state.
   - **Timed_Waiting**- A thread that is waiting for another thread to perform an action for up to a specified waiting time is in timed_waiting state.

   

   Read more about different thread states in Java

    

   here

   .

8. ------

9. **What are the thread priorities and how to set thread priority?**

   When a Java thread is created, it inherits its priority from the thread that created it. Thread's priority can be modified at any time after its creation using the setPriority() method which is a member of Thread class.

   ```
    public final void setPriority(int newPriority)
    
   ```

   Here newPriority specifies the new priority setting for the calling thread. The priority level ranges from 1 (least important) to 10 (most important) and the default priority level is 5.

   In Thread class, three constants are provided to define min, max and default priority for a thread.

   ```
   public final static int MIN_PRIORITY = 1;
   public final static int NORM_PRIORITY = 5;
   public final static int MAX_PRIORITY = 10;
   ```

   

   Read more about thread priorities in Java

    

   here

   .

10. ------

11. **What does isAlive() method do in Java threading?**

    isAlive() method is the member of the Thread class and its general form is -

    ```
    public final boolean isAlive()
    ```

    isAlive()

     

    method tests if the thread it is called upon is alive. A thread is alive if it has been started and has not yet died. The isAlive( ) method returns true if the thread upon which it is called is still running, otherwise it returns false.

    

    If we have a thread -

    ```
    Thread t1 = new Thread(new MyRunnableClass(), "t1");
    ```

    Then we can use isAlive to check if it is still running or not.

    ```
    System.out.println("t1 Alive - " + t1.isAlive());
    ```

    Read more about isAlive() method

     

    here

    .

12. ------

13. **What is join() method?**

    Join() method is used when you want to wait for the thread to finish. Its general form is -

    ```
    public final void join() throws InterruptedException
    ```

    This method waits until the thread on which it is called terminates.

    

    Read more about join() method

     

    here

    .

14. ------

15. **Can we call run() method directly instead of start()?**

    When we call start() method on the thread that causes this thread to begin execution and it's the Java Virtual Machine that calls the run method of this thread.
    If we directly call run method it will be treated as a normal overridden method of the thread class (or runnable interface). This run method will be executed with in the context of the current thread not in a new thread.

    Read more about Can we call run() method directly

     

    here

    .

16. ------

17. **Can we start the same thread twice in Java?**

    No. A thread can only be started once and any attempt to start the same thread twice will throw IllegalThreadStateException.

    Read more about Can we start the same thread twice

     

    here

    .

18. ------

19. **What are wait, notify, notifyall method used for in Java multi-threading?**

    Java provides **inter-thread communication using the wait(), notify() and notifyAll() methods of the Object class**.

    - **wait method**- tells the current thread (thread which is executing code inside a synchronized method or block) to give up monitor and go to sleep, until another thread invokes the notify() or notifyAll() method for this object.
    - **notify method**- Wakes up a single thread that is waiting on this object's monitor. If more than one threads are waiting on this object, one of them is chosen to be awakened. The choice is arbitrary and occurs at the discretion of the implementation.
    - **notifyAll method**- wakes up all the threads that called wait( ) on the same object.

    

    Read more about wait, notify, notifyall

     

    here

    .

20. ------

21. **What is the difference between notify() and notifyAll()?**

    - **Notify method** wakes up a single thread that is waiting to acquire a lock on the object. If more than one threads are waiting on this object, one of them is chosen to be awakened.
    - **NotifyAll method** wakes up all the threads that called wait on the same object. Note that only one of these threads will get a lock to the object.

22. ------

23. **Why wait(), notify() and notifyAll() methods are in object class?**

    wait() and notify() work at the **monitor level**, thread which is currently holding the monitor is asked to give up that monitor through wait and through notify (or notifyAll) thread which are waiting on the object's monitor are notified that thread can wake up. *Important point to note here is that monitor is assigned to an object not to a particular thread.* That's one reason why these methods are in Object class.

    wait, notify and notifyAll() are used for **inter-thread communication**. But threads themselves have no knowledge of each others status. It is the shared object among the threads that acts as a communicator among the threads. Threads lock an object, wait on an object and notify an object.

    Read more about Why wait(), notify() and notifyAll() methods are in object class

     

    here

    .

24. ------

25. **When notify method is called does the thread which comes out of waiting start executing instantly?**

    No, thread which comes out of waiting because of the notify() method will not be able to proceed until the current thread relinquishes the lock on this object. The awakened thread just changes to the runnable state and it is ready to be scheduled again. The awakened thread will compete in the usual manner with any other threads that might be actively competing to synchronize on this object.

    Read more about wait, notify, notifyall

     

    here

    .

26. ------

27. **Why wait, notify and notifyAll must be called inside a synchronized method or block?**

    wait method tells the current thread (thread which is executing code inside a synchronized method or block) to give up monitor. Object's lock is acquired by a thread only when it is executing in a synchronized context. So it makes sense to use wait() method, which asks thread to release the lock only in synchronized context.

    Same way; when object's notify() or notifyAll() method is called, single thread (in case of notify) or all of the threads (in case of notifyAll), waiting for the object's lock change state to runnable and contend for the object's lock, and the thread that gets the lock starts execution. Here again, notify and notifyAll can inform other threads, that the object's lock can be acquired now, only if these methods are called from the synchronized object.

    Read more about Why wait, notify and notifyAll must be called inside a synchronized method or block

     

    here

    .

28. ------

29. **What is spurious wakeup?**

    Once wait is called on an object the thread that is currently executing with in the synchronized context waits until notify or notfyAll method is called. But there is a possibility that a waiting thread resumes again even when notify() or notifyAll() are not called (this will rarely occur in practice). This is known as **spurious wakeup**.
    To guard against it the recommendation is that; call to wait() method should be with in a loop that checks the condition on which the thread is waiting.

30. ------

31. **What does yield method do?**

    **Yield**- A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint. The executing thread is suspended and the CPU is given to some other runnable thread. This thread will wait until the CPU becomes available again.Technically, in process scheduler's terminology, the executing thread is put back into the ready queue of the processor and waits for its next turn.
    yield is a static method of the Thread class. When called it will work on the currently executing thread, not on any particular thread.

    Read more about yield method

     

    here

    .

32. ------

33. **What is the difference between sleep and wait?**

    - The very first difference is that sleep is defined as a static method in Thread class and operates on the currently executing thread. Whereas wait() method is in Object class and operates on the thread which is currently holding the lock on the object on which the wait method is called.
    - Since wait method is supposed to release the lock on an object so it has to be called from a synchronized context (with in a synchronized method or synchronized block). If not called from a synchronized context will result in a IllegalMonitorStateException being thrown. With sleep method there is no such mandatory condition it doesn't need to be called from a synchronized context.
    - Wait method will release the lock it holds on an object before going to waiting state which gives another thread a chance to enter the synchronized block. Sleep method if called from a synchronized block or method will not release the lock so another won't get a chance to enter the synchronized block.

    Read more about difference between sleep and wait

     

    here

    .

34. ------

35. **Why Thread.sleep() and yield() are static?**

    yield and sleep methods are used to work on the currently executing thread. Thus these methods are static so that you don't call it on any other thread.

36. ------

37. **What is synchronization in Java multi-threading?**

    In a multithreaded environment when more than one thread are trying to access a shared resource we need to have some way to ensure that the resource will be used by only one thread at a time. The process by which it is ensured is called synchronization.

    Read more about synchronization in Java multi-threading

     

    here

    .

38. ------

39. **What is a synchronized block?**

    It is not always needed to synchronize the whole method, let's say we have a 100 line code method, out of which critical section (shared resource) is only 7 lines then it makes sense to synchronize only those 7 lines rather than synchronizing the whole method. That is known as synchronized block or statement.

    ```
    class Message{
      public void displayMsg(String msg){
        System.out.println("Inside displayMsg method " +    Thread.currentThread().getName());
        synchronized(this){
          System.out.print("**" + msg);        
          try {
              Thread.sleep(3);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          System.out.println("*");
        }
      }
    }
    ```

    Read more about synchronization in Java multi-threading

     

    here

    .

40. ------

41. **What is race condition in multi-threading?**

    Race condition occurs in a multi-threaded environment when more than one thread try to access a shared resource (modify, write) at the same time. Note that it is safe if multiple threads are trying to read a shared resource as long as they are not trying to change it. Since multiple threads try to race each other to finish executing a method thus the name race condition.

    Read more about race condition in Java multi-threading

     

    here

    .

42. ------

43. **What is a deadlock in Multi-threading?**

    Deadlock describes a situation where two or more threads are blocked forever, waiting for each other. To describe it in a simple manner let's assume there are two threads Thread-1 and Thread-2 and two objects obj1 and obj2. Thread-1 already holds a lock on obj1 and for further processing it needs a lock on obj2. At the same time Thread-2 holds a lock on obj2 and wants a lock on obj1. In that kind of scenario both threads will be waiting for each other forever to release lock they are already holding thus creating a deadlock.

    Read more about deadlock in Multi-threading

     

    here

    .

44. ------

45. **Write a program to create deadlock in Java.**

    A deadlock can be created by having nested synchronized blocks where lock on objects are acquired in a reverse manner. i.e. if there are two objects obj1 and obj2 then first thread tries to acquire lock in sequence obj1 and then obj2. In the same time second thread tries to acquire lock in sequence obj2 first and then obj1.

    ```
    class Test{
      private final String name;
      public Test(String name){
        this.name = name;
      }
      public String getName() {
        return this.name;
      }
    }
    
    class ThreadA implements Runnable{
      private Test test1;
      private Test test2;
      ThreadA(Test test1, Test test2){
        this.test1 = test1;
        this.test2 = test2;
      }
      @Override
      public void run() {
        synchronized(test1){
          System.out.println("" + test1.getName());
          synchronized(test2){
            System.out.println("Reached here");
          }
        }      
      }  
    }
    
    class ThreadB implements Runnable{
      private Test test1;
      private Test test2;
      ThreadB(Test test1, Test test2){
        this.test1 = test1;
        this.test2 = test2;
      }
      @Override
      public void run() {
        synchronized(test2){
          System.out.println("" + test2.getName());
          synchronized(test1){
            System.out.println("Reached here");
          }
        }   
      }
    }
    public class DeadLockDemo1{
      public static void main(String[] args) {
        Test test1 = new Test("Test-1");
        Test test2 = new Test("Test-2");
        Thread t1 = new Thread(new ThreadA(test1, test2));
        Thread t2 = new Thread(new ThreadB(test1, test2));
        t1.start();
        
        t2.start();
      }
    }
    ```

    Thread t1 will start execution of run method in ThreadA and acquire lock on object test1 and then try to acquire lock on object test2. Meanwhile Thread t2 will start execution of run method in ThreadB and acquire lock on object test2 and then try to acquire lock on object test1. So both threads are trying to acquire a lock which is already held by another thread. Thus causing a deadlock.

    Read more about deadlock in Multi-threading

     

    here

    .

46. ------

47. **What is a thread local variable?**

    In Java there is a class called ThreadLocal which provides another way of thread-safety apart from synchronization. Usually when we have multiple threads sharing an object we need to synchronize the critical section of the code in order to make it thread safe.
    ThreadLocal class provides thread-local variables where each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. Since each and every threadhas its own copy of the object so explicit synchronization is not needed to provide thread safety.

    Read more about thread local variable

     

    here

    .

48. ------

49. **What is volatile in Java?**

    When a variable is declared as volatile, the compiler (even runtime) seeing the keyword volatile knows that this variable is shared. So the volatile variables are not cached in registers and read operation on a volatile variable always returns the most recent write by any thread.

    Read more about volatile

     

    here

    .

50. ------

51. **What is thread contention in multi-threading?**

    In multi-threaded application access to shared resources must be synchronized in order to avoid thread interference.

    However, synchronization can introduce thread contention. It occurs when two or more threads try to access the same resource simultaneously but can't get the lock causing the Java runtime to execute one or more threads slowly, or even suspend their execution.

    **Starvation** and **livelock** are forms of thread contention.

52. ------

53. **What is thread starvation?**

    In any multi-threaded application, you may come across a situation where a thread (or a bunch of threads) is unable to gain regular access to shared resources and is unable to make progress. This situation is known as thread starvation.

    Thread starvation may happen because other “greedy” threads are gaining the lock and access to the shared resource resulting in a thread (or a bunch of threads) getting starved of access to shared resources and CPU time.

    Read more about thread starvation

     

    here

    .

54. ------

55. **What are the reasons of thread starvation?**

    Some of the reasons why thread starvation may happen in Java -

    - If there is an object providing a synchronized method that requires a lots of processing time. If there is a thread invoking this method quite frequently, other threads that also need to access the synchronized method will be blocked.
    - If there are some higher priority threads those threads will get executed first rather than the threads having lower priority.
    - If you are using wait-notify signalling, then theoretically a thread that is waiting to get access to a shared resource may wait indefinitely if always some other thread is notified and get access to the shared resource.

    Read more about thread starvation

     

    here

    .

56. ------

57. **What is livelock?**

    Livelock in Java multi-threading is a situation where two or more threads are acting on a response to an action of each other and not able to make any progress because of that.

    Read more about livelock

     

    here

    .

58. ------

59. **How livelock is different from deadlock?**

    How livelock is different from deadlock is that in case of deadlock threads get blocked whereas in case of livelock threads are active but busy responding to each other thus not making any progress.

    Read more about livelock

     

    here

    .