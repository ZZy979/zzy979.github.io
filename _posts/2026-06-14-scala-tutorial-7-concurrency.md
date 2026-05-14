---
title: Scala基础教程 第7节 并发
date: 2026-05-14 10:53:53 +0800
categories: [Scala]
tags: [scala, concurrency, future, synchronized block]
---
<https://docs.scala-lang.org/overviews/core/futures.html>

<https://docs.scala-lang.org/overviews/scala-book/futures.html>

## 7.1 引言
在Scala中编写并发代码时，仍然可以使用原生的Java线程。但应该优先选择Scala的`Future`，它能够使并发编程容易得多。

[Future[T]](https://www.scala-lang.org/api/2.13.18/scala/concurrent/Future.html)表示一个当前可能尚不可用、但会在**未来**某个时刻可用（也可能抛出异常）的`T`类型的值。`Future`提供了一种以高效、非阻塞的方式来执行并行操作的机制。

典型的`Future`示例如下所示：

```scala
val inverseFuture: Future[Matrix] = Future {
  fatMatrix.inverse() // non-blocking long lasting computation
}(executionContext)
```

这段代码将逆矩阵计算`fatMatrix.inverse()`的执行委托给一个`ExecutionContext`，并将计算结果封装在`inverseFuture`中。

习惯上会将执行上下文作为隐式参数传递：

```scala
implicit val ec: ExecutionContext = ...
val inverseFuture: Future[Matrix] = Future {
  fatMatrix.inverse()
} // ec is implicitly passed
```

## 7.2 执行上下文
[ExecutionContext](https://www.scala-lang.org/api/2.13.18/scala/concurrent/ExecutionContext.html)类似于`java.util.concurrent.Executor`，可以在新线程或线程池中执行计算。

`scala.concurrent`包自带了一个`ExecutionContext`实现——一个全局静态线程池。也可以将Java `Executor`转换为`ExecutionContext`。还可以通过扩展`ExecutionContext`特质来实现自己的执行上下文。

### 7.2.1 全局执行上下文
`ExecutionContext.global`是基于[ForkJoinPool](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html)实现的执行上下文，对于大多数情况应该足够了。`ForkJoinPool`管理着一些线程，线程的最大数量称为**并行度**(parallelism level)。默认情况下，全局执行上下文会将其底层`ForkJoinPool`的并行度设置为可用处理器数量（即`Runtime.availableProcessors()`）。可以通过设置以下VM系统属性覆盖默认配置：
* `scala.concurrent.context.numThreads`：并行度，可以是具体数值（如`4`）或者处理器数量的倍数（如`x2`），默认为`x1`。
* `scala.concurrent.context.minThreads`：并行度最小值，默认为`1`。
* `scala.concurrent.context.maxThreads`：并行度最大值，默认为`x1`。

要使用全局执行上下文，只需添加导入：

```scala
import scala.concurrent.ExecutionContext.Implicits.global
```

或者声明一个隐式值：

```scala
implicit val ec = ExecutionContext.global
```

### 7.2.2 包装Java执行器
可以使用`ExecutionContext.fromExecutor()`方法将Java `Executor`包装为执行上下文。例如：

```scala
implicit val ec = ExecutionContext.fromExecutor(new ThreadPoolExecutor(...))
```

## 7.3 Future
`Future`是持有可能在未来的某个时刻可用的值的对象。这个值通常是某种异步计算的结果：
* 如果计算尚未完成，则称future **未完成**。
* 如果计算已经完成，则称future **已完成**。

已完成的future有两种形式：
* 如果计算产生了结果，则称future **成功完成**。
* 如果计算抛出了异常，则称future **失败**。

Future有一个重要的特性：它**只能被赋值一次**。一旦future对象被赋予了一个值或异常，它就永远不能被覆盖了。

注：实际上`Future`特质只有获取结果的方法，设置结果的方法来自`Promise`特质（类似于Java中的`CompletableFuture`），见7.5节。

### 7.3.1 创建Future
创建`Future`对象最简单的方式是调用`Future.apply()`方法。该方法会启动异步计算，并返回一个持有计算结果的`Future`对象（这个过程本身是非阻塞的）。一旦`Future`对象已完成，其结果就可用了。

考虑下面的示例。假设要使用某个社交网络的API来获取指定用户的好友列表：

```scala
import scala.concurrent._
import ExecutionContext.Implicits.global

val session = socialNetwork.createSessionFor("user", credentials) // hypothetical
val friends: Future[List[Friend]] = Future {
  session.getFriends()
}
```

获取用户好友列表的方法`getFriends()`需要通过网络发送请求，可能耗时较长，因此应该以异步方式执行。`Future`正是用于这一目的。一旦服务器返回响应，`Future`对象`friends`中的好友列表就可用了。如果该方法抛出异常（例如网络错误），`friends`对象将会失败，并包含异常信息。

下面是另一个示例。假设你有一个文本文件，希望查找特定关键词首次出现的位置。由于从磁盘读取文件内容时可能会阻塞，因此应该异步执行这项任务。

```scala
import scala.io.Source

val firstOccurrence: Future[Int] = Future {
  val source = Source.fromFile("myText.txt")
  source.toSeq.indexOfSlice("myKeyword")
}
```

注：`Future`的伴生对象还提供了一些方法，能够将多个future归约为单个future
* `sequence()` 将`Iterable[Future[T]]`转换为`Future[Iterable[T]]`
* `traverse()` 使用函数`T => Future[S]`将`Iterable[T]`转换为`Future[Iterable[S]]`
* `firstCompletedOf()` 返回一个future，包含给定future序列中第一个完成的future的结果
* `find()` 返回一个future，包含给定future序列中第一个成功完成且结果满足谓词的future的结果
* `reduceLeft()` 使用函数`(T, T) => T`将`Iterable[Future[T]]`归约为`Future[T]`
* `foldLeft()` 类似于`reduceLeft()`，但可以指定起始值

### 7.3.2 回调
当`Future`完成时，我们需要使用其结果做进一步处理。

一种方式是一直等待直到future完成，之后使用其结果来继续计算。这需要使用`Future[T]`类的以下方法：
* `isCompleted` 如果future已完成则返回`true`。
* `value` 返回future的当前值，类型为`Option[Try[T]]`（参见6.8.3节）：
  * 如果未完成为`None`
  * 如果成功完成则为`Some(Success(result))`
  * 如果失败则为`Some(Failure(exception))`

注：与`java.util.concurrent.Future`的`get()`方法不同，Scala `Future`的`value`方法是非阻塞的。

例如：

```scala
// wait until the future is completed
while (!fut.isCompleted) {
  Thread.sleep(100)
}

// now fut.value must not be None
fut.value.get match {
  case Success(result) => println(s"successfully completed with $result")
  case Failure(exception) => println(s"failed with $exception")
}
```

另外也可以使用`Await`，见7.4.2节。

不过，从性能角度来看，更好的做法是采用**非阻塞**的方式——在future上注册**回调**(callback)。一旦future已完成，这个回调就会被**异步地**调用。如果在注册回调时future已经完成，那么回调可能被异步执行，也可能在当前线程上被同步执行。

注册回调最通用的方式是使用`onComplete()`方法，该方法接受一个`Try[T] => U`类型的回调函数。如果future成功完成，则使用一个`Success[T]`类型的值调用回调函数；如果失败，则为`Failure[T]`类型的值。可以在同一个future上注册多个回调。

回到社交网络示例，假设通过`getRecentPosts()`方法获取用户最近发布的帖子列表并打印出来，该方法返回一个`List[String]`。

```scala
import scala.util.{Success, Failure}

val recentPosts: Future[List[String]] = Future {
  session.getRecentPosts()
}

recentPosts.onComplete {
  case Success(posts) => for (post <- posts) println(post)
  case Failure(e) => println("An error has occurred: " + e.getMessage)
}
```

`onComplete`方法能够处理成功和失败两种情况。如果只需处理成功的结果，可以使用`foreach()`方法，该方法接受一个`T => U`类型的回调函数，只有在成功时才会被调用。（注：`fut.foreach(f)`等价于`fut.onComplete(_.foreach(f))`，也等价于`for (x <- fut) f(x)`）

例如：

```scala
recentPosts.foreach { posts =>
  for (post <- posts) println(post)
}
```

这等价于以下`for`循环：

```scala
for {
  posts <- recentPosts
  post <- posts
} println(post)
```

注意，回调是在future对象完成后的**某个时间**由**某个线程**执行的。不能保证它会被执行计算的线程或创建回调的线程调用。另外，回调的执行顺序也是不确定的，可能同时并发执行。这意味着在回调中访问共享变量需要同步。

下面列出了回调的语义：
1. 注册`onComplete()`回调可以确保最终在future完成后被调用。
2. 注册`foreach()`回调与`onComplete()`具有相同的语义，区别是只有当future成功完成时才会被调用。
3. 在已完成的future上注册回调可以确保回调最终被执行（同1）。
4. 如果在一个future上注册了多个回调，其执行顺序是不确定的，可能会并发执行。不过，特定的`ExecutionContext`实现可能会产生确定的顺序。
5. 如果某个回调抛出异常，其他回调正常执行。
6. 如果某个回调永不结束（例如包含无限循环），则其他回调可能根本不会执行。可能会阻塞的回调必须使用`blocking`结构（见7.4.1节）。
7. 一旦执行完，回调就会从future对象中删除。

### 7.3.3 组合方法
回调机制能够在future完成时执行后续计算。但是，它有时不太方便，导致代码很笨重。

例如，假设有一个货币交易服务API，你想购买美元，但仅当有利可图（例如汇率低于某个预期值）时才购买。可以使用回调实现：

```scala
val rateQuote = Future {
  connection.getCurrentValue(USD)
}

for (quote <- rateQuote) {
  val purchase = Future {
    if (isProfitable(quote)) connection.buy(amount, quote)
    else throw new Exception("not profitable")
  }

  for (amount <- purchase)
    println("Purchased " + amount + " USD")
}
```

这段代码首先创建了一个获取当前汇率报价的future `rateQuote`。当成功完成后，在其回调（`for`循环等价于`foreach()`）中创建了另一个future `purchase`，决定是否购买，如果有利可图则发送购买请求。最后，当`purchase`完成后（即决定购买且成功），在标准输出打印一条通知消息。

这虽然可行，但不太方便。首先，必须嵌套回调。如果想在`purchase`完成后卖出其他货币，就必须再嵌套一层，使代码过于笨重且难以理解。其次，`purchase`仅在当前回调的作用域中可见。这意味着后续代码无法向其注册其他回调（例如卖出其他货币）。

出于这些原因，`Future`提供了组合方法。这些方法都是函数式的，每个方法都返回一个由当前future派生出的新future。

注：这些组合方法类似于Java `CompletableFuture`提供的组合方法，参见[《Java核心技术》笔记 卷I 第12章]({% post_url 2025-01-12-java-note-v1ch12-concurrency %}) 12.7.2节。

#### map
一个基本的组合方法是`map()`，接受一个`T => S`类型的函数，返回一个`Future[S]`。一旦原future成功完成，新future用映射后的值完成；如果原future失败，或者映射函数抛出异常，则新future也失败。（注：`fut.map(f)`等价于`for (x <- fut) yield f(x)`）

下面使用`map()`重写前面的示例：

```scala
val purchase = rateQuote.map { quote =>
  if (isProfitable(quote)) connection.buy(amount, quote)
  else throw new Exception("not profitable")
}

for (amount <- purchase)
  println("Purchased " + amount + " USD")
```

通过在`rateQuote`上使用`map()`，我们消除了一个回调和嵌套。如果想继续卖出其他货币，只需对`purchase`使用`map()`即可。

如果`isProfitable()`返回`false`，从而引发异常，则`purchase`会失败。另外，假设`getCurrentValue()`抛出异常导致`rateQuote`失败，`purchase`也会以相同的异常失败。

总之，`map()`返回的future
* `Success(r)` → `Success(f(r))`
* `Failure(e)` → `Failure(e)`
* 函数`f`抛出异常`e2` → `Failure(e2)`

这种异常传播语义也存在于其他组合方法。

#### flapMap/withFilter
`Future`的设计目标之一是能够用于`for`推导式，这是通过`flatMap()`和`withFilter()`方法实现的：
* `flatMap()`方法接受一个`T => Future[S]`类型的函数，将结果值映射到一个新的future `g`；然后返回一个`Future[S]`，该future在`g`完成后完成。
* `withFilter()`/`filter()`方法接受一个`T => Boolean`类型的函数，返回一个`Future[T]`。如果原future的结果满足谓词，则新future以相同的结果成功，否则以`NoSuchElementException`失败。

注：一般地，
* `for`推导式

```scala
for {
  x <- fut1
  y <- fut2
} yield f(x, y)
```

等价于

```scala
fut1.flatMap { x =>
  fut2.map(y => f(x, y))
}
```

以及

```scala
fut1.zipWith(fut2)(f)
```

* 带守卫的`for`推导式

```scala
for {
  x <- fut1
  y <- fut2
  if p(x, y)
} yield f(x, y)
```

等价于

```scala
fut1.flatMap { x =>
  fut2.withFilter(y => p(x, y))
    .map(y => f(x, y))
}
```

回到货币交易的例子。假设我们想把美元兑换成瑞士法郎(CHF)，必须获取两种货币的汇率报价，然后决定是否购买。下面的例子在`for`推导式中使用了`flatMap()`和`withFilter()`：

```scala
val usdQuote = Future { connection.getCurrentValue(USD) }
val chfQuote = Future { connection.getCurrentValue(CHF) }

val purchase = for {
  usd <- usdQuote
  chf <- chfQuote
  if isProfitable(usd, chf)
} yield connection.buy(amount, chf)

for (amount <- purchase)
  println("Purchased " + amount + " CHF")
```

只有当`usdQuote`和`chfQuote`都完成时，`purchase`才会开始计算。

#### recover
由于future可能包含两种类型的值（计算结果和异常），因此需要处理异常的组合方法。

货币示例中的`connection.buy()`方法接受两个参数：购买金额和期望报价，返回购买的金额。如果在此期间报价发生了变化，则抛出`QuoteChangedException`。如果希望`purchase`在这种情况下包含0而不是异常，可以使用组合方法`recover()`：

```scala
val purchase: Future[Int] = rateQuote.map { quote =>
  connection.buy(amount, quote)
}.recover {
  case e: QuoteChangedException => 0
}
```

`recover()`方法接受一个`PartialFunction[Throwable, T]`类型的偏函数`pf`（见2.4.10节），返回一个`Future[T]`。如果原future成功完成，则新future包含相同的结果；否则将`pf`应用于使原future失败的`Throwable`对象（尝试“恢复”）。如果`pf`对其有定义，则新future以该函数值成功，否则以相同的异常失败。

`recoverWith()`方法接受一个`PartialFunction[Throwable, Future[T]]`类型的偏函数`pf`，返回一个`Future[T]`。如果原future成功完成，则新future包含相同的结果；否则将`pf`应用于使原future失败的`Throwable`对象。如果`pf`对其有定义，则新future用`pf`返回的future的结果完成，否则仍然失败。它与`recover()`的关系类似于`flatMap()`与`map()`的关系。

#### fallbackTo
组合方法`fallbackTo()`接受一个`Future[T]`类型的参数，返回一个`Future[T]`。如果原future成功完成，则新future包含相同的结果，否则包含参数future的成功结果。如果两个future都失败，则新future以原future的异常失败。

下面的示例尝试打印美元的当前值，如果获取失败则打印瑞士法郎的值：

```scala
val usdQuote = Future(connection.getCurrentValue(USD))
  .map(usd => "Value: " + usd + "$")
val chfQuote = Future(connection.getCurrentValue(CHF))
  .map(chf => "Value: " + chf + "CHF")
val anyQuote = usdQuote.fallbackTo(chfQuote)

anyQuote.foreach(println)
```

#### andThen
组合方法`andThen()`仅用于副作用，当future完成时调用给定的偏函数（参数是`Try[T]`），然后返回包含与原future相同结果的future（因此可以链式调用）。这确保了多个`andThen()`调用是有序的。下面的示例将社交网络中的最新帖子存储到集合，然后打印到屏幕：

```scala
val allPosts = mutable.Set[String]()

Future(session.getRecentPosts())
  .andThen { case Success(posts) => allPosts ++= posts }
  .andThen { case _ => for (post <- allPosts) println(post) }
```

#### failed
`failed`方法返回一个`Future[Throwable]`。如果原future失败，则新future以异常对象成功，否则以`NoSuchElementException`失败。例如：

```scala
val f = Future(2 / 0)
for (exc <- f.failed) println(exc)
```

## 7.4 阻塞
`Future`通常是异步的，不会阻塞底层执行线程。但是，在某些情况下阻塞是必要的。有两种形式的阻塞：
* 在future内部阻塞执行线程
* 在future外部等待它完成

### 7.4.1 在Future内部阻塞
可以使用`blocking`结构向`ExecutionContext`通知其中的代码可能会阻塞，这可以提高性能或避免死锁。例如：

```scala
import scala.concurrent.{Future, blocking}

Future {
  blocking {
    myLock.lock()
    // ...
  }
}
```

但是，其实现完全由执行上下文决定。有些执行上下文（如全局执行上下文）通过`ManagedBlocker`实现`blocking`，而有些执行上下文（如固定线程池）会将其忽略。

阻塞代码也可能抛出异常。在这种情况下，异常被转发给调用者。

### 7.4.2 在Future外部阻塞
为了性能和防止死锁，强烈建议不要在future上阻塞，回调和组合方法是使用其结果的首选方式。但是，在某些情况下阻塞可能是必要的。

在前面的货币交易示例中，一个需要阻塞的地方是在程序的末尾，以确保所有future都已完成（程序不会自动等待）。例如：

```scala
import scala.concurrent._
import scala.concurrent.duration._

object AwaitPurchase {
  def main(args: Array[String]): Unit = {
    val purchase = ...

    Await.result(purchase, 5.seconds)
    println("Purchased " + purchase.value + " USD")
  }
}
```

`Await.result()`等待future完成并返回其结果。如果future失败，则异常被转发给调用者。第二个参数是`Duration`对象，指定最大等待时间。如果超过最大等待时间则抛出`TimeoutException`。

注：表达式`5.seconds`中的`seconds`是`DurationConversions`特质定义的方法。`Int`没有该方法，而是由实现该特质的隐式类`DurationInt`提供的。

`Await.ready()`类似于`result()`，但不获取结果，而是返回future对象本身。

`Future`特质扩展了`Awaitable`特质，包含`read()`和`result()`方法。但这些方法只被执行上下文调用，客户端不应该调用。

## 7.5 Promise
除了`Future.apply()`方法，还可以使用**promise**来创建future对象。

Future被定义为异步计算结果的**只读**容器，而promise可以被视为**可写的**单次赋值容器，用于完成future。可以使用`Promise`特质的`success()`方法使future成功完成，也可以使用`failure()`方法使future失败。

`Promise`的`future`方法返回与其关联的future对象。默认实现类`DefaultPromise`同时实现了`Promise`和`Future`特质，其`future`方法返回自身。

考虑下面的生产者-消费者示例，生产者使用promise将值传递给消费者：生产者向promise `p`写入值，消费者从与`p`关联的future `f`读取值。

```scala
val p = Promise[T]()
val f = p.future

val producer = Future {
  val r = produceSomething()
  if (isValid(r))
    p.success(r)
  else
    p.failure(new SomeException)
}

val consumer = Future {
  for (r <- f) {
    doSomethingWith(r)
  }
}
```

Promise只能**单次赋值**，重复调用`success()`或`failure()`会抛出`IllegalStateException`。

Promise也可以用`complete()`方法完成，该方法接受一个`Try[T]`类型的值。实际上，`success(r)`等价于`complete(Success(r))`，`failure(e)`等价于`complete(Failure(e))`。

使用promise的以上方法以及future的组合方法（无副作用）编写的程序是**确定性的**(deterministic)。这意味着如果没有抛出异常，无论并行执行如何调度，程序的结果将始终是相同的。在某些情况下，可能需要只在promise尚未完成的情况下完成它（例如，有几个在不同的future中执行的HTTP请求，客户端只对第一个HTTP响应感兴趣）。因此promise提供了`tryComplete()`、`trySuccess()`和`tryFailure()`方法。但使用这些方法会导致程序不是确定性的，而是取决于执行调度。

`completeWith()`方法用另一个future完成promise。当future完成后，promise也会使用其结果完成。

## 7.6 同步块
在Java中可以使用`synchronized`关键字实现同步块和同步方法。在Scala中也可以实现同步。

Scala同步块

```scala
obj.synchronized {
  // ...
}
```

等价于Java代码

```java
synchronized (obj) {
    // ...
}
```

Scala同步方法

```scala
def method(): Unit = synchronized {
  // ...
}
```

等价于Java代码

```java
public synchronized void method() {
    // ...
}
```

下面是一个示例：

```scala
class Counter {
  private var value = 0

  def increment(): Unit = synchronized {
    value += 1
  }

  def getValue: Int = synchronized {
    value
  }
}

val futures = (1 to 100).map { _ =>
  Future {
    counter.increment()
    counter.getValue
  }
}
val result = Future.reduceLeft(futures)(_ + _)
println(Await.result(result, 5.seconds))  // 5050
```
