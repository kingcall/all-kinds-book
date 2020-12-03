### Java StampedLock With Examples

In Java 8 a new kind of lock **StampedLock** is added which apart from providing separate read and write locks also has a feature for **optimistic locking** for read operations. StampedLock in Java also provides method to upgrade read lock to write lock which is not there in [ReentrantReadWriteLock in Java](https://www.netjstech.com/2016/02/reentrantreadwritelock-in-java.html).

The locking methods of StampedLock in Java return a **stamp** represented by a long value. You can use these stamps to either release a lock, to check if the lock is still valid, to convert a lock.

So, if you want to use a StampedLock for acquiring a write lock it can be done as -

```
StampedLock sl = new StampedLock();
//acquiring writelock
long stamp =  sl.writeLock();
try{
 ...
 ...
}finally {
 //releasing lock
 sl.unlockWrite(stamp);
}
```

### Modes in Java StampedLock

As already mentioned StampedLock has an extra mode optimistic reading apart from reading and writing.

- **Writing mode**– When you try to acquire a write lock. It can be done using Method writeLock() which exclusively acquires the lock, blocking if necessary until available. This method retruns a stamp that can be used in method unlockWrite(long)to release the lock or in conversion of the lock.
  Untimed and timed versions of tryWriteLock are also provided. This method won’t block and return stamp as zero if the lock is not immediately available (or with in the given in case of timed version). When the lock is held in write mode, no read locks may be obtained, and all optimistic read validations will fail.
- **Reading Mode**– When you try to acquire a read lock. It can be done using method readLock() which non-exclusively acquires the lock, blocking if necessary until available returning a stamp that can be used to unlock or convert mode. Untimed and timed versions of tryReadLock are also provided.
- **Optimistic Reading**– That is the new mode added in StampedLock. Method **tryOptimisticRead()** is used to read in optimistic mode. This method returns a non-zero stamp only if the lock is not currently held in write mode.
  Method **validate(long)** is used to validate if the values read optimistically are correct or not. Validate() method returns true if the lock has not been acquired in write mode since obtaining a given stamp.

### Lock conversion in StampedLock

StampedLock class in Java also supports methods that conditionally provide conversions across the three modes. The forms of these methods are designed to help reduce some of the code bloat that otherwise occurs in retry-based designs.

**For example**, method tryConvertToWriteLock(long)attempts to "upgrade" a mode, returning a valid write stamp if

```
(1) already in writing mode 
(2) in reading mode and there are no other readers or 
(3) in optimistic mode and the lock is available. 
```

- **tryConvertToWriteLock(long stamp)**- If the lock state matches the given stamp, performs one of the following actions. If the stamp represents holding a write lock, returns it. Or, if a read lock, if the write lock is available, releases the read lock and returns a write stamp. Or, if an optimistic read, returns a write stamp only if immediately available. This method returns zero in all other cases.
- **tryConvertToReadLock(long stamp)**- If the lock state matches the given stamp, performs one of the following actions. If the stamp represents holding a write lock, releases it and obtains a read lock. Or, if a read lock, returns it. Or, if an optimistic read, acquires a read lock and returns a read stamp only if immediately available. This method returns zero in all other cases.
- **tryConvertToOptimisticRead(long stamp)**- If the lock state matches the given stamp then, if the stamp represents holding a lock, releases it and returns an observation stamp. Or, if an optimistic read, returns it if validated. This method returns zero in all other cases, and so may be useful as a form of "tryUnlock".

### Features of StampedLock

\1. The scheduling policy of StampedLock does not consistently prefer readers over writers or vice versa so there is no acquisition preference. All "try" methods are best-effort and do not necessarily conform to any scheduling or fairness policy.

\2. Unlike [ReentrantLocks](https://www.netjstech.com/2016/02/reentrantlock-in-java-concurrency.html), StampedLocks are not reentrant, so locked bodies should not call other unknown methods that may try to re-acquire locks (although you may pass a stamp to other methods that can use or convert it).

### StampedLock Java examples

Let’s see some examples in order to get a better understanding of the StampedLock.

**Using the read/write lock**

This example uses the write lock in order to get an exclusive lock for a counter and there is also a read lock which tries to read the value of the counter.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.StampedLock;

public class CounterST {
  int c = 0;
  public static void main(String[] args) {
    StampedLock sl = new StampedLock();
    ExecutorService executor = Executors.newFixedThreadPool(2);
    CounterST cst = new CounterST();
                
    // Runnable as lambda - read
    Runnable readTask = ()->{
      long stamp = sl.readLock();
      try{
        System.out.println("value " + cst.getValue());          
      }finally{
        sl.unlockRead(stamp);
      }
    };
        
    // Runnable as lambda - Write lock
    Runnable writeTask = ()->{
      long stamp = sl.writeLock();
      try {
        cst.increment();
      }finally{
        sl.unlockWrite(stamp);
      }
    };
        
    // 3 write tasks
    executor.submit(writeTask);
    executor.submit(writeTask);
    executor.submit(writeTask);
    // 1 read task
    executor.submit(readTask);
    executor.shutdown();
  }
  public  void increment() {     
    c++;
    System.out.println("in increment " + c);
  }

  public  int getValue() {
    return c;
  }
}
```

**Output**

```
in increment 1
in increment 2
value 2
in increment 3
```

**Using tryOptimisticRead method - StampedLock example**

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.StampedLock;

public class StampedLockDemo {
  public static void main(String[] args) {
    StampedLock sl = new StampedLock();
    ExecutorService executor = Executors.newFixedThreadPool(2);
    // Runnable as lambda - optimistic read
    Runnable r1 = ()->{
      long stamp = sl.tryOptimisticRead();
      try{
        System.out.println("In optimistic lock " + sl.validate(stamp));
        try {
          TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
        System.out.println("In optimistic lock " + sl.validate(stamp));            
      }finally{
        sl.unlock(stamp);
      }
    };
        
    // Runnable as lambda - Write lock
    Runnable r2 = ()->{
      System.out.println("about to get write lock");
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
      }
      long stamp = sl.writeLock();
      try{
        System.out.println("After getting write lock ");
          
      }finally{
        sl.unlock(stamp);
        System.out.println("Relinquished write lock ");
      }
    };
        
    executor.submit(r2);
    // Optimistic read
    executor.submit(r1);
    executor.submit(r2);
    
    executor.shutdown();
  }
}
```

**Output**

```
about to get write lock
In optimistic lock true
After getting write lock 
Relinquished write lock 
about to get write lock
In optimistic lock false
After getting write lock 
Relinquished write lock 
```

Here it can be seen when write lock is acquired at that time validate method returns false for optimistic read. Optimistic mode can be thought of as an extremely weak version of a read-lock, that can be broken by a writer at any time, from the output you can see the same thing.

Before acquiring the write lock thread goes to sleep and the optimistic lock is acquired in the mean time. When the write lock is acquired later, optimistic read lock is broken so the validate method returns false for the same stamp.

Since optimistic read mode is a new feature provided by StampedLock so let’s see one more example which is provided in Javadocs.

```
double distanceFromOrigin() { // A read-only method
  long stamp = sl.tryOptimisticRead();
  double currentX = x, currentY = y;
  if (!sl.validate(stamp)) {
    stamp = sl.readLock();
    try {
      currentX = x;
      currentY = y;
    } finally {
      sl.unlockRead(stamp);
    }
  }
  return Math.sqrt(currentX * currentX + currentY * currentY);
}
```

In this method you can notice that initially values are assigned to variables currentX and currentY after getting the optimistic lock. Then validate method is used to check if the lock has not been exclusively acquired since issuance of the given stamp. In case validate method returns false (which means write lock is acquired by some thread after optimistic lock is acquired) then read lock method is acquired and values are assigned again. Here note that read lock may block if there is any write lock. So that is the benefit of optimistic lock you can acquire it and read the values and then check if there is any change in the values, if there is then only you need to go through the blocking read lock.

**StampedLock Example using tryConvertToWriteLock method**

```
void moveIfAtOrigin(double newX, double newY) { // upgrade
  // Could instead start with optimistic, not read mode
  long stamp = sl.readLock();
  try {
    while (x == 0.0 && y == 0.0) {
      long ws = sl.tryConvertToWriteLock(stamp);
      if (ws != 0L) {
        stamp = ws;
        x = newX;
        y = newY;
        break;
      }
      else {
        sl.unlockRead(stamp);
        stamp = sl.writeLock();
      }
    }
  } finally {
    sl.unlock(stamp);
  }
}
```

Here it can be seen that initially read lock is acquired and then some condition is checked if it satisfies then only an attempt is made to convert the read lock to write lock. If returned stamp is not zero that means conversion was successful otherwise go through the procedure of releasing the read lock and acquiring a write lock.

**Source**: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/concurrent/locks/StampedLock.html

That's all for this topic **Java StampedLock With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!