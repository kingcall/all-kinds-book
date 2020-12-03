### Synchronization in Java - Synchronized Method And Block

In this post we'll see how thread synchronization is done using synchronized keyword in Java.

In a multithreaded environment when multiple threads are trying to access a shared resource we need to have some mechanism to ensure that the resource will be used only by one thread at a time. The process by which it is ensured is called **synchronization** in multi-threading.

**Table of contents**

1. [How synchronization in Java works](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizationJava)
2. [Where can we use synchronized keyword in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizedKeyword)
3. [Synchronized Method in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizedMethod)
4. [Java synchronized method example](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizedMethodExp)
5. [Synchronized statement (block) in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizedBlockJava)
6. [Java synchronized block example](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizedBlockExp)
7. [Drawback of using Synchronization in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html#synchronizationDrawback)



### How synchronization in Java works

For achieving synchronization in Java **concept of monitor** is used. *Every object created in Java has one associated monitor (mutually exclusive lock)*. At any given time Only one [thread](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html) can own the monitor.

The Java programming language provides two basic synchronization idioms using the synchronized keyword.

- synchronized methods
- synchronized statements (also known as synchronized blocks).



Before any thread executes the code which is with in a synchronized method (or synchronized block) compiler provides instructions to acquire the lock on the specified object.

*When any thread acquires a [lock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html) it is said to have entered the monitor. All other threads which need to execute the same shared piece of code (locked monitor) will be suspended until the thread which initially acquired the lock releases it*.

### Where can we use synchronized keyword in Java

As already stated synchronized keyword can be used with methods or blocks in Java. This can be further divided into use with instance methods and static methods. Which means, we can have four different ways synchronized keyword in Java can be used.

- instance method
- An enclosed code block with in an instance method (Synchronized block).
- static method
- An enclosed code block with in a static method.



Refer [Static Synchronization in Java Multi-Threading](https://www.netjstech.com/2018/05/static-synchronization-in-java-multi-threading.html) to see how to use synchronization with static method and block and why it is needed.

Let's see where and why we should use a specific type of synchronized method or block.

### Synchronized Method in Java

We can synchronize a method by adding synchronized keyword within a method signature.

**General Form of synchronized instance method**

```
synchronized <returntype> method_name(parameter_list){
  ..
  ..
}
```

### Java synchronized method example

Let's see an example code where first we write the code without synchronizing the method and later we have the same code where method is synchronized.

Here we have a class **Message** whose object will be shared among threads. In class Message there is a method **displayMsg** and you want one thread to finish printing the message with in the method then only another thread starts executing the method.

**Code when synchronized keyword is not used**

```
// This class' shared object will be accessed by threads
class Message{
 public void displayMsg(String msg){
   System.out.println("Inside displayMsg method " + Thread.currentThread().getName());
   System.out.println("**" + msg); 
   try {
      Thread.sleep(10);
   } catch (InterruptedException e) {
      e.printStackTrace();
   }
   System.out.println("*");
 }
}
 
class MyClass implements Runnable{
  Thread t;
  Message msg;
  String message;
  MyClass(Message msg, String str){ 
    this.msg = msg;
    this.message = str;
    // creating threads, 4 threads will be created 
    // all sharing the same object msg
    t = new Thread(this);
    t.start();
  }
  @Override
  public void run() {
     msg.displayMsg(message);
  }
}

public class SynchronizedDemo {
  public static void main(String[] args) {
    Message msg = new Message();
    MyClass mc1 = new MyClass(msg, "I");
    MyClass mc2 = new MyClass(msg, "am");
    MyClass mc3 = new MyClass(msg, "not");
    MyClass mc4 = new MyClass(msg, "synchronized");
  }
}
```

**Output**

I got the following output, for you output may differ as it depends upon which thread is picked first.

```
Inside displayMsg method Thread-0
Inside displayMsg method Thread-3
**synchronized
Inside displayMsg method Thread-1
**am
Inside displayMsg method Thread-2
**not
**I
*
*
*
*
```

It can be seen how output is all jumbled, because all the 4 threads share the same object and synchronized keyword is not used to ensure that only single thread has the lock on the object and only that thread can execute the method.

**Code when synchronized is used**

If we synchronize a method only a single thread will access the method at the given time. In that case the **displayMsg()** method will look like -

```
class Message{
    public synchronized void displayMsg(String msg){
        System.out.println("Inside displayMsg method " + Thread.currentThread().getName());
        System.out.print("**" + msg);        
        try {
            Thread.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("*");
    }
}
```

**Output**

```
Inside displayMsg method Thread-0
**I*
Inside displayMsg method Thread-1
**am*
Inside displayMsg method Thread-2
**not*
Inside displayMsg method Thread-3
**synchronized*
```

Now it can be seen how one thread finishes its execution then only another thread starts its execution of the method. Though which thread is picked first is up to the scheduler that's why the message may not print correctly but the point here is that whichever thread enters the synchronized method that finishes then only the next thread starts its execution.

Please note that a synchronized instance method is synchronized on the instance(object) of the class. So, if a class has more than one object then one thread at a time can enter each object's synchronized method.

### Synchronized statement (block) in Java

You don't need to synchronize the whole method, let's say we have a 100 line code method, out of which **critical section** (shared resource) comprises of 7 lines only then it makes sense to synchronize those 7 lines only rather than the whole method. That way we can improve performance.

synchronized statements must specify the object that provides the intrinsic lock.

**General Form of Synchronized statement (synchronized block)**

```
synchronized(object_reference){
  // code block
}
```

Though it is also possible to synchronize on a string but it is considered a bad idea as "Literal strings within different classes in different packages likewise represent references to the same String object." See this for reference - http://www.javalobby.org/java/forums/t96352.html

### Java synchronized block example

If we use the same example used above with synchronized method it can be changed to use **synchronized block** like this-

```
//This class' shared object will be accessed by threads
class Message{
    public void displayMsg(String msg){
        System.out.println("Inside displayMsg method " + Thread.currentThread().getName());
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

**Output**

```
Inside displayMsg method Thread-0
Inside displayMsg method Thread-3
Inside displayMsg method Thread-2
Inside displayMsg method Thread-1
**I*
**am*
**not*
**synchronized*
```

It can be seen from the output how the first print statement is executed by all the threads as that is not inside the synchronized block. After that only a single thread executes at a time and finishes its execution of the synchronized block only then another thread enters the synchronized block.

### Drawback of using Synchronization in Java

Synchronization can introduce thread contention, which occurs when two or more threads try to access the same resource simultaneously and cause the Java runtime to execute one or more threads more slowly, or even suspend their execution. [Thread Starvation](https://www.netjstech.com/2017/10/thread-starvation-in-java-multi-threading.html) and [livelock](https://www.netjstech.com/2017/12/livelock-in-java-multi-threading.html) are forms of thread contention.

That's all for this topic **Synchronization in Java - Synchronized Method And Block**. If you have any doubt or any suggestions to make please drop a comment. Thanks!