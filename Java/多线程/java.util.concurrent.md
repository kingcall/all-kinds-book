Concurrent Collections 是 Java™ 5 的巨大附加产品，但是在关于注解和泛型的争执中很多 Java 开发人员忽视了它们。此外（或者更老实地说），许多开发人员避免使用这个数据包，因为他们认为它一定很复杂，就像它所要解决的问题一样。

事实上，`java.util.concurrent` 包含许多类，能够有效解决普通的并发问题，无需复杂工序。阅读本文，了解 `java.util.concurrent` 类，比如 `CopyOnWriteArrayList` 和 `BlockingQueue` 如何帮助您解决多线程编程的棘手问题。

## 1. TimeUnit

尽管 *本质上* 不是 Collections 类，但 `java.util.concurrent.TimeUnit` 枚举让代码更易读懂。使用 `TimeUnit` 将使用您的方法或 API 的开发人员从毫秒的 “暴政” 中解放出来。

`TimeUnit` 包括所有时间单位，从 `MILLISECONDS` 和 `MICROSECONDS` 到 `DAYS` 和 `HOURS` ，这就意味着它能够处理一个开发人员所需的几乎所有的时间范围类型。同时，因为在列举上声明了转换方法，在时间加快时，将 `HOURS` 转换回 `MILLISECONDS` 甚至变得更容易。

## 2. CopyOnWriteArrayList

创建数组的全新副本是过于昂贵的操作，无论是从时间上，还是从内存开销上，因此在通常使用中很少考虑；开发人员往往求助于使用同步的 `ArrayList` 。然而，这也是一个成本较高的选择，因为每当您跨集合内容进行迭代时，您就不得不同步所有操作，包括读和写，以此保证一致性。

这又让成本结构回到这样一个场景：需多读者都在读取 `ArrayList` ，但是几乎没人会去修改它。

`CopyOnWriteArrayList` 是个巧妙的小宝贝，能解决这一问题。它的 Javadoc 将 `CopyOnWriteArrayList` 定义为一个 “`ArrayList` 的线程安全变体，在这个变体中所有易变操作（添加，设置等）可以通过复制全新的数组来实现”。

集合从内部将它的内容复制到一个没有修改的新数组，这样读者访问数组内容时就不会产生同步成本（因为他们从来不是在易变数据上操作）。

本质上讲， `CopyOnWriteArrayList` 很适合处理 `ArrayList` 经常让我们失败的这种场景：读取频繁，但很少有写操作的集合，例如 JavaBean 事件的 `Listener` s。

## 3. BlockingQueue

`BlockingQueue` 接口表示它是一个 `Queue` ，意思是它的项以先入先出（FIFO）顺序存储。在特定顺序插入的项以相同的顺序检索 — 但是需要附加保证，从空队列检索一个项的任何尝试都会阻塞调用线程，直到这个项准备好被检索。同理，想要将一个项插入到满队列的尝试也会导致阻塞调用线程，直到队列的存储空间可用。

`BlockingQueue` 干净利落地解决了如何将一个线程收集的项”传递”给另一线程用于处理的问题，无需考虑同步问题。Java Tutorial 的 Guarded Blocks 试用版就是一个很好的例子。它构建一个单插槽绑定的缓存，当新的项可用，而且插槽也准备好接受新的项时，使用手动同步和 `wait()` / `notifyAll()` 在线程之间发信。（详见 [Guarded Blocks 实现](http://java.sun.com/docs/books/tutorial/essential/concurrency/guardmeth.html) 。）

尽管 Guarded Blocks 教程中的代码有效，但是它耗时久，混乱，而且也并非完全直观。退回到 Java 平台较早的时候，没错，Java 开发人员不得不纠缠于这种代码；但现在是 2010 年 — 情况难道没有改善？

清单 1 显示了 Guarded Blocks 代码的重写版，其中我使用了一个 `ArrayBlockingQueue` ，而不是手写的 `Drop` 。

##### 清单 1. BlockingQueue

```
import java.util.*;
import java.util.concurrent.*;

class Producer
    implements Runnable
{
    private BlockingQueue<String> drop;
    List<String> messages = Arrays.asList(
        "Mares eat oats",
        "Does eat oats",
        "Little lambs eat ivy",
        "Wouldn't you eat ivy too?");

    public Producer(BlockingQueue<String> d) { this.drop = d; }

    public void run()
    {
        try
        {
            for (String s : messages)
                drop.put(s);
            drop.put("DONE");
        }
        catch (InterruptedException intEx)
        {
            System.out.println("Interrupted! " +
                "Last one out, turn out the lights!");
        }
    }
}

class Consumer
    implements Runnable
{
    private BlockingQueue<String> drop;
    public Consumer(BlockingQueue<String> d) { this.drop = d; }

    public void run()
    {
        try
        {
            String msg = null;
            while (!((msg = drop.take()).equals("DONE")))
                System.out.println(msg);
        }
        catch (InterruptedException intEx)
        {
            System.out.println("Interrupted! " +
                "Last one out, turn out the lights!");
        }
    }
}

public class ABQApp
{
    public static void main(String[] args)
    {
        BlockingQueue<String> drop = new ArrayBlockingQueue(1, true);
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

显示更多

`ArrayBlockingQueue` 还体现了”公平” — 意思是它为读取器和编写器提供线程先入先出访问。这种替代方法是一个更有效，但又冒穷尽部分线程风险的政策。（即，允许一些读取器在其他读取器锁定时运行效率更高，但是您可能会有读取器线程的流持续不断的风险，导致编写器无法进行工作。）

##### 注意 Bug！

顺便说一句，如果您注意到 Guarded Blocks 包含一个重大 bug，那么您是对的 如果开发人员在 `main()` 中的 `Drop` 实例上同步，会出现什么情况呢？

`BlockingQueue` 还支持接收时间参数的方法，时间参数表明线程在返回信号故障以插入或者检索有关项之前需要阻塞的时间。这么做会避免非绑定的等待，这对一个生产系统是致命的，因为一个非绑定的等待会很容易导致需要重启的系统挂起。

## 4. ConcurrentMap

`Map` 有一个微妙的并发 bug，这个 bug 将许多不知情的 Java 开发人员引入歧途。`ConcurrentMap` 是最容易的解决方案。

当一个 `Map` 被从多个线程访问时，通常使用 `containsKey()` 或者 `get()` 来查看给定键是否在存储键/值对之前出现。但是即使有一个同步的 `Map`，线程还是可以在这个过程中潜入，然后夺取对 `Map` 的控制权。问题是，在对 `put()` 的调用中，锁在 `get()` 开始时获取，然后在可以再次获取锁之前释放。它的结果是个竞争条件：这是两个线程之间的竞争，结果也会因谁先运行而不同。

如果两个线程几乎同时调用一个方法，两者都会进行测试，调用 put，在处理中丢失第一线程的值。幸运的是，`ConcurrentMap` 接口支持许多附加方法，它们设计用于在一个锁下进行两个任务：`putIfAbsent()`，例如，首先进行测试，然后仅当键没有存储在 `Map` 中时进行 put。

## 5. SynchronousQueues

根据 Javadoc，`SynchronousQueue` 是个有趣的东西：

> _这是一个阻塞队列，其中，每个插入操作必须等待另一个线程的对应移除操作，反之亦然。一个同步队列不具有任何内部容量，甚至不具有 1 的容量。

本质上讲，`SynchronousQueue` 是之前提过的 `BlockingQueue` 的又一实现。它给我们提供了在线程之间交换单一元素的极轻量级方法，使用 `ArrayBlockingQueue` 使用的阻塞语义。在清单 2 中，我重写了[清单 1](https://developer.ibm.com/zh/articles/j-5things4/#清单-1-blockingqueue) 的代码，使用 `SynchronousQueue` 替代 `ArrayBlockingQueue`：

##### 清单 2. SynchronousQueue

```
import java.util.*;
import java.util.concurrent.*;

class Producer
    implements Runnable
{
    private BlockingQueue<String> drop;
    List<String> messages = Arrays.asList(
        "Mares eat oats",
        "Does eat oats",
        "Little lambs eat ivy",
        "Wouldn't you eat ivy too?");

    public Producer(BlockingQueue<String> d) { this.drop = d; }

    public void run()
    {
        try
        {
            for (String s : messages)
                drop.put(s);
            drop.put("DONE");
        }
        catch (InterruptedException intEx)
        {
            System.out.println("Interrupted! " +
                "Last one out, turn out the lights!");
        }
    }
}

class Consumer
    implements Runnable
{
    private BlockingQueue<String> drop;
    public Consumer(BlockingQueue<String> d) { this.drop = d; }

    public void run()
    {
        try
        {
            String msg = null;
            while (!((msg = drop.take()).equals("DONE")))
                System.out.println(msg);
        }
        catch (InterruptedException intEx)
        {
            System.out.println("Interrupted! " +
                "Last one out, turn out the lights!");
        }
    }
}

public class SynQApp
{
    public static void main(String[] args)
    {
        BlockingQueue<String> drop = new SynchronousQueue<String>();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

显示更多

实现代码看起来几乎相同，但是应用程序有额外获益：`SynchronousQueue` 允许在队列进行一个插入，只要有一个线程等着使用它。

在实践中，`SynchronousQueue` 类似于 Ada 和 CSP 等语言中可用的 “会合通道”。这些通道有时在其他环境中也称为 “连接”，这样的环境包括 .NET（见参考资源）。

## 结束语

当 Java 运行时知识库提供便利、预置的并发性时，为什么还要苦苦挣扎，试图将并发性导入到您的 Collections 类？本系列的[下一篇文章](https://developer.ibm.com/zh/articles/j-5things5/))将会进一步探讨 `java.util.concurrent` 名称空间的内容。



并发 Collections 提供了线程安全、经过良好调优的数据结构，简化了并发编程。然而，在一些情形下，开发人员需要更进一步，思考如何调节和/或限制线程执行。由于 `java.util.concurrent` 的总体目标是简化多线程编程，您可能希望该包包含同步实用程序，而它确实包含。

本文是 [第 1 部分](https://developer.ibm.com/zh/articles/j-5things4/) 的延续，将介绍几个比核心语言原语（监视器）更高级的同步结构，但它们还未包含在 Collection 类中。一旦您了解了这些锁和门的用途，使用它们将非常直观。

##### 关于本系列

您觉得自己懂 Java 编程？事实是，大多数开发人员都只领会到了 Java 平台的皮毛，所学也只够应付工作。在本[本系列](https://developer.ibm.com/zh/series/5-things-you-didnt-know-about/) 中，Ted Neward 深度挖掘 Java 平台的核心功能，揭示一些鲜为人知的事实，帮助您解决最棘手的编程困难。

## 1. Semaphore

在一些企业系统中，开发人员经常需要限制未处理的特定资源请求（线程/操作）数量，事实上，限制有时候能够提高系统的吞吐量，因为它们减少了对特定资源的争用。尽管完全可以手动编写限制代码，但使用 Semaphore 类可以更轻松地完成此任务，它将帮您执行限制，如清单 1 所示：

##### 清单 1. 使用 Semaphore 执行限制

```
import java.util.*;import java.util.concurrent.*;

public class SemApp
{
    public static void main(String[] args)
    {
        Runnable limitedCall = new Runnable() {
            final Random rand = new Random();
            final Semaphore available = new Semaphore(3);
            int count = 0;
            public void run()
            {
                int time = rand.nextInt(15);
                int num = count++;

                try
                {
                    available.acquire();

                    System.out.println("Executing " +
                        "long-running action for " +
                        time + " seconds... #" + num);

                    Thread.sleep(time * 1000);

                    System.out.println("Done with #" +
                        num + "!");

                    available.release();
                }
                catch (InterruptedException intEx)
                {
                    intEx.printStackTrace();
                }
            }
        };

        for (int i=0; i<10; i++)
            new Thread(limitedCall).start();
    }
}
```

显示更多

即使本例中的 10 个线程都在运行（您可以对运行 `SemApp` 的 Java 进程执行 `jstack` 来验证），但只有 3 个线程是活跃的。在一个信号计数器释放之前，其他 7 个线程都处于空闲状态。（实际上， `Semaphore` 类支持一次获取和释放多个 *permit* ，但这不适用于本场景。）

## 2. CountDownLatch

如果 `Semaphore` 是允许一次进入一个（这可能会勾起一些流行夜总会的保安的记忆）线程的并发性类，那么 `CountDownLatch` 就像是赛马场的起跑门栅。此类持有所有空闲线程，直到满足特定条件，这时它将会一次释放所有这些线程。

##### 清单 2. CountDownLatch：让我们去赛马吧！

```
import java.util.*;
import java.util.concurrent.*;

class Race
{
    private Random rand = new Random();

    private int distance = rand.nextInt(250);
    private CountDownLatch start;
    private CountDownLatch finish;

    private List<String> horses = new ArrayList<String>();

    public Race(String... names)
    {
        this.horses.addAll(Arrays.asList(names));
    }

    public void run()
        throws InterruptedException
    {
        System.out.println("And the horses are stepping up to the gate...");
        final CountDownLatch start = new CountDownLatch(1);
        final CountDownLatch finish = new CountDownLatch(horses.size());
        final List<String> places =
            Collections.synchronizedList(new ArrayList<String>());

        for (final String h : horses)
        {
            new Thread(new Runnable() {
                public void run() {
                    try
                    {
                        System.out.println(h +
                            " stepping up to the gate...");
                        start.await();

                        int traveled = 0;
                        while (traveled < distance)
                        {
                            // In a 0-2 second period of time....
                            Thread.sleep(rand.nextInt(3) * 1000);

                            // ... a horse travels 0-14 lengths
                            traveled += rand.nextInt(15);
                            System.out.println(h +
                                " advanced to " + traveled + "!");
                        }
                        finish.countDown();
                        System.out.println(h +
                            " crossed the finish!");
                        places.add(h);
                    }
                    catch (InterruptedException intEx)
                    {
                        System.out.println("ABORTING RACE!!!");
                        intEx.printStackTrace();
                    }
                }
            }).start();
        }

        System.out.println("And... they're off!");
        start.countDown();

        finish.await();
        System.out.println("And we have our winners!");
        System.out.println(places.get(0) + " took the gold...");
        System.out.println(places.get(1) + " got the silver...");
        System.out.println("and " + places.get(2) + " took home the bronze.");
    }
}

public class CDLApp
{
    public static void main(String[] args)
        throws InterruptedException, java.io.IOException
    {
        System.out.println("Prepping...");

        Race r = new Race(
            "Beverly Takes a Bath",
            "RockerHorse",
            "Phineas",
            "Ferb",
            "Tin Cup",
            "I'm Faster Than a Monkey",
            "Glue Factory Reject"
            );

        System.out.println("It's a race of " + r.getDistance() + " lengths");

        System.out.println("Press Enter to run the race....");
        System.in.read();

        r.run();
    }
}
```

显示更多

注意，在[清单 2](https://developer.ibm.com/zh/articles/j-5things5/#清单-2-countdownlatch：让我们去赛马吧！) 中， `CountDownLatch` 有两个用途：首先，它同时释放所有线程，模拟马赛的起点，但随后会设置一个门闩模拟马赛的终点。这样，”主” 线程就可以输出结果。 为了让马赛有更多的输出注释，可以在赛场的 “转弯处” 和 “半程” 点，比如赛马跨过跑道的四分之一、二分之一和四分之三线时，添加 `CountDownLatch` 。

## 3. Executor

[清单 1](https://developer.ibm.com/zh/articles/j-5things5/#清单-1-使用-semaphore-执行限制) 和[清单 2](https://developer.ibm.com/zh/articles/j-5things5/#清单-2-countdownlatch：让我们去赛马吧！) 中的示例都存在一个重要的缺陷，它们要求您直接创建 `Thread` 对象。这可以解决一些问题，因为在一些 JVM 中，创建 `Thread` 是一项重量型的操作，重用现有 `Thread` 比创建新线程要容易得多。而在另一些 JVM 中，情况正好相反： `Thread` 是轻量型的，可以在需要时很容易地新建一个线程。当然，如果 Murphy 拥有自己的解决办法（他通常都会拥有），那么您无论使用哪种方法对于您最终将部署的平台都是不对的。

JSR-166 专家组在一定程度上预测到了这一情形。Java 开发人员无需直接创建 `Thread` ，他们引入了 `Executor` 接口，这是对创建新线程的一种抽象。如清单 3 所示， `Executor` 使您不必亲自对 `Thread` 对象执行 `new` 就能够创建新线程：

##### 清单 3. Executor

```
Executor exec = getAnExecutorFromSomeplace();
exec.execute(new Runnable() { ... });
```

显示更多

使用 `Executor` 的主要缺陷与我们在所有工厂中遇到的一样：工厂必须来自某个位置。不幸的是，与 CLR 不同，JVM 没有附带一个标准的 VM 级线程池。

`Executor` 类 *实际上* 充当着一个提供 `Executor` 实现实例的共同位置，但它只有 `new` 方法（例如用于创建新线程池）；它没有预先创建实例。所以您可以自行决定是否希望在代码中创建和使用 `Executor` 实例。（或者在某些情况下，您将能够使用所选的容器/平台提供的实例。）

### ExecutorService 随时可以使用

尽管不必担心 `Thread` 来自何处，但 `Executor` 接口缺乏 Java 开发人员可能期望的某种功能，比如结束一个用于生成结果的线程并以非阻塞方式等待结果可用。（这是桌面应用程序的一个常见需求，用户将执行需要访问数据库的 UI 操作，然后如果该操作花费了很长时间，可能希望在它完成之前取消它。）

对于此问题，JSR-166 专家创建了一个更加有用的抽象（ `ExecutorService` 接口），它将线程启动工厂建模为一个可集中控制的服务。例如，无需每执行一项任务就调用一次 `execute()` ， `ExecutorService` 可以接受一组任务并返回一个表示每项任务的未来结果的 *未来列表* 。

## 4. ScheduledExecutorServices

尽管 `ExecutorService` 接口非常有用，但某些任务仍需要以计划方式执行，比如以确定的时间间隔或在特定时间执行给定的任务。这就是 `ScheduledExecutorService` 的应用范围，它扩展了 `ExecutorService` 。

如果您的目标是创建一个每隔 5 秒跳一次的 “心跳” 命令，使用 `ScheduledExecutorService` 可以轻松实现，如清单 4 所示：

##### 清单 4. ScheduledExecutorService 模拟心跳

```
import java.util.concurrent.*;

public class Ping
{
    public static void main(String[] args)
    {
        ScheduledExecutorService ses =
            Executors.newScheduledThreadPool(1);
        Runnable pinger = new Runnable() {
            public void run() {
                System.out.println("PING!");
            }
        };
        ses.scheduleAtFixedRate(pinger, 5, 5, TimeUnit.SECONDS);
    }
}
```

显示更多

这项功能怎么样？不用过于担心线程，不用过于担心用户希望取消心跳时会发生什么，也不用明确地将线程标记为前台或后台；只需将所有的计划细节留给 `ScheduledExecutorService` 。

顺便说一下，如果用户希望取消心跳， `scheduleAtFixedRate` 调用将返回一个 `ScheduledFuture` 实例，它不仅封装了结果（如果有），还拥有一个 `cancel` 方法来关闭计划的操作。

## 5. Timeout 方法

为阻塞操作设置一个具体的超时值（以避免死锁）的能力是 `java.util.concurrent` 库相比起早期并发特性的一大进步，比如监控锁定。

这些方法几乎总是包含一个 `int`/`TimeUnit` 对，指示这些方法应该等待多长时间才释放控制权并将其返回给程序。它需要开发人员执行更多工作 — 如果没有获取锁，您将如何重新获取？ — 但结果几乎总是正确的：更少的死锁和更加适合生产的代码。（关于编写生产就绪代码的更多信息，请参见 参考资料 中 Michael Nygard 编写的 *Release It!* 。）

## 结束语

`java.util.concurrent` 包还包含了其他许多好用的实用程序，它们很好地扩展到了 Collections 之外，尤其是在 `.locks` 和 `.atomic` 包中。深入研究，您还将发现一些有用的控制结构，比如 `CyclicBarrier` 等。

与 Java 平台的许多其他方面一样，您无需费劲地查找可能非常有用的基础架构代码。在编写多线程代码时，请记住本文讨论的实用程序和 [上一篇文章](https://developer.ibm.com/zh/articles/j-5things4/) 中讨论的实用程序。