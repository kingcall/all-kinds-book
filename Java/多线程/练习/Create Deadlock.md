### How to Create Deadlock in Java

This post is about writing a Java program to create deadlock in a multi-threaded application.

[Deadlock](https://www.netjstech.com/2015/07/deadlock-in-java-multi-threading.html) can happen if there are nested [synchronized blocks](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) in your code. There are 2 things to note here-

- Locks are acquired at object level.
- Only that thread which has acquired the lock can enter the synchronized block.

Logic for the Java program is that there are two Thread classes **ThreadA** and **ThreadB** and there are two objects of the class **DeadLckDemo**. In both of these classes **ThreadA** and **ThreadB** there are nested synchronized blocks and the [object](https://www.netjstech.com/2015/04/object-in-java.html) reference provided in the blocks is reversed in both of those classes.

In one of the class if nested block is as-

```
synchronized(obj1){
  System.out.println("" + Thread.currentThread().getName());
  synchronized(obj2){
    System.out.println("Reached here");
  }
}
```

Then in other class it is like this-

```
synchronized(obj2){
  System.out.println("" + Thread.currentThread().getName());
  synchronized(obj1){
    System.out.println("Reached here");
  }
}
```

If two [threads are started](https://www.netjstech.com/2015/06/can-we-start-same-thread-twice-in-java.html), one for **ThreadA** and another one for **ThreadB**. Thread **t1** will acquire a [lock](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html) on **obj1** and enter the outer synchronized block. Meanwhile thread **t2** will start and get a lock on **obj2** and enter the outer block in **ThreadB** class. That's where both of these threads will enter in a deadlock.

Thread **t1** will try to acquire lock on object **obj2** which is currently held by thread **t2** whereas thread **t2** will try to acquire a lock on object **obj1** which is curretly held by thread **t1**. That way these threads will wait for each other to release locks on respective objects and create a deadlock.

### Java Program to create deadlock

```
public class DeadLckDemo {
  public static void main(String[] args) {
    DeadLckDemo obj1 = new DeadLckDemo();
    DeadLckDemo obj2 = new DeadLckDemo();
    Thread t1 = new Thread(new ThreadA(obj1, obj2), "Thread-1");
    Thread t2 = new Thread(new ThreadB(obj1, obj2), "Thread-2");
    
    t1.start();    
    t2.start();
  }
}

class ThreadA implements Runnable{
  private DeadLckDemo obj1;
  private DeadLckDemo obj2;
  ThreadA(DeadLckDemo obj1, DeadLckDemo obj2){
    this.obj1 = obj1;
    this.obj2 = obj2;
  }
  @Override
  public void run() {
    synchronized(obj1){
      System.out.println("" + Thread.currentThread().getName());
      synchronized(obj2){
        System.out.println("Reached here");
      }
    }      
  }   
}

class ThreadB implements Runnable{
  private DeadLckDemo obj1;
  private DeadLckDemo obj2;
  ThreadB(DeadLckDemo obj1, DeadLckDemo obj2){
    this.obj1 = obj1;
    this.obj2 = obj2;
  }
  @Override
  public void run() {
    synchronized(obj2){
      System.out.println("" + Thread.currentThread().getName());
      synchronized(obj1){
        System.out.println("Reached here");
      }
    }   
  }
}
```

That's all for this topic **How to Create Deadlock in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!