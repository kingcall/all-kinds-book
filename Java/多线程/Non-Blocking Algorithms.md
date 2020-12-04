### Non-Blocking Algorithms in Java

In a multi-threading application if you use a [lock](https://www.netjstech.com/2016/05/lock-striping-in-java-concurrency.html) or [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) only one thread at any given time can get hold to the monitor and enter the critical section, all other threads wait for the lock to get free.

Same way, if any data structure has to be used in a multi-threaded environment then it has to use some concurrent algorithm, if that concurrent algorithm allows only one thread at any given time and block all the others then that algorithm is a blocking algorithm. **Examples**- Synchronized [ArrayList](https://www.netjstech.com/2015/08/how-arraylist-works-internally-in-java.html) or [HashMap](https://www.netjstech.com/2015/05/how-hashmap-internally-works-in-java.html), implementations of [BlockingQueue interface](https://www.netjstech.com/2016/02/blockingqueue-in-java-concurrency.html) like [ArrayBlockingQueue](https://www.netjstech.com/2016/02/arrayblockingqueue-in-java-concurrency.html) or [LinkedBlockingQueue](https://www.netjstech.com/2016/03/linkedblockingqueue-in-java.html) use that kind of lock-based algorithm thus run the risk of blocking the threads (may be for ever).

If a thread holding the lock is waiting for some resource like I/O or delayed due to some other fault then other waiting threads will not make any progress.

**As example**- If you are using an ArrayBlockingQueue, which is a bounded blocking queue, with capacity as 10. In that case if queue is full and another thread comes to put (using put() method) a value then the thread is blocked until some other thread takes (using take() method) a value out.

### Non-blocking algorithm

To prevent the problems as stated above non-blocking algorithm based classes/data structures are introduced in Java starting Java 5. Some of the examples are atomic operation supporting classes like [AtomicInteger](https://www.netjstech.com/2016/06/atomicinteger-in-java-concurrency.html), AtomicLong and Concurrent collection like [ConcurrentLinkedQueue in Java](https://www.netjstech.com/2016/04/concurrentlinkedqueue-in-java.html).

An algorithm is called non-blocking if it doesn't block threads in such a way that only one thread has access to the data structure and all the other threads are waiting. Same way failure of any thread in a non-blocking algorithm doesn't mean failure or suspension of other threads.

### Compare-And-Swap

Implementation of non-blocking data structures in Java like atomic variables or ConcurrentLinkedQueue use an atomic read-modify-write kind of instruction based on compare-and-swap.

Reference- https://en.wikipedia.org/wiki/Compare-and-swap

According to the description from "Java Concurrency in Practice" by Brian Goetz. CAS has three operands

1. A memory location M on which to operate
2. Expected old value OV
3. New value NV



CAS will match the expected old value OV to the value stored at the memory location M, if both match then only CAS will update the memory location M to the new value NV, otherwise it does nothing. In either case, it returns the value currently in M. The variant of Compare-and-swap called compare-and-set returns a boolean value indicating success/failure of the operation. In Java classes like AtomicInteger compare-and-set method is provided.

When multiple threads attempt to update the same variable simultaneously using CAS, one of those threads wins and updates the variable's value, and the rest lose. But the losers are not punished by suspension, as they could be if they failed to acquire a lock; instead, they are told that they didn't win the race this time but can try again.

Because a thread that loses a CAS is not blocked, it can decide whether it wants to

- try again,
- take some other recovery action,
- or

- do nothing

**As example**- At memory location M current value stored is 5, CAS is called by one thread with expected old value as 5 and new value as 6. At the same time another thread tries to change the value at M by passing 6 as old value and 7 as new value.

In that case first thread will succeed in changing the value stored at M to 6 whereas the other thread will report failure as matching for it will fail.

So the point is that threads are not blocked they may fail to get the desired result and may have to call CAS in a loop to get the result, which will result in more CPU cycles but no blocking/failure of threads because one of the thread has acquired lock and not releasing it.

**Recommendations for learning**

1. [Java Programming Masterclass Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fjava-the-complete-java-developer-course%2F)
2. [Java In-Depth: Become a Complete Java Engineer!](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fjava-in-depth-become-a-complete-java-engineer%2F)
3. [Spring Framework Master Class Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fspring-tutorial-for-beginners%2F)
4. [Complete Python Bootcamp Course](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcomplete-python-bootcamp%2F)
5. [Python for Data Science and Machine Learning](https://click.linksynergy.com/deeplink?id=*H/8FfjgiRQ&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fpython-for-data-science-and-machine-learning-bootcamp%2F)

That's all for this topic **Non-Blocking Algorithms in Java**. If you have any doubt or any suggestions to make please drop a comment. Thanks!