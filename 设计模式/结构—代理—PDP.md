[toc]
## 代理模式
- 它在不改变原始类（或叫被代理类）代码的情况下，**通过引入代理类来给原始类附加功能**
- 将框架代码和业务代码解耦，代理模式就派上用场

### 使用方法
#### 原始类存在接口
- 参照基于接口而非实现编程的设计思想，将原始类对象替换为代理类对象的时候，为了让代码改动尽量少，在刚刚的代理模式的代码实现中，代理类和原始类需要实现相同的接口

###### 优势
- 基于接口模式的优点在于更加灵活**，对于接口的所有子类都可以代理**，缺点在于不需要扩展的方法也需要进行代理。

#### 原始类不存在接口
- 如果原始类并没有定义接口，并且原始类代码并不是我们开发维护的（比如它来自一个第三方的类库），我们也没办法直接修改原始类，给它重新定义一个接口
- 对于这种外部类的扩展，我们一般都是采用继承的方式

###### 存在的问题
- 一方面，我们需要在代理类中，将原始类中的所有的方法，都重新实现一遍，并且为每个方法都附加相似的代码逻辑。
- 另一方面，如果要添加的附加功能的类有不止一个，我们需要针对每个类都创建一个代理类
- 如果50 个要添加附加功能的原始类，那我们就要创建 50 个对应的代理类。这会导致项目中类的个数成倍增加，增加了代码维护成本。并且，每个代理类中的代码都有点像模板式的“重复”代码，也增加了不必要的开发成本
- 其实主要问题还是面向实现的编程不够灵活
- 继承模式的优点在于**只需要针对需要扩展的方法进行代理**，缺点在于只能针对单一父类进行代理

#### 动态代理的原理解析
- 所谓动态代理（DynamicProxy），就是我**们不事先为每个原始类编写代理类**，而是在运行的时候，**动态地创建原始类对应的代理类**，然后在系统中**用代理类替换掉原始类**
- 因为 Java 语言本身就已经**提供了动态代理的语法**（实际上，动态代理底层依赖的就是 Java 的**反射语法**）
- jdk动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。
- 而cglib动态代理是利用asm开源包，对被代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
##### 应用
###### 业务系统的非功能性需求开发
- 代理模式最常用的一个应用场景就是，**在业务系统中开发一些非功能性需求**，比如：**监控、统计、鉴权、限流、事务、幂等、日志**。我们将这些附加功能与业务功能解耦，放到代理类中统一处理，让程序员只需要关注业务方面的开发

###### 代理模式在RPC中的应用
- RPC 框架也可以看作一种代理模式，GoF 的《设计模式》一书中把它称作远程代理。通过远程代理，将网络通信、数据编解码等细节隐藏起来。客户端在使用 RPC 服务的时候，就像使用本地函数一样，无需了解跟服务器交互的细节。除此之外，RPC 服务的开发者也只需要开发业务逻辑，就像开发本地使用的函数一样，不需要关注跟客户端的交互细节

###### 代理模式在缓存中的应用
- 假设我们要开发一个接口请求的缓存功能，对于某些接口请求，如果入参相同，在设定的过期时间内，直接返回缓存结果，而不用重新进行逻辑处理。比如，针对获取用户个人信息的需求，我们可以开发两个接口，一个支持缓存，一个支持实时查询。对于需要实时数据的需求，我们让其调用实时查询接口，对于不需要实时数据的需求，我们让其调用支持缓存的接口。

- 那如何来实现接口请求的缓存功能呢？最简单的实现方法就是刚刚我们讲到的，给每个需要支持缓存的查询需求都开发两个不同的接口，一个支持缓存，一个支持实时查询。但是，这样做显然增加了开发成本，而且会让代码看起来非常臃肿（接口个数成倍增加），也不方便缓存接口的集中管理（增加、删除缓存接口）、集中配置（比如配置每个接口缓存过期时间）

- 代理模式就能派上用场了，确切地说，应该是动态代理。如果是基于 Spring 框架来开发的话，那就可以在 AOP 切面中完成接口缓存的功能。在应用启动的时候，我们从配置文件中加载需要支持缓存的接口，以及相应的缓存策略（比如过期时间）等。当请求到来的时候，我们在 AOP 切面中拦截请求，如果请求中带有支持缓存的字段（比如 http://…?..&cached=true），我们便从缓存（内存缓存或者 Redis 缓存等）中获取数据直接返回

###### spring-aop
- Spring AOP 底层的实现原理就是基于动态代理。用户配置好需要给哪些类创建代理，并定义好在执行原始类的业务代码前后执行哪些附加功能。Spring 为这些类创建动态代理对象，并在 JVM 中替代原始类对象。原本在代码中执行的原始类的方法，被换作执行代理类的方法，也就实现了给原始类添加附加功能的目的

### 例子
#### 统计信息收集
##### 普通版本
```

public class UserController {
  //...省略其他属性和方法...
  private MetricsCollector metricsCollector; // 依赖注入

  public UserVo login(String telephone, String password) {
    long startTimestamp = System.currentTimeMillis();

    // ... 省略login逻辑...

    long endTimeStamp = System.currentTimeMillis();
    long responseTime = endTimeStamp - startTimestamp;
    RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
    metricsCollector.recordRequest(requestInfo);

    //...返回UserVo数据...
  }

  public UserVo register(String telephone, String password) {
    long startTimestamp = System.currentTimeMillis();

    // ... 省略register逻辑...

    long endTimeStamp = System.currentTimeMillis();
    long responseTime = endTimeStamp - startTimestamp;
    RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
    metricsCollector.recordRequest(requestInfo);

    //...返回UserVo数据...
  }
}
```
- 第一，性能计数器框架代码侵入到业务代码中，跟业务代码高度耦合。如果未来需要替换这个框架，那替换的成本会比较大。
- 第二，收集接口请求的代码跟业务代码无关，本就不应该放到一个类中。业务类最好职责更加单一，只聚焦业务处理。

##### 代理类版本
```
public class UserControllerProxy implements IUserController {
    private MetricsCollector metricsCollector;
    private UserController userController;

    public UserControllerProxy(UserController userController) {
        this.userController = userController;
        this.metricsCollector = new MetricsCollector();
    }

    @Override
    public UserVo login(UserVo userVo) {
        long startTimestamp = System.currentTimeMillis();
        // 委托
        UserVo userVoResult = userController.login(userVo);
        long endTimeStamp = System.currentTimeMillis();
        long responseTime = endTimeStamp - startTimestamp;
        RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
        metricsCollector.recordRequest(requestInfo);
        return userVoResult;
    }

    @Override
    public UserVo register(UserVo userVo) {
        long startTimestamp = System.currentTimeMillis();
        // 委托
        UserVo userVoResult = userController.register(userVo);
        long endTimeStamp = System.currentTimeMillis();
        long responseTime = endTimeStamp - startTimestamp;
        RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
        metricsCollector.recordRequest(requestInfo);

        return userVoResult;
    }
}
```

##### 基于实现类的代理版本
```
public class UserControllerProxyByextend extends UserController {
    private MetricsCollector metricsCollector;
    public UserControllerProxyByextend() {
        this.metricsCollector = new MetricsCollector();
    }

    public UserVo login(UserVo userVo) {
        long startTimestamp = System.currentTimeMillis();

        UserVo userVoResult = super.login(userVo);

        long endTimeStamp = System.currentTimeMillis();
        long responseTime = endTimeStamp - startTimestamp;
        RequestInfo requestInfo = new RequestInfo("login", responseTime, startTimestamp);
        metricsCollector.recordRequest(requestInfo);

        return userVoResult;
    }

    public UserVo register(UserVo userVo) {
        long startTimestamp = System.currentTimeMillis();

        UserVo userVoResult = super.register(userVo);

        long endTimeStamp = System.currentTimeMillis();
        long responseTime = endTimeStamp - startTimestamp;
        RequestInfo requestInfo = new RequestInfo("register", responseTime, startTimestamp);
        metricsCollector.recordRequest(requestInfo);

        return userVo;
    }
}

```

