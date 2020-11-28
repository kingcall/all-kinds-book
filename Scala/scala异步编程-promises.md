## promises的含义
- 如果说futures是为了一个还没有存在的结果，而当成一种只读占位符的对象类型去创建，那么promise就被认为是一个**可写的**，可以实现一个future的**单一赋值容器**。这就是说，promise通过这种success方法可以成功去实现一个带有值的future。相反的，因为一个失败的promise通过failure方法就会实现一个带有异常的future
- 一个promise p通过p.future方式返回future。 这个futrue对象被指定到promise p。根据这种实现方式，可能就会出现p.future与p相同的情况。
```
import scala.concurrent.{ Future, Promise }
import scala.concurrent.ExecutionContext.Implicits.global

val p = Promise[T]()
val f = p.future

val producer = Future {
  val r = produceSomething()
  p success r
  continueDoingSomethingUnrelated()
}

val consumer = Future {
  startDoingSomething()
  f onSuccess {
    case r => doSomethingWithResult()
  }
}
```

Promises也能通过一个complete方法来实现，这个方法采用了一个potential value Try[T]，这个值要么是一个类型为Failure[Throwable]的失败的结果值，要么是一个类型为Success[T]的成功的结果值。

```
val f = Future { 1 }
val p = promise[Int]

p completeWith f

p.future onSuccess {
  case x => println(x)
}
```