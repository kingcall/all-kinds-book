### Java Phaser With Examples

**Phaser in Java** is also one of the synchronization aid provided in concurrency util. Phaser is similar to other synchronization barrier utils like [CountDownLatch](https://www.netjstech.com/2016/01/countdownlatch-in-java-concurrency.html) and [CyclicBarrier](https://www.netjstech.com/2016/01/cyclicbarrier-in-java-concurrency.html). What sets Phaser apart is it is **reusable** (like CyclicBarrier) and **more flexible** in usage. In both CountDownLatch and CyclicBarrier number of parties (thread) that are registered for waiting can't change where as in Phaser that **number can vary**. Also note that Phaser has been introduced in **Java 7**.

Phaser in Java is more suitable for use where it is required to *[synchronize threads](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) over one or more phases of activity*. Though Phaser can be used to synchronize a single phase, in that case it acts more like a CyclicBarrier. But it is more suited where threads should wait for a phase to finish, then advance to next phase, wait again for that phase to finish and so on.

**Table of contents**

1. [Java Phaser constructors](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserConstructor)
2. [How Phaser in Java works](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserWorks)
3. [Methods in Java Phaser class](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserMethods)
4. [Java Phaser Features](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserFeatures)
5. [Phaser Java example code](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserExp)
6. [Phaser Monitoring](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaserMonitoring)
7. [Overriding onAdvance() method in Phaser](https://www.netjstech.com/2016/01/phaser-in-java-concurrency.html#PhaseronAdvance)



### Java Phaser constructors

Phaser class in Java has 4 constructors

- **Phaser()**- Creates a new phaser with no initially registered parties, no parent, and initial phase number 0.
- **Phaser(int parties)**- Creates a new phaser with the given number of registered unarrived parties, no parent, and initial phase number 0.
- **Phaser(Phaser parent)**- Creates a new phaser with the given parent with no initially registered parties.
- **Phaser(Phaser parent, int parties)**- Creates a new phaser with the given parent and number of registered unarrived parties.

### How Phaser in Java works

First thing is to create a new instance of **Phaser**.

Next thing is to **register one or more parties** with the Phaser. That can be done using **register()**, **bulkRegister(int)** or by specifying number of parties in the constructor.

Now since Phaser is a synchronization barrier so we have to make phaser wait until all registered parties finish a phase. That waiting can be done using **arrive()** or any of the variants of arrive() method. When the number of arrivals is equal to the parties which are registered that phase is considered completed and it advances to next phase (if there is any), or terminate.

Note that each generation of a phaser has an associated phase number. The phase number starts at zero, and advances when all parties arrive at the phaser, wrapping around to zero after reaching Integer.MAX_VALUE.

### Methods in Java Phaser class

Some of the methods in Phaser class are as given below-

- resgister()

  \- Adds a new unarrived party to this phaser. It returns the arrival phase number to which this registration applied.

  

- **arrive()**- Arrives at this phaser, without waiting for others to arrive. Note that arrive() method does not suspend execution of the calling thread. Returns the arrival phase number, or a negative value if terminated. Note that this method should not be called by an unregistered party.

- **arriveAndDeregister()**- Arrives at this phaser and deregisters from it without waiting for others to arrive. Returns the arrival phase number, or a negative value if terminated.

- **arriveAndAwaitAdvance()**- This method awaits other threads to arrives at this phaser. Returns the arrival phase number, or the (negative) current phase if terminated. If you want to wait for all the other registered parties to complete a given phase then use this method.

- **bulkRegister(int parties)**– Used to register perties in bulk. Given number of new unarrived parties will be registered to this phaser.

- **onAdvance(int phase, int registeredParties)**– If you want to perform some action before the phase is advanced you can override this method. Also used to control termination.

### Java Phaser Features

**1. Phaser is more flexible**- Unlike the case for other barriers, the number of parties registered to synchronize on a Phaser may vary over time. Tasks may be **registered at any time** (using methods **register()**, **bulkRegister(int)**, or by specifying initial number of parties in the [constructor](https://www.netjstech.com/2015/04/constructor-in-java.html)). Tasks may also be optionally deregistered upon any arrival (using **arriveAndDeregister()**).

**2. Phaser termination**- A Phaser may enter a termination state, that may be checked using method isTerminated(). Upon termination, all synchronization methods immediately return without waiting for advance, as indicated by a negative return value. Similarly, attempts to register upon termination have no effect.

**3. Phaser Tiering**- Phasers in Java may be tiered (i.e., constructed in tree structures) to reduce contention. Phasers with large numbers of parties may experience heavy synchronization contention costs. These may be set up as a groups of sub-phasers which share a common parent. This may greatly increase throughput even though it incurs greater per-operation overhead.

### Phaser Java example code

Let's try to make things clearer through an example. So we'll have two phases in the application. In the first phase we have three threads reading 3 different files, parsing and storing them in DB, then in second phase 2 threads are started to query the DB table on the inserted records. Let's assume that one of the field is age in the DB table and we want to query count of those having age greater than 40 using one thread and in another thread we want to get the count of those having age less than or equal to 40.

```
public class PhaserDemo {

 public static void main(String[] args) {
  Phaser ph = new Phaser(1);
  int curPhase;
  curPhase = ph.getPhase();
  System.out.println("Phase in Main " + curPhase + " started");
  // Threads for first phase
  new FileReaderThread("thread-1", "file-1", ph);
  new FileReaderThread("thread-2", "file-2", ph);
  new FileReaderThread("thread-3", "file-3", ph);
  //For main thread
  ph.arriveAndAwaitAdvance();
  System.out.println("New phase " + ph.getPhase() + " started");
  // Threads for second phase
  new QueryThread("thread-1", 40, ph);
  new QueryThread("thread-2", 40, ph);
  curPhase = ph.getPhase();
  ph.arriveAndAwaitAdvance();
  System.out.println("Phase " + curPhase + " completed");
  // deregistering the main thread
  ph.arriveAndDeregister();
 }
}

class FileReaderThread implements Runnable {
 private String threadName;
 private String fileName;
 private Phaser ph;

 FileReaderThread(String threadName, String fileName, Phaser ph){
  this.threadName = threadName;
  this.fileName = fileName;
  this.ph = ph;
  ph.register();
  new Thread(this).start();
 }
 @Override
 public void run() {
  System.out.println("This is phase " + ph.getPhase());
  
  try {
   Thread.sleep(20);
   System.out.println("Reading file " + fileName + " thread " 
                           + threadName + " parsing and storing to DB ");
   // Using await and advance so that all thread wait here
   ph.arriveAndAwaitAdvance();
  } catch (InterruptedException e) {
   e.printStackTrace();
  }
  ph.arriveAndDeregister();
 }
}

class QueryThread implements Runnable {
 private String threadName;
 private int param;
 private Phaser ph;
 
 QueryThread(String threadName, int param, Phaser ph){
  this.threadName = threadName;
  this.param = param;
  this.ph = ph;
  ph.register();
  new Thread(this).start();
 }
 
 @Override
 public void run() {
  
  System.out.println("This is phase " + ph.getPhase());
  System.out.println("Querying DB using param " + param 
                  + " Thread " + threadName);
  ph.arriveAndAwaitAdvance();
  System.out.println("Threads finished");
  ph.arriveAndDeregister();
 }
}
```

**Output**

```
Phase in Main 0 started
This is phase 0
This is phase 0
This is phase 0
Reading file file-1 thread thread-1 parsing and storing to DB 
Reading file file-2 thread thread-2 parsing and storing to DB 
Reading file file-3 thread thread-3 parsing and storing to DB 
New phase 1 started
This is phase 1
Querying DB using param 40 Thread thread-1
This is phase 1
Querying DB using param 40 Thread thread-2
Threads finished
Threads finished
Phase 1 completed
```

Here it can be seen that first a Phaser instance ph is created with initial party count as 1, which corresponds to the main thread.

Then in the first set of 3 threads which are used in the first phase ph object is also passed which is used for synchronization. As you can see in the run method of the FileReaderThread class arriveAndAwaitAdvance() method is used so that the threads wait there for other threads. We have registered 3 more threads after the initial main thread so arriveAndAwaitAdvance() is used in the main method too to make the main thread wait before advancing.

In the second phase another set of two threads are created which are using the same phaser object ph for synchronization.

Logic for reading the file, parsing the file and storing it in the DB is not given here. Also the queries used in the second thread are not given. The scenario used here is to explain Phaser so that's where the concentration is.

### Phaser Monitoring

Phaser class in Java has several methods for monitoring. These methods can be called by any caller not only by registered parties.

- **getRegisteredParties()**- Returns the number of parties registered at this phaser.
- **getArrivedParties()**- Returns the number of registered parties that have arrived at the current phase of this phaser.
- **getUnarrivedParties()**- Returns the number of registered parties that have not yet arrived at the current phase of this phaser.
- **getPhase()**- Returns the current phase number.

### Overriding onAdvance() method in Phaser

If you want to perform an action before advancing from one phase to another, it can be done by overriding the **onAdvance()** method of the **Phaser** class. This method is invoked when the Phaser advances from one phase to another.
If this method returns **true**, this phaser will be set to a final termination state upon advance, and subsequent calls to isTerminated() will return true.
If this method returns **false**, phaser will be kept alive.

**onAdvance() method**

```
protected boolean onAdvance(int phase, int registeredParties)
```

Here

- **phase**- current phase number on entry to this method, before this phaser is advanced.
- **registeredParties**- the current number of registered parties.

One of the use case to override onAdvance() method is to ensure that your phaser executes a given number of phases and then stop.

So we'll create a class called PhaserAdvance that will extend Phaser and [override](https://www.netjstech.com/2015/04/method-overriding-in-java.html) the **onAdvance()** method to ensure that specified number of phases are executed.

**Overriding onAdvance() method example**

```
public class PhaserAdvance extends Phaser{
  PhaserAdvance(int parties){
    super(parties);
  }
    
  // Overriding the onAdvance method
  @Override
  protected boolean onAdvance(int phase, int registeredParties) {
    System.out.println("In onAdvance method, current phase which is completed 
      is " + phase );
    // Want to ensure that phaser runs for 2 phases i.e. phase 1 
    // or the no. of registered parties become zero
    if(phase == 1 || registeredParties == 0){
      System.out.println("phaser will be terminated ");
      return true;
    }else{
      System.out.println("phaser will continue ");
      return false;
    }     
  }
    
  public static void main(String... args) {
    // crating phaser instance
    PhaserAdvance ph = new PhaserAdvance(1);
    // creating three threads
    new TestThread("thread-1", ph);
    new TestThread("thread-2", ph);
    new TestThread("thread-3", ph);
    
    while(!ph.isTerminated()){
      ph.arriveAndAwaitAdvance();
    }
    System.out.println("In main method, phaser is terminated");
  }
}

class TestThread implements Runnable {
  private String threadName;
  private Phaser ph;

  TestThread(String threadName, Phaser ph){
    this.threadName = threadName;
    this.ph = ph;
    // register new unarrived party to this phaser
    ph.register();
    new Thread(this).start();
  }
  @Override
  public void run() {
    // be in the loop till the phaser is terminated
    while(!ph.isTerminated()){
      System.out.println("This is phase " + ph.getPhase() + 
        " And Thread - "+ threadName);
      // Using await and advance so that all thread wait here
      ph.arriveAndAwaitAdvance();
    }      
  }
}
```

**Output**

```
This is phase 0 And Thread - thread-1
This is phase 0 And Thread - thread-2
This is phase 0 And Thread - thread-3
In onAdvance method, current phase which is completed is 0
phaser will continue 
This is phase 1 And Thread - thread-3
This is phase 1 And Thread - thread-2
This is phase 1 And Thread - thread-1
In onAdvance method, current phase which is completed is 1
phaser will be terminated 
In main method, phaser is terminated
```

Here it can be seen that a new class PhaserAdvance is created extending the Phaser class. This PhaserAdvance class overrides the onAdvance() method of the Phaser class. In the overridden onAdvance() method it is ensured that 2 phases are executed thus the if condition with phase == 1 (phase count starts from 0).

That's all for this topic **Java Phaser With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!