### Why wait(), notify() And notifyAll() Must be Called Inside a Synchronized Method or Block

Why wait(), notify() and notifyAll() methods in Java must be called inside a synchronized method or block is a very frequently asked [Java multi-threading interview question](https://www.netjstech.com/2015/08/java-multi-threading-interview-questions.html). It is very closely related to another multi-threading question [Why wait(), notify() and notifyAll() methods are in Object class?](https://www.netjstech.com/2015/07/why-wait-notify-and-notifyall-methods-in-object-class-java-multi-threading.html)

There are few points we should be aware of before going into the reasons for why [wait(), notify() and notifyAll()](https://www.netjstech.com/2015/07/inter-thread-communiction-wait-notify-java-multi-thread.html) must be called inside a synchronized method or block.

1. Every object created in Java **has one associated monitor** (mutually exclusive lock). Only one thread can own a monitor at any given time.
2. For achieving [synchronization in Java](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) this monitor is used. When any thread enters a synchronized method/block it acquires the [lock](https://www.netjstech.com/2016/05/lock-striping-in-java-concurrency.html) on the specified object. When any thread acquires a lock it is said to have entered the monitor. All other threads which need to execute the same shared piece of code (locked monitor) will be suspended until the thread which initially acquired the lock releases it.
3. **wait** method tells the current thread (thread which is executing code inside a synchronized method or block) to give up monitor and go to [waiting state](https://www.netjstech.com/2015/06/lifecycle-of-thread-thread-states-in-java-multithreading.html).
4. **notify** method wakes up a single thread that is waiting on this object's monitor.
5. **notifyAll** method wakes up all the threads that called wait() on the same object.

Any method or a block of code, if not qualified with the keyword synchronized can be executed by more than one [thread](https://www.netjstech.com/2015/06/creating-thread-in-java.html) at any given time as object's monitor(lock) is not in the picture. Where as when a method is [synchronized](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) (or there is a synchronized block) only a single thread who has acquired the object's monitor can access the code.

I hope you would have got the picture by now. *Since **wait()** method is about thread releasing the object's lock and go to sleep where as **notify/notifyAll** methods are about notifying the thread(s) waiting for the object's lock*. So, **wait()**, **notify()** and **notifyAll()** methods (as mentioned above) should be invoked on an object only when the current thread has already acquired the lock on an object.
In fact not doing so will result in **java.lang.IllegalMonitorStateException**.

**As example** suppose I have this code where I have commented the synchronized keyword and trying to call wait

```
while(true){
    //synchronized (sharedListObj) {
    // While condition as mandated to avoid spurious wakeup
    while(sharedListObj.size() >= 1){
        try {
            sharedListObj.wait();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    // Putting value in the list
    System.out.println("Adding to queue - " + Thread.currentThread().getName() + " " + ++i);
    sharedListObj.add(i);
    //sharedListObj.notify();    
    // To get out of while(true) loop
    if(i > 4) break;
    //}
}
```

This will throw exception-

```
Exception in thread "Producer" java.lang.IllegalMonitorStateException
 at java.lang.Object.wait(Native Method)
 at java.lang.Object.wait(Object.java:502)
 at org.netjs.examples.Producer.run(InterThreadDemo.java:20)
 at java.lang.Thread.run(Thread.java:745)
```

To summarize it, **wait()** method tells the current thread (thread which is executing code inside a synchronized method or block) to give up monitor. *Object's lock is acquired by a thread only when it is executing in a synchronized context. So it makes sense to use **wait()** method, which asks thread to release the lock, only in synchronized context*.

Same way; when object's **notify()** or **notifyAll()** method is called, single thread (in case of notify) or all of the threads (in case of notifyAll), waiting for the object's lock change state to runnable and contend for the object's lock, and the thread that gets the lock starts execution. *Here again, **notify()** and **notifyAll()** methods can inform other threads, that the object's lock can be acquired now, only if these methods are called from the synchronized object*.

**Recommendations for learning**

1. [Java Programming Masterclass Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fjava-the-complete-java-developer-course%2F)
2. [Java In-Depth: Become a Complete Java Engineer!](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fjava-in-depth-become-a-complete-java-engineer%2F)
3. [Spring Framework Master Class Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fspring-tutorial-for-beginners%2F)
4. [Complete Python Bootcamp Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcomplete-python-bootcamp%2F)
5. [Python for Data Science and Machine Learning](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fpython-for-data-science-and-machine-learning-bootcamp%2F)

So these are the reasons Why wait(), notify() And notifyAll() Must be Called Inside a Synchronized Method or Block. Please do share with me if you know any other reason for doing the same.