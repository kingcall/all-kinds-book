[toc]
## 单例设计模式

[扩展阅读](https://mp.weixin.qq.com/s/NZLe8Mc_Al3ne3rdEAxWXQ)
- 当只需要一个对象来协调整个系统的操作时，这种模式就非常有用,**它描述了如何解决重复出现的设计问题，如我们项目中的配置工具类,日志工具类等等**。
- 一个类只允许创建一个对象（或者实例），那这个类就是一个单例类，这种设计模式就叫作单例设计模式，简称单例模式。
- 单例模式的一些应用场景，比如，避免资源访问冲突、表示业务概念上的全局唯一类

### 如何实现单例
1. 构造函数需要是 private 访问权限的，这样才能避免外部通过 new 创建实例
2. 考虑对象创建时的线程安全问题
3. 考虑是否支持延迟加载
4. 考虑 getInstance() 性能是否高（是否加锁）

### 饿汉模式

```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static final IdGenerator instance = new IdGenerator();
  private IdGenerator() {}
  public static IdGenerator getInstance() {
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```
#### 优势
- 饿汉式的实现方式比较简单。在类加载的时候，instance 静态实例就已经创建并初始化好了，所以，instance 实例的创建过程是线程安全的。

#### 不足
- 因为不支持延迟加载，如果实例占用资源多（比如占用内存多）或初始化耗时长（比如需要加载各种配置文件），提前初始化实例是一种浪费资源的行为。最好的方法应该在用到的时候再去初始化

#### 扩展
- 如果初始化耗时长，那我们最好不要等到真正要用它的时候，才去执行这个耗时长的初始化过程，这会影响到系统的性能（比如，在响应客户端接口请求的时候，做这个初始化操作，会导致此请求的响应时间变长，甚至超时）
- 如果实例占用资源多，按照 fail-fast 的设计原则（有问题及早暴露），那我们也希望在程序启动时就将这个实例初始化好。如果资源不够，就会在程序启动的时候触发报错（比如 Java 中的 PermGen Space OOM），我们可以立即去修复。这样也能避免在程序运行一段时间后，突然因为初始化这个实例占用资源过多，导致系统崩溃，影响系统的可用性

### 懒汉模式
```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static IdGenerator instance;
  private IdGenerator() {}
  public static synchronized IdGenerator getInstance() {
    if (instance == null) {
      instance = new IdGenerator();
    }
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```
#### 优势
- 延迟加载
#### 不足
- getInstance() 这个方法加了一把大锁（synchronzed），导致这个函数的并发度很低。量化一下的话，并发度是 1，也就相当于串行操作了。而这个函数是在单例使用期间，一直会被调用。如果这个单例类偶尔会被用到，那这种实现方式还可以接受。但是，**如果频繁地用到，那频繁加锁、释放锁及并发度低等问题，会导致性能瓶颈**，这种实现方式就不可取了。

### 双重检测(double check)
- 饿汉式不支持延迟加载，懒汉式有性能问题，不支持高并发
- 一种既支持延迟加载、又支持高并发的单例实现方式，也就是双重检测实现方式
```

public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static IdGenerator instance;
  private IdGenerator() {}
  public static IdGenerator getInstance() {
    if (instance == null) {
      synchronized(IdGenerator.class) { // 此处为类级别的锁
        if (instance == null) {
          instance = new IdGenerator();
        }
      }
    }
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```
#### 优势
- 解决了懒汉模式的并发问题

#### 说明
- 有人说，这种实现方式有些问题。因为指令重排序，可能会导致 IdGenerator 对象被 new 出来，并且赋值给 instance 之后，还没来得及初始化（执行构造函数中的代码逻辑），就被另一个线程使用了
- 。要解决这个问题，我们需要给 instance 成员变量加上 volatile 关键字，禁止指令重排序才行。
- 实际上，只有很低版本的 Java 才会有这个问题。我们现在用的高版本的 Java 已经在 JDK 内部实现中解决了这个问题（解决的方法很简单，只要把对象 new 操作和初始化操作设计为原子操作，就自然能禁止重排序）。

### 双重检测优化
- 如果存在上面的指令重问题，那么就添加volatile，但是这个时候可以在**判断的时候将 volatile变量赋值给局部变量进行判断**
- 因为volatile修饰的静态变量访问比较慢，如果不用局部变量则getInstance需要多次访问instance变量，使用局部变量可以有一定的性能提升。
```
public class IdGeneratorDoubleCheckUpSingleton {
    private AtomicLong id = new AtomicLong(0);
    private static volatile IdGeneratorDoubleCheckUpSingleton instance;
    private IdGeneratorDoubleCheckUpSingleton() {}
    public static IdGeneratorDoubleCheckUpSingleton getInstance() {
        IdGeneratorDoubleCheckUpSingleton tmp = instance;
        if (tmp == null) {
            synchronized(IdGenerator.class) { // 此处为类级别的锁
                if (tmp == null) {
                    instance = new IdGeneratorDoubleCheckUpSingleton();
                }
            }
        }
        return instance;
    }
    public long getId() {
        return id.incrementAndGet();
    }
}
```

### 静态内部类
- 一种比双重检测更加简单的实现方法，那就是利用 Java 的静态内部类。它有点类似饿汉式，但又能做到了延迟加载
```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private IdGenerator() {}

  private static class SingletonHolder{
    private static final IdGenerator instance = new IdGenerator();
  }
  
  public static IdGenerator getInstance() {
    return SingletonHolder.instance;
  }
 
  public long getId() { 
    return id.incrementAndGet();
  }
}
```
- SingletonHolder 是一个静态内部类，当外部类 IdGenerator 被加载的时候，并不会加载 SingletonHolder
- 只有当调用 getInstance() 方法时，SingletonHolder 才会被加载，这个时候才会创建 instance

#### 优势
-  instance 的**唯一性、创建过程的线程安全性，都由 JVM 来保证**。
-  所以，这种实现方法既保证了线程安全，又能做到延迟加载

### 枚举实现
- 种实现方式通过** Java 枚举类型本身的特性**，保证了实例创建的线程安全性和实例的唯一性。
```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private IdGenerator() {}

  private static class SingletonHolder{
    private static final IdGenerator instance = new IdGenerator();
  }
  
  public static IdGenerator getInstance() {
    return SingletonHolder.instance;
  }
 
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

#### 优势
- 最简单的实现
## 例子
### 处理资源访问冲突
```

public class Logger {
  private FileWriter writer;
  
  public Logger() {
    File file = new File("/Users/wangzheng/log.txt");
    writer = new FileWriter(file, true); //true表示追加写入
  }
  
  public void log(String message) {
    writer.write(mesasge);
  }
}

// Logger类的应用示例：
public class UserController {
  private Logger logger = new Logger();
  
  public void login(String username, String password) {
    // ...省略业务逻辑代码...
    logger.log(username + " logined!");
  }
}

public class OrderController {
  private Logger logger = new Logger();
  
  public void create(OrderVo order) {
    // ...省略业务逻辑代码...
    logger.log("Created an order: " + order.toString());
  }
}
```
#### 解决方案
##### 类级别锁
- 除了使用类级别锁之外
##### 分布式锁
- 分布式锁是最常听到的一种解决方案。不过，实现一个安全可靠、无 bug、高性能的分布式锁，并不是件容易的事情
##### 并发队列

- 除此之外，并发队列（比如 Java 中的 BlockingQueue）也可以解决这个问题：多个线程同时往并发队列里写日志，一个单独的线程负责将并发队列中的数据，写入到日志文件

##### 单例模式
- 单例模式相对于之前类级别锁的好处是，不用创建那么多 Logger 对象，一方面节省内存空间，另一方面节省系统文件句柄（对于操作系统来说，文件句柄也是一种资源，不能随便浪费

### 表示全局唯一类
- 从业务概念上，如果有些数据在系统中只应保存一份，那就比较适合设计为单例类
- 比如，配置信息类。在系统中，我们只有一个配置文件，当配置文件被加载到内存之后，以对象的形式存在，也理所应当只有一份

## 单例存在的问题
- 单例模式书写简洁、使用方便，在代码中，我们不需要创建对象，直接通过类似 IdGenerator.getInstance().getId() 这样的方法来调用就可以了
- 但是，这种使用方法有点类似硬编码（hardcode），会带来诸多问题

### 单例对 OOP 特性的支持不友好
- OOP 的四大特性是封装、抽象、继承、多态。单例这种设计模式对于其中的抽象、继承、多态都支持得不好。

#### 违反了基于最小接口编程的原则
```
public class Order {
  public void create(...) {
    //...
    long id = IdGenerator.getInstance().getId();
    //...
  }
}

public class User {
  public void create(...) {
    // ...
    long id = IdGenerator.getInstance().getId();
    //...
  }
}
```
- 如果未来某一天，我们希望针对不同的业务采用不同的 ID 生成算法。比如，订单 ID 和用户 ID 采用不同的 ID 生成器来生成。为了应对这个需求变化，我们需要修改所有用到 IdGenerator 类的地方，这样代码的改动就会比较大。
```
public class Order {
  public void create(...) {
    //...
    long id = IdGenerator.getInstance().getId();
    // 需要将上面一行代码，替换为下面一行代码
    long id = OrderIdGenerator.getIntance().getId();
    //...
  }
}

public class User {
  public void create(...) {
    // ...
    long id = IdGenerator.getInstance().getId();
    // 需要将上面一行代码，替换为下面一行代码
    long id = UserIdGenerator.getIntance().getId();
  }
}
```
#### 单例对继承、多态特性的支持也不友好
- 单例对继承、多态特性的支持也不友好。这里我之所以会用“不友好”这个词，而非“完全不支持”
- 是因为从理论上来讲，单例类也可以被继承、也可以实现多态，只是实现起来会非常奇怪，会导致代码的可读性变差。
- 不明白设计意图的人，看到这样的设计，会觉得莫名其妙。
- 所以，一旦你选择将某个类设计成到单例类，也就意味着**放弃了继承和多态和抽象**这两个强有力的面向对象特性，也就相当于损失了可以应对未来需求变化的扩展性

### 单例会隐藏类之间的依赖关系
- 代码的可读性非常重要。在阅读代码的时候，我们希望一眼就能看出类与类之间的依赖关系，搞清楚这个类依赖了哪些外部类。
- 通过构造函数、参数传递等方式声明的类之间的依赖关系，我们通过查看函数的定义，就能很容易识别出来。
- 但是，单例类不需要显示创建、不需要依赖参数传递，在函数中直接调用就可以了。
- 如果代码比较复杂，这种调用关系就会非常隐蔽。在阅读代码的时候，我们就需要仔细查看每个函数的代码实现，才能知道这个类到底依赖了哪些单例类

### 单例对代码的扩展性不友好
- 单例类只能有一个对象实例。如果未来某一天，我们需要在代码中创建两个实例或多个实例，那就要对代码有比较大的改动
- 单例类在某些情况下会影响代码的扩展性、灵活性。所以，数据库连接池、线程池这类的资源池，最好还是不要设计成单例类。实际上，一些开源的数据库连接池、线程池也确实没有设计成单例类

### 单例对可测试性不好
- 单例模式的使用会影响到代码的可测试性。
- 如果单例类依赖比较重的外部资源，比如DB，我们在写单元测试的时候，希望能通过 mock 的方式将它替换掉。
- 而单例类这种硬编码式的使用方式，导致无法实现 mock 替换

#### 例子
- 如果单例类持有成员变量（比如 IdGenerator 中的 id 成员变量），那它实际上相当于一种全局变量，被所有的代码共享。如果这个全局变量是一个可变全局变量，也就是说，它的成员变量是可以被修改的，那我们在编写单元测试的时候，还需要注意不同测试用例之间，修改了单例类中的同一个成员变量的值，从而导致测试结果互相影响的问题。

### 单例不支持有参数的构造函数
- 单例不支持有参数的构造函数，比如我们创建一个连接池的单例对象，我们没法通过参数来指定连接池的大小

## 单例的替代方案
### 静态方法
```
// 静态方法实现方式
public class IdGenerator {
  private static AtomicLong id = new AtomicLong(0);
  
  public static long getId() { 
    return id.incrementAndGet();
  }
}
// 使用举例
long id = IdGenerator.getId();
```
- 静态方法这种实现思路，并不能解决我们之前提到的问题。实际上，它比单例更加不灵活，比如，它无法支持延迟加载

### 基于新的使用方式
- 我们将单例生成的对象，作为参数传递给函数（也可以通过构造函数传递给类的成员变量），可以解决单例隐藏类之间依赖关系的问题。
- 不过，对于单例存在的其他问题，比如对 OOP 特性、扩展性、可测性不友好等问题，还是无法解决(个人觉得其实解决了了)

```
public class StudentIdGenerator implements IdGenerator {
    private AtomicLong id = new AtomicLong(0);
    private static final StudentIdGenerator instance = new StudentIdGenerator();
    @Override
    public long getId() {
        if (id.incrementAndGet()%2 == 0){
            return id.get();
        }else {
            return id.incrementAndGet();
        }
    }
    public static StudentIdGenerator getInstance() {
        return instance;
    }
}

public class UserIdGenerator implements IdGenerator {
    private AtomicLong id = new AtomicLong(0);
    private static final UserIdGenerator instance = new UserIdGenerator();
    @Override
    public long getId() {
        return id.incrementAndGet();
    }
    public static UserIdGenerator getInstance() {
        return instance;
    }
}

public class Application {
    public static void main(String[] args) {
        test1(StudentIdGenerator.getInstance());
        test1(UserIdGenerator.getInstance());
    }

    public static void test1(IdGenerator idGenerator){
        while (true){
            System.out.println(String.format("%s -------> %d",idGenerator.getClass().getName(),idGenerator.getId()));
        }
    }

    public static void test2(IdGenerator idGenerator){
        while (true){
            System.out.println(String.format("%s -------> %d",idGenerator.getClass().getName(),idGenerator.getId()));
        }
    }

}
```

### 终极解决方案
- 们可能要从根上，寻找其他方式来实现全局唯一类。
- 实际上，类对象的全局唯一性可以通过多种不同的方式来保证。
- 我们既可以通过单例模式来强制保证
- 也可以通过工厂模式、IOC 容器（比如 Spring IOC 容器）来保证
- 还可以通过程序员自己来保证（自己在编写代码的时候自己保证不要创建两个类对象）。这就类似 Java 中内存对象的释放由 JVM 来负责，而 C++ 中由程序员自己负责，道理是一样的

## 多例模式
- “单例”指的是，一个类只能创建一个对象。对应地，“多例”指的就是，一个类可以创建多个对象，但是个数是有限制的，比如只能创建 3 个对象
- 多例模式的理解方式有点类似工厂模式。它跟工厂模式的不同之处是，多例模式创建的对象都是同一个类的对象，而工厂模式创建的是不同子类的对象
- 多例的实现也比较简单，通过一个 Map 来存储对象类型和对象之间的对应关系，来控制对象的个数。

### 例子
#### 饿汉模式
```
public class BackendServerHungery {
    private long serverNo;
    private String serverAddress;

    private static final int SERVER_COUNT = 3;
    private static final Map<Long, BackendServerHungery> serverInstances = new HashMap<>();

    static {
        serverInstances.put(1L, new BackendServerHungery(1L, "192.134.22.138:8080"));
        serverInstances.put(2L, new BackendServerHungery(2L, "192.134.22.139:8080"));
        serverInstances.put(3L, new BackendServerHungery(3L, "192.134.22.140:8080"));
    }

    private BackendServerHungery(long serverNo, String serverAddress) {
        this.serverNo = serverNo;
        this.serverAddress = serverAddress;
    }

    public BackendServerHungery getInstance(long serverNo) {
        return serverInstances.get(serverNo);
    }

    public BackendServerHungery getRandomInstance() {
        Random r = new Random();
        int no = r.nextInt(SERVER_COUNT)+1;
        return serverInstances.get(no);
    }
}
```
#### 懒汉模式
```
public class BackendServerLazy {
    private static final int SERVER_COUNT = 3;
    private static final Map<Integer, BackendServerLazy> serverInstances = new HashMap<>();

    private BackendServerLazy() {
    }


    public static synchronized BackendServerLazy getRandomInstance() {
        Random r = new Random();
        int no = r.nextInt(SERVER_COUNT)+1;
        BackendServerLazy instance= serverInstances.get(no);
        if (instance==null){
            serverInstances.put(no, new BackendServerLazy());
        }
        instance=serverInstances.get(no);
        return instance;
    }

    public static void main(String[] args) {
        Set<BackendServerLazy> instances=new HashSet<> ();
        int times = 1;
        while (instances.size() < 3){
            instances.add(BackendServerLazy.getRandomInstance());
            System.out.println(String.format("第 %d 次",times));
            times++;
        }
        System.out.println("三个实例已经全部初始化");
    }
}
```


## 扩展
### 进程下的单例
- 平时我们说的都是进程下的单例
- 单例类对象的唯一性的作用范围并非进程，而是类加载器（Class Loader)

### 线程下的单例
#### 自定义实现
```
/**
 * 在代码中，我们通过一个 HashMap 来存储对象，其中 key 是线程 ID，value 是对象。这样我们就可以做到，不同的线程对应不同的对象，同一个线程只能对应一个对象
 * 实际上，Java 语言本身提供了 ThreadLocal 工具类，可以更加轻松地实现线程唯一单例。不过，ThreadLocal 底层实现原理也是基于下面代码中所示的 HashMap。
 */
public class IdGenerator {
    private AtomicLong id = new AtomicLong(0);

    private static final ConcurrentHashMap<Long, IdGenerator> instances
            = new ConcurrentHashMap<>();

    private IdGenerator() {}

    public static IdGenerator getInstance() {
        Long currentThreadId = Thread.currentThread().getId();
        instances.putIfAbsent(currentThreadId, new IdGenerator());
        return instances.get(currentThreadId);
    }

    public long getId() {
        return id.incrementAndGet();
    }
}

```
#### ThreadLocal 实现
```
import java.util.concurrent.atomic.AtomicLong;
public class IdgeneratorThreadLocal {
    private AtomicLong id = new AtomicLong(0);
    static ThreadLocal<IdgeneratorThreadLocal> instances = new ThreadLocal<>();

    private IdgeneratorThreadLocal() {}

    public static IdgeneratorThreadLocal getInstance() {
        instances.set(new IdgeneratorThreadLocal());
        return instances.get();
    }

    public long getId() {
        return id.incrementAndGet();
    }
}

```
### 集群下的单例

- 我们需要把这个单例对象序列化并存储到外部共享存储区（比如文件）。进程在使用这个单例对象的时候，需要先从外部共享存储区中将它读取到内存，并反序列化成对象，然后再使用，使用完成之后还需要再存储回外部共享存储区
- 为了保证任何时刻，在**进程间都只有一份对象存在，一个进程在获取到对象之后，需要对对象加锁**，避免其他进程再将其获取。在进程使用完这个对象之后，还需要显式地将对象从内存中删除，并且释放对对象的加锁

