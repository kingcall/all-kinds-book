[toc]
# scala 异步编程
- Scala Future 和 Promise类提供了进行异步处理的强大方式，包括组织和排序事件的非阻塞方式。

## 异步事件的处理方式
- 阻塞：一个等待事件的协调线程。
- 非阻塞：事件向应用程序生成某种形式的通知，而没有线程显式等待它。

## Future
- 其基本思想很简单，所谓Future，指的是一类占位符对象，用于指代某些尚未完成的计算的结果。一般来说，由Future指代的计算都是并行执行的，计算完毕后可另行获取相关计算结果
- 若该计算过程正常结束，或中途抛出异常，我们就说该Future已就位。
- 为了在语法和概念层面更加简明扼要地使用这些回调，Scala还提供了flatMap、foreach和filter等算子，使得我们能够以非阻塞的方式对future进行组合
- Future的一个重要属性在于它只能被赋值一次。一旦给定了某个值或某个异常，future对象就变成了不可变对象——无法再被改写。
### Future的就位分为两种情况
- 当Future带着某个值就位时，我们就说该Future携带计算结果成功就位- - 当Future因对应计算过程抛出异常而就绪，我们就说这个Future因该异常而失败。

### 回调处理
- 默认情况下，future和promise并不采用一般的阻塞操作，而是依赖回调进行非阻塞操作
- 在许多future的实现中，一旦future的client对future的结果感兴趣，它不得不阻塞它自己的计算直到future完成——然后才能使用future的值继续它自己的计算。虽然这在Scala的Future API中是允许的，但是从性能的角度来看更好的办法是一种完全非阻塞的方法
#### OnComplete方法
- 注册回调最通常的形式是使用OnComplete方法，即创建一个Try[T] => U类型的回调函数。如果future成功完成，回调则会应用到Success[T]类型的值中，否则应用到 Failure[T] 类型的值中。
- 如果Try[T]获得一个值则它为Success[T] ，否则为Failure[T]的异常。
##### complete 处理
- onComplete方法一般在某种意义上它允许客户处理future计算出的成功或失败的结果而不仅仅是对于处理成功的结果
```
val f: Future[List[String]] = Future {
  session.getRecentPosts
}

f onComplete {
  case Success(posts) => for (post <- posts) println(post)
  case Failure(t) => println("An error has occured: " + t.getMessage)
}
```

##### Success 结果
- 对于仅仅处理成功的结果，onSuccess 回调使用如下(该回调以一个偏函数未参数)
```
val f: Future[List[String]] = Future {
  session.getRecentPosts
}

f onSuccess {
  case posts => for (post <- posts) println(post)
}
```
##### onFailure
```
val f: Future[List[String]] = Future {
  session.getRecentPosts
}

f onFailure {
  case t => println("An error has occured: " + t.getMessage)
}

f onSuccess {
  case posts => for (post <- posts) println(post)
}
```
#### 多回调的意义
- 为一个future 对象注册多个回调函数
### 函数组合(combinators)
#### 常规的无限组合
- 代码过度嵌套，过于冗长，并且难以理解。
```
val rateQuote = Future {
  connection.getCurrentValue(USD)
}

rateQuote onSuccess { case quote =>
  val purchase = Future {
    if (isProfitable(quote)) connection.buy(amount, quote)
    else throw new Exception("not profitable")
  }

  purchase onSuccess {
    case _ => println("Purchased " + amount + " USD")
    val future=Future{
        ......
    }
  }
}
```
#### map
#### flatmap
### 投影(Projections)
## Blocking
## 异常(Exceptions)
- 当异步计算抛出未处理的异常时，与那些计算相关的futures就失败了。失败的futures存储了一个Throwable的实例，而不是返回值。Futures提供onFailure回调方法，它用一个PartialFunction去表示一个Throwable。