[toc]
## 接口隔离原则
- 接口隔离原则的英文翻译是“ Interface Segregation Principle”，缩写为 ISP。Robert Martin 在 SOLID 原则中是这样定义它的：“Clients should not be forced to depend upon interfaces that they do not use。”直译成中文的话就是：客户端不应该强迫依赖它不需要的接口。其中的“客户端”，可以理解为接口的调用者或者使用者

## 接口的理解
1. 一组 API 接口集
2. 合单个 API 接口或函数
3. OOP 中的接口概念

## 例子
### 一组 API 接口集
#### 删除用户
```
public interface UserService {
  boolean register(String cellphone, String password);
  boolean login(String cellphone, String password);
  UserInfo getUserInfoById(long id);
  UserInfo getUserInfoByCellphone(String cellphone);
}

public class UserServiceImpl implements UserService {
  //...
}
```
- 现在我们的后台管理系统要实现删除用户的功能，希望用户系统提供一个删除用户的接口，删除用户是一个非常慎重的操作，我们只希望通过后台管理系统来执行，所以这个接口只限于给后台管理系统使用。
- 如果我们把它放到 UserService 中，那所有使用到UserService的系统，都可以调用这个接口。不加限制地被其他业务系统调用，就有可能导致误删用户

##### 解决方案
- 好的解决方案是从架构设计的层面，通过接口鉴权的方式来限制接口的调用。
- 不过，如果暂时没有鉴权框架来支持，我们还可以从代码设计的层面，尽量避免接口被误用
- 参照接口隔离原则，调用者不应该强迫依赖它不需要的接口，将删除接口单独放到另外一个接口 RestrictedUserService 中，然后将 RestrictedUserService 只打包提供给后台管理系统来使用
```
public interface UserService {
  boolean register(String cellphone, String password);
  boolean login(String cellphone, String password);
  UserInfo getUserInfoById(long id);
  UserInfo getUserInfoByCellphone(String cellphone);
}

public interface RestrictedUserService {
  boolean deleteUserByCellphone(String cellphone);
  boolean deleteUserById(long id);
}

public class UserServiceImpl implements UserService, RestrictedUserService {
  // ...省略实现代码...
}
```
### 单个 API 接口或函数
- 那接口隔离原则就可以理解为：函数的设计要功能单一，不要将多个不同的功能逻辑在一个函数中实现

#### 统计功能
```

public class Statistics {
  private Long max;
  private Long min;
  private Long average;
  private Long sum;
  private Long percentile99;
  private Long percentile999;
  //...省略constructor/getter/setter等方法...
}

public Statistics count(Collection<Long> dataSet) {
  Statistics statistics = new Statistics();
  //...省略计算逻辑...计算，max,min,avg,
  return statistics;
}
```
- 上面的代码中，count() 函数的功能不够单一，包含很多不同的统计功能，比如，求最大值、最小值、平均值等等。
- 按照接口隔离原则，我们应该把 count() 函数拆成几个更小粒度的函数

##### 解决方案
```

public Long max(Collection<Long> dataSet) { //... }
public Long min(Collection<Long> dataSet) { //... } 
public Long average(Colletion<Long> dataSet) { //... }
// ...省略其他统计函数...
```
##### 说明
- 在某种意义上讲，count() 函数也不能算是职责不够单一，毕竟它做的事情只跟统计相关，对每个统计需求，Statistics 定义的那几个统计信息都有涉及，那 count() 函数的设计就是合理的
- 如果每个统计需求只涉及 Statistics罗列的统计信息中一部分，比如，有的只需要用到max、min、average这三类统计信息，有的只需要用到 average、sum。而 count() 函数每次都会把所有的统计信息计算一遍，就会做很多无用功，势必影响代码的性能

### OOP 中的接口
- 假设我们要为配置信息添加一个页面展示的功能，如果我们将展示抽象成一个接口，然后每个配置信息去实现这个接口就可以了，如果有一天我们想把统计信息也展示，只需要让统计信息实现这个接口即可。

### java 中的类库
- java.util.concurrent 并发包提供了 AtomicInteger 这样一个原子类，其中有一个函数 getAndIncrement() 是这样定义的：给整数增加一，并且返回未増之前的值。我的问题是，这个函数的设计是否符合单一职责原则和接口隔离原则
- 纯理论分析，这么设计是不符合“接口隔离”原则的，毕竟，get是一个操作，increment是另一个操作。
- 结合具体场景，Atomic类的设计目的是保证操作的原子性，专门看了一下AtomicInteger的源码，发现没有单独的 increment 方法，然后思考了一下线程同步时的问题，场景需要保证 get 与 increment 中间不插入其他操作，否则函数的正确性无法保证，从场景的角度，它又是符合原则的。
- 通过调用者如何使用接口来间接地判定。如果调用者只使用部分接口或接口的部分功能，那接口的设计就不够职责单一

### 总结
- 在设计微服务或者类库接口的时候，如果部分接口只被部分调用者使用，那我们就需要将这部分接口隔离出来，单独给对应的调用者使用，而不是强迫其他调用者也依赖这部分不会被用到的接口
- 接口隔离的目标是微课扩展性，但是过程却是单一职责，因为职责越单一，复用性和扩展性越好，在单一职责的体系下，需求变更其实就是添加其他的职责——满足了OCP

### 对比SRP
- 接口隔离原则跟单一职责原则有点类似，不过稍微还是有点区别。
- 单一职责原则针对的是模块、类、接口的设计。
- 而接口隔离原则相对于单一职责原则，一方面它更侧重于接口的设计，另一方面它的思考的角度不同。
- 它提供了一种判断接口是否职责单一的标准：**通过调用者如何使用接口来间接地判定**。如果调用者只使用部分接口或接口的部分功能，那接口的设计就不够职责单一