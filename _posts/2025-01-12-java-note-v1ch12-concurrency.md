---
title: 《Java核心技术》笔记 卷I 第12章 并发
date: 2025-01-12 11:16:39 +0800
categories: [Java, Core Java]
tags: [java, concurrency, thread, lock, condition variable, intrinsic lock, synchronized method, synchronized block, atomic, thread-local, blocking queue, concurrent hash map, thread pool, future, process]
---
**多任务**(multitasking)是操作系统可以（看起来）在同一时刻运行多个程序的能力。例如，在编辑邮件的同时可以打印文件。操作系统会为每个进程分配CPU时间片，给人并行处理的感觉。

多线程在更低一层扩展了多任务的概念：一个程序看起来在同时执行多个任务。每个任务在一个**线程**(thread)中执行，线程是控制线程(thread of control)的简称。可以同时运行多个线程的程序称为**多线程的**(multithreaded)。

**进程**(process)与**线程**有什么区别呢？本质的区别在于每个进程都拥有自己的一整套变量，线程则共享数据（内存空间）。这是有风险的。不过，共享变量使线程之间通信比进程之间通信更高效、更容易。此外，在有些操作系统中，线程比进程更轻量级，创建和销毁线程的开销比进程要小得多。

在实际中，多线程非常有用。例如，浏览器应该能够同时下载多个文件，Web服务器需要能够处理并发的请求，GUI程序用一个独立的线程收集用户界面事件。本章将介绍如何为Java应用添加多线程能力。

温馨提示：并发编程可能会变得相当复杂。本章涵盖了应用程序员可能需要的所有工具。然而，对于更复杂的系统级编程，建议参看更高级的文献，例如Java Concurrency in Practice (Addison-Wesley Professional, 2006)。

## 12.1 什么是线程
首先来看一个使用两个线程的简单程序。这个程序可以在银行账户之间转账。我们使用`Bank`类来存储给定数目的账户的余额，`transfer()`方法将一定金额从一个账户转移到另一个账户。具体实现见程序清单12-2。

第一个线程从账户0转账到账户1，第二个线程从账户2转账到账户3。

下面是在一个单独的线程中运行任务的步骤：

1.将任务代码放在一个实现了`Runnable`接口的类的`run()`方法中。由于`Runnable`是函数式接口，可以使用lambda表达式创建实例：

```java
Runnable r = () -> {
    // task code
};
```

2.由`Runnable`构造一个`Thread`对象：

```java
var t = new Thread(r);
```

3.启动线程：

```java
t.start();
```

下面的代码创建了一个单独的线程来完成转账：

```java
Runnable r = () -> {
    try {
        for (int i = 0; i < STEPS; i++) {
            double amount = MAX_AMOUNT * Math.random();
            bank.transfer(0, 1, amount);
            Thread.sleep((int) (DELAY * Math.random()));
        }
    }
    catch (InterruptedException e) {
    }
};
var t = new Thread(r);
t.start();
```

这个线程每次会（从账户0到账户1）转账随机金额，然后休眠随机延迟，重复给定的步数。

我们需要捕获`sleep()`方法可能抛出的`InterruptedException`异常，这个异常将在12.3.1节讨论。通常，中断用来请求终止一个线程。这里（`Runnable`对象的）`run()`方法会在发生`InterruptedException`异常时退出。

这个程序还会启动第二个线程，从账户2向账户3转账。运行这个程序时，将得到类似这样的输出：

```
Thread[Thread-1,5,main]     606.77 from 2 to 3 Total Balance:  400000.00
Thread[Thread-0,5,main]      98.99 from 0 to 1 Total Balance:  400000.00
Thread[Thread-1,5,main]     476.78 from 2 to 3 Total Balance:  400000.00
Thread[Thread-0,5,main]     653.64 from 0 to 1 Total Balance:  400000.00
Thread[Thread-1,5,main]     807.14 from 2 to 3 Total Balance:  400000.00
Thread[Thread-0,5,main]     481.49 from 0 to 1 Total Balance:  400000.00
Thread[Thread-0,5,main]     203.73 from 0 to 1Thread[Thread-1,5,main]     111.76 from 2 to 3 Total Balance:  400000.00
 Total Balance:  400000.00
Thread[Thread-1,5,main]     794.88 from 2 to 3 Total Balance:  400000.00
...
```

可以看到，两个线程的输出是交错的，这说明它们在并发运行。实际上，有时两个输出行也会交错，输出会更混乱。

完整代码见程序清单12-1。

注释：还可以通过创建`Thread`类的子类来定义线程，如下所示：

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // task code
    }
}
```

然后构造这个子类的对象并调用其`start()`方法。然而，现在不再推荐这种方法。应该将要并行运行的**任务**与运行**机制**解耦开。如果有很多任务，为每个任务分别创建一个单独的线程开销太大。相反，可以使用线程池，参见12.6.2节。

警告：**不要**调用`Thread`或`Runnable`对象的`run()`方法。这会在**同一个**线程中执行任务，而没有启动新线程。而应当调用`start()`方法，这会创建一个新线程来执行`run()`方法。

[程序清单12-1 threads/ThreadTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/threads/ThreadTest.java)

[程序清单12-2 threads/Bank.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/threads/Bank.java)

注：在输出中可能会看到总余额小于400000，之后又恢复正常，如下所示。

```
Thread[Thread-1,5,main]     526.79 from 2 to 3 Total Balance:  400000.00
Thread[Thread-0,5,main]     616.88 from 0 to 1 Total Balance:  400000.00
Thread[Thread-1,5,main]     290.31 from 2 to 3 Total Balance:  400000.00
Thread[Thread-0,5,main]Thread[Thread-1,5,main]     908.25 from 2 to 3 Total Balance:  399229.66
     770.34 from 0 to 1 Total Balance:  400000.00
Thread[Thread-1,5,main]     641.83 from 2 to 3 Total Balance:  400000.00
...
```

这是因为一个线程计算总余额（读`accounts`数组）时另一个线程可能正在转账（写`accounts`数组）。例如，线程1在累加`accounts[3]`时，线程2正执行到`accounts[2] -= 770.34`和`accounts[3] += 770.34`之间（即已从账户2扣款$770.34，但还未存入账户3），导致`getTotalBalance()`返回错误的值$399229.66（银行存款“丢失”了$770.34）。而线程1只会写入`accounts[0]`和`accounts[1]`、线程2只会写入`accounts[2]`和`accounts[3]`，不存在数据竞争。因此实际上存款并没有丢失，下一次计算总余额时又“恢复正常”。这种同步问题详见12.4节。

## 12.2 线程状态
线程可以有如下6种状态：
* New（新建）
* Runnable（可运行）
* Blocked（阻塞）
* Waiting（等待）
* Timed waiting（计时等待）
* Terminated（已终止）

下面几节分别对每种状态进行解释。

要确定一个线程的当前状态，只需调用`getState()`方法。

### 12.2.1 新建
当用`new`运算符创建一个线程时，这个线程还没有开始运行。这意味着它处于**新建**(new)状态。在线程可以运行之前还有一些基础工作要做。

### 12.2.2 可运行
一旦调用了`start()`方法，线程就处于**可运行**(runnable)状态。**一个可运行的线程可能正在运行也可能没有运行**（这就是为什么这个状态称为“可运行”而不是“正在运行”）。让线程运行的时间取决于操作系统。

正在运行的线程不一定始终保持运行。事实上，需要让运行中的线程偶尔暂停，使其他线程有机会运行。线程调度的细节依赖于操作系统提供的服务。抢占式(preemptive)调度系统给每个可运行线程一个时间片来执行任务。当时间片用完时，操作系统会抢占该线程，并给另一个线程运行的机会（见12.4.2节图）。选择下一个线程时，操作系统会考虑线程的优先级，详见12.3.5节。

在有多个处理器的机器上，每个处理器可以运行一个线程，因此可以有多个线程并行运行。当然，如果线程数多于处理器，调度器仍然需要分配时间片。

注：“并行”和“并发”的区别
* **并行**(parallel)是指在同一时刻，多个处理器或核心同时执行多个任务。
* **并发**(concurrency)是指在一个时间段内，多个任务在同一个处理器或核心上运行。并发不是真正意义上的“同时进行”，而是通过时间片轮转的方式，使用户感觉好像是多个任务在同时运行。

### 12.2.3 阻塞和等待
当线程处于**阻塞**(blocked)或**等待**(waiting)状态时，它暂时是不活动的。它不执行任何代码，并且消耗最少的资源。要由线程调度器重新激活这个线程。具体细节取决于它是怎样到达非活动状态的。
* 当线程试图获取一个内部对象锁（而不是`java.util.concurrent`库中的`Lock`），而这个锁当前被其他线程持有，该线程就会被**阻塞**（将在12.4.3节讨论`java.util.concurrent`锁，12.4.5节讨论内部对象锁）。当所有其他线程都释放了这个锁、并且线程调度器允许该线程持有这个锁时，它将变成非阻塞状态。
* 当线程等待另一个线程通知调度器某个条件时，它会进入**等待**状态（12.4.4节将讨论条件）。调用`Object.wait()`或`Thread.join()`方法，或者等待`java.util.concurrent`库中的`Lock`或`Condition`时，就会出现这种情况。在实际中，阻塞和等待状态之间的区别并不太重要。
* 有几个方法有超时参数。调用这些方法会导致线程进入**计时等待**(timed waiting)状态。这一状态将一直保持到超时过期或收到适当的通知。带有超时参数的方法包括`Thread.sleep()`和计时版的`Object.wait()`、`Thread.join()`、`Lock.tryLock()`以及`Condition.await()`。

下图展示了线程可能的状态以及状态之间的转移。

![线程状态](/assets/images/java-note-v1ch12-concurrency/线程状态.png)

### 12.2.4 已终止
线程会由于以下两个原因之一而被终止：
* `run()`方法正常退出
* 未捕获的异常终止了`run()`方法

可以调用`stop()`方法杀死一个线程，不过该方法已废弃，详见[Java Thread Primitive Deprecation](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/doc-files/threadPrimitiveDeprecation.html)。

## 12.3 线程属性
下面几节将讨论线程的各种属性，包括中断状态、守护线程、未捕获异常的处理器以及不应使用的一些遗留特性。

### 12.3.1 中断线程
当一个线程的`run()`方法返回时，这个线程将终止。在Java的最初版本中，还有一个`stop()`方法可以终止一个线程。但是，这个方法现在已经废弃。12.4.12节将讨论它被废弃的原因。

除了`stop()`方法，没有办法**强制**终止线程。`interrupt()`方法可以用来**请求**终止线程。

当对一个线程调用`interrupt()`方法时，就会设置线程的**中断状态**(interrupted status)。这是一个`boolean`标志。每个线程都应该不时检查它是否被中断。

要确定是否设置了中断状态，首先调用静态方法`Thread.currentThread()`获得当前线程，然后调用`isInterrupted()`方法：

```java
while (!Thread.currentThread().isInterrupted() && /* more work to do */) {
    // do more work
}
```

但是，如果线程被阻塞，就无法检查中断状态。这里就要引入`InterruptedException`。在一个被`sleep()`或`wait()`等调用阻塞的线程上调用`interrupt()`方法时，那个阻塞调用（`sleep()`或`wait()`）将被`InterruptedException`终止。

Java语言并没有要求被中断的线程应当终止。中断一个线程只是要引起它的注意。**被中断的线程可以决定如何响应中断。** 有些线程非常重要，应该处理这个异常然后继续执行。但更普遍的情况是，线程将中断解释为终止请求。这种线程的`run()`方法具有如下形式：

```java
Runnable r = () -> {
    try {
        ...
        while (!Thread.currentThread().isInterrupted() && /* more work to do */) {
            // do more work
        }
    }
    catch (InterruptedException e) {
        // thread was interrupted during sleep or wait
    }
    finally {
        // cleanup, if required
    }
    // exiting the run method terminates the thread
};
```

如果在每次循环之后调用`sleep()`（或其他可中断方法），`isInterrupted()`检查既没有必要也没有用处。因为如果在设置了中断状态时调用`sleep()`方法，线程不会休眠，而是会清除中断状态（！）并抛出`InterruptedException`。因此，如果循环调用了`sleep()`，不要检查中断状态，而是捕获`InterruptedException`，如下所示：

```java
Runnable r = () -> {
    try {
        ...
        while (/* more work to do */) {
            // do more work
            Thread.sleep(delay);
        }
    }
    catch (InterruptedException e) {
        // thread was interrupted during sleep
    }
    finally {
        // cleanup, if required
    }
    // exiting the run method terminates the thread
};
```

注释：有两个非常类似的方法`interrupted()`和`isInterrupted()`。`interrupted()`是一个静态方法，检查**当前**线程是否被中断，并**清除**该线程的中断状态。另一方面，`isInterrupted()`是一个实例方法，可用于检查任意线程是否被中断，调用它不会改变中断状态。（注：即`Thread.interrupted()`返回`Thread.currentThread().isInterrupted()`，之后将其置为`false`）

你可能会发现很多代码在很低的层次上抑制了`InterruptedException`，如下：

```java
void mySubTask() {
    ...
    try {
        sleep(delay);
    }
    catch (InterruptedException e) {}  // don't ignore!
    ...
}
```

不要这样做！如果想不出在`catch`子句中可以做什么，仍然有两个合理的选择：
* 在`catch`子句中调用`Thread.currentThread().interrupt()`来设置中断状态，之后调用者可以检测。
* 或者，更好的选择是用`throws InterruptedException`标记方法，并去掉`try`语句，之后调用者（或者最终的`run()`方法）可以捕获这个异常。

### 12.3.2 守护线程
可以通过调用`t.setDaemon(true)`将一个线程转换为**守护线程**(daemon thread)，该方法必须在线程启动之前调用。守护线程的唯一用途是为其他线程提供服务，例如计时器线程和清理过期缓存项的线程。当只剩下守护线程时，虚拟机就会退出，因为此时就没必要继续运行程序了。

### 12.3.3 线程名
默认情况下，线程有容易记住的名字，如`Thread-2`。可以用`setName()`方法设置名字：

```java
var t = new Thread(runnable);
t.setName("Web crawler");
```

### 12.3.4 未捕获异常的处理器
**线程的`run()`方法不能抛出任何检查型异常**，但是非检查型异常会导致线程终止。在这种情况下，线程会死亡。

但是，没有可以将异常传播到的`catch`子句（注：即使将`t.start()`调用放在`try`块中也无法捕获`run()`方法中抛出的异常）。而是在线程死亡之前，异常会被传递到一个未捕获异常的处理器。

这个处理器必须属于一个实现了`Thread.UncaughtExceptionHandler`接口的类。这个接口只有一个方法：

```java
void uncaughtException(Thread t, Throwable e);
```

可以用`setUncaughtExceptionHandler()`方法为任何线程安装一个处理器。也可以用`Thread`类的静态方法`setDefaultUncaughtExceptionHandler()`为所有线程安装一个默认处理器。例如，处理器可以使用日志API将未捕获异常的报告发送到日志文件。

如果没有安装默认处理器，则默认处理器为`null`。如果没有为单个线程安装处理器，那么处理器是该线程的`ThreadGroup`对象。

线程组是可以一起管理的线程的集合。`ThreadGroup`类实现了`Thread.UncaughtExceptionHandler`接口，其`uncaughtException()`方法执行以下操作：
1. 如果该线程组有父线程组，则调用父线程组的`uncaughtException()`方法。
2. 否则，如果`Thread.getDefaultUncaughtExceptionHandler()`方法返回一个非`null`的处理器，则调用该处理器。
3. 否则，如果`Throwable`参数是`ThreadDeath`的实例，则什么都不做。
4. 否则，将线程的名字以及`Throwable`参数的栈轨迹输出到`System.err`。

### 12.3.5 线程优先级
在Java中，每个线程有一个**优先级**(priority)。默认情况下，线程会继承创建它的那个线程的优先级。可以用`setPriority()`方法改变线程的优先级。可以将优先级设置为`MIN_PRIORITY`（定义为1）和`MAX_PRIORITY`（定义为10）之间的任何值。`NORM_PRIORITY`定义为5。

每当线程调度器有机会选择新线程时，会首先选择具有较高优先级的线程。但是，线程优先级**高度依赖于系统**。例如，Windows有7个优先级别，Java的一些优先级会映射到相同的操作系统优先级。在Linux的Oracle JVM中，线程优先级会被完全忽略，即所有线程都有相同的优先级。

在没有使用操作系统线程的Java早期版本中，线程优先级可能很有用。现在不应该再使用了。

## 12.4 同步
在大多数实际的多线程应用中，两个或多个线程需要共享对相同数据的访问。如果两个线程访问同一个对象，并分别调用了修改该对象状态的方法，这两个线程就会相互冲突。取决于线程访问数据的次序，可能会导致对象被破坏。这种情况通常称为**竞态条件**(race condition)。

### 12.4.1 竞态条件的一个例子
为了避免多线程破坏共享数据，必须了解如何**同步访问**(synchronize the access)。在本节中，将会看到如果没有使用同步会发生什么。下一节将介绍如何同步数据访问。

还是考虑我们模拟的银行。与12.1节中的例子不同，我们随机地选择转账的源账户和目标账户。

```java
Runnable r = () -> {
    try {
        while (true) {
            int toAccount = (int) (bank.size() * Math.random());
            double amount = MAX_AMOUNT * Math.random();
            bank.transfer(fromAccount, toAccount, amount);
            Thread.sleep((int) (DELAY * Math.random()));
        }
    }
    catch (InterruptedException e) {
    }
};
```

这个模拟程序运行时，我们不清楚在某一时刻某个账户中有多少钱，但是我们知道所有账户的总金额应该保持不变。这个程序永远不会结束，只能按Ctrl+C来终止程序。

下面是典型的输出：

```
...
Thread[Thread-11,5,main]     588.48 from 11 to 44 Total Balance:  100000.00
Thread[Thread-12,5,main]     976.11 from 12 to 22 Total Balance:  100000.00
Thread[Thread-14,5,main]     521.51 from 14 to 22 Total Balance:  100000.00
Thread[Thread-13,5,main]     359.89 from 13 to 81 Total Balance:  100000.00
...
Thread[Thread-36,5,main]     401.71 from 36 to 73 Total Balance:   99291.06
Thread[Thread-35,5,main]     691.46 from 35 to 77 Total Balance:   99291.06
Thread[Thread-37,5,main]      78.64 from 37 to 3 Total Balance:   99291.06
Thread[Thread-34,5,main]     197.11 from 34 to 69 Total Balance:   99291.06
Thread[Thread-36,5,main]      85.96 from 36 to 4 Total Balance:   99291.06
...
Thread[Thread-4,5,main]Thread[Thread-33,5,main]       7.31 from 31 to 32 Total Balance:   99979.24
     627.50 from 4 to 5 Total Balance:   99979.24
...
```

可以看到，这里出现了错误。对于最初的交易，银行余额保持在$100000，这是正确的。但是经过一段时间后，余额略有变化。运行这个程序时，错误可能很快就发生，也可能需要很长时间才发生。你可能不希望将辛苦挣来的钱存进这样的银行。

看你能否找出程序清单12-3和`Bank`类（程序清单12-2）的问题。下一节将揭开谜团。

[程序清单12-3 unsynch/UnsynchBankTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/unsynch/UnsynchBankTest.java)

### 12.4.2 竞态条件详解
当两个线程试图同时更新同一个账户时，就会出现上述问题。假设两个线程同时执行语句`accounts[to] += amount;`，问题在于这不是**原子**(atomic)操作。这个语句可能如下处理：
1. 将`accounts[to]`加载到寄存器。
2. 增加`amount`。
3. 将结果写回`accounts[to]`。

假设线程1执行了步骤1和2，然后被抢占。线程2被唤醒，并更新了同一个元素`accounts[to]`。然后，线程1被唤醒并执行步骤3。**这个动作会覆盖线程2所做的修改。** 这样一来，总金额就不再正确了（如下图所示）。

![两个线程同时访问](/assets/images/java-note-v1ch12-concurrency/两个线程同时访问.png)

注释：可以查看执行类中每条语句的虚拟机字节码。运行命令`javap -c -v threads.Bank`对Bank.class文件进行反编译（注：要在threads的上层目录中运行）。例如，语句`accounts[to] += amount;`被翻译为下面的字节码：

```
aload_0
getfield #7  // Field accounts:[D
iload_2
dup2
daload
dload_3
dadd
dastore
```

这些代码的含义无关紧要。重要的是这个自增语句是由多条指令组成的，执行这些指令的线程有可能在任何一条指令处被中断。

在具有多核的现代处理器上，出现这种问题的风险相当高。我们通过交错执行打印语句和更新余额的语句，提高在单核处理器上观察到这一问题的概率。如果删除打印语句，出问题的风险会降低，但并没有完全消失。这种错误可能几分钟、几小时或几天后才出现。**对于程序员而言，最糟糕的事情莫过于无规律出现错误。**

真正的问题是`transfer()`方法可能会在中间被中断。如果能够确保该方法在线程被抢占之前运行完成，那么银行账户对象的状态就永远不会被破坏。

### 12.4.3 锁对象
有两种机制可以防止代码块并发访问：`synchronized`关键字和`ReentrantLock`类。

使用`ReentrantLock`保护代码块的基本结构如下：

```java
myLock.lock(); // a ReentrantLock object
try {
    // critical section
}
finally {
    myLock.unlock(); // make sure the lock is unlocked even if an exception is thrown
}
```

这一结构确保**任何时刻只有一个线程进入临界区**(critical section)。一旦一个线程锁定了锁对象，任何其他线程都无法通过（这个锁对象的）`lock()`语句。当其他线程调用`lock()`时，就会被阻塞，直到第一个线程释放这个锁对象。

警告：把`unlock()`操作包在`finally`子句中是至关重要的。如果临界区中的代码抛出异常，锁必须被释放。否则，其他线程将永远阻塞。

注释：使用锁时，不能使用带资源的`try`语句（见7.2.5节）。首先，解锁方法名不是`close`。但即使将它重命名，带资源的`try`语句也无法正常工作。其首部期望声明一个新变量。但是**如果使用锁，应该让多个线程共享同一个变量**（通过参数或类字段）（注：如果每个线程分别创建一个锁对象，则无法起到保护临界区的效果）。

下面使用一个锁来保护`Bank`类的`transfer()`方法。

```java
public class Bank {
    private Lock bankLock = new ReentrantLock();
    ...
    public void transfer(int from, int to, double amount) {
        bankLock.lock();
        try {
            System.out.print(Thread.currentThread());
            accounts[from] -= amount;
            System.out.printf(" %10.2f from %d to %d", amount, from, to);
            accounts[to] += amount;
            System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
        }
        finally {
            bankLock.unlock();
        }
    }
}
```

假设线程1调用了`transfer()`，但是在执行结束前被抢占。线程2也调用了`transfer()`，但由于不能获得锁而在调用`lock()`时被阻塞。它必须等待线程1执行完`transfer()`方法并释放锁，然后才能开始运行（见下图）。

![非同步与同步线程的比较](/assets/images/java-note-v1ch12-concurrency/非同步与同步线程的比较.png)

尝试将加锁代码添加到`transfer()`方法并再次运行程序。这个程序可以一直运行下去，银行余额绝对不会出错。

[lock/Bank.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/lock/Bank.java)

[lock/LockBankTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/lock/LockBankTest.java)

注意每个`Bank`对象都有自己的锁对象。如果两个线程试图访问同一个`Bank`对象，那么锁可以用来保证串行化访问。但是，如果两个线程访问不同的`Bank`对象，每个线程会得到不同的锁，两个线程都不会阻塞——本应如此，因为在操作不同的`Bank`实例时，线程之间不会相互影响。

这个锁称为**可重入**(reentrant)锁，因为线程可以重复获得已拥有的锁。锁有一个**持有计数**(hold count)来跟踪对`lock()`方法的嵌套调用。线程每次调用`lock()`后都要调用`unlock()`来释放锁。由于这一特性，被锁保护的代码可以调用另一个使用相同的锁的方法。

例如，`transfer()`方法锁定了`bankLock`对象，此时持有计数为1。之后调用了`getTotalBalance()`方法，该方法也会锁定`bankLock`对象，此时持有计数为2。当`getTotalBalance()`方法退出时，持有计数变回1。当`transfer()`方法退出时，持有计数变为0，线程释放锁。

通常，你希望**保护更新或访问共享对象的代码块**，从而确保这些操作执行完之后其他线程才能使用同一个对象。

警告：要注意确保不能由于抛出异常而绕过临界区中的代码。如果在临界区结束之前抛出了异常，`finally`子句将释放锁，但是对象可能处于被破坏的状态。

注：
* 调用`lock()`可能会阻塞、不能被中断。`tryLock()`不会阻塞；带超时参数的`tryLock()`可以被中断（允许打破死锁）。`lockInterruptibly()`相当于超时设为无限的`tryLock()`。在等待条件时，`await()`也可以提供超时参数。
* 对于读多写少的场景，更适合使用读写锁`ReentrantReadWriteLock`。

### 12.4.4 条件对象
通常，线程进入临界区后却发现只有满足了某个条件之后它才能执行。可以使用**条件对象**(condition object)来管理那些已经获得了锁却不能做有用工作的线程。本节将介绍Java库中条件对象的实现（由于历史原因，条件对象经常被称为**条件变量**(condition variable)）。

现在来改进模拟的银行。我们不希望从资金不足的账户中转账。

注：本节与上一节中`transfer()`方法的区别：在上一节中，当余额不足时**直接退出**，即“如果余额足够则转账，否则什么都不做”，因此调用`transfer()`可能转账也可能不转账。而本节中，当余额不足时会**一直等待**，直到能够转账，因此调用`transfer()`早晚一定会完成转账。

注意不能使用下面这样的代码：

```java
if (bank.getBalance(from) >= amount) {
    // thread might be deactivated at this point
    bank.transfer(from, to, amount);
}
```

当前线程完全有可能在通过测试和调用`transfer()`之间被抢占。当这个线程再次运行时，账户余额可能已经低于取款金额。

注：对于`LockBankTest`这个特定的程序，实际上不会有这种情况，因为只有线程`i`会从账户`i`取款，其他线程只会向账户`i`存款。但是对于一般的程序来说，上述情况完全有可能发生。

必须确保在测试（检查余额）和转账动作之间没有其他线程修改余额。为此，可以使用锁来保护测试和转账动作：

```java
public void transfer(int from, int to, int amount) {
    bankLock.lock();
    try {
        while (accounts[from] < amount) {
            // wait
            ...
        }
        // transfer funds
        ...
    }
    finally {
        bankLock.unlock();
    }
}
```

现在，当账户余额不足时，线程会等待，直到其他线程向该账户存款。但是，这个线程刚刚锁定了`bankLock`，因此其他线程没有存款的机会。这里就要引入条件对象。

**一个锁对象可以有一个或多个关联的条件对象。** 可以用`newCondition()`方法获得一个条件对象。习惯上会给每个条件对象一个合适的名字来反映它所表示的条件。例如，在这里设置了一个条件对象来表示“余额充足”条件。（注：条件对象本身不包含测试条件，具体语义取决于使用它的线程）

```java
public class Bank {
    private Condition sufficientFunds;
    ...
    public Bank() {
        ...
        sufficientFunds = bankLock.newCondition();
    }
}
```

如果`transfer()`方法发现余额不足，就会调用

```java
sufficientFunds.await();
```

当前线程会暂停（进入等待状态）并**放弃锁**，使得另一个线程可以运行（希望能增加账户余额）。

调用`lock()`和`await()`存在本质上的不同。一旦一个线程调用了`await()`方法，就会进入这个条件的**等待集**(wait set)。当锁可用时，该线程并不会变为可运行状态，而是保持非活动状态，直到另一个线程在同一条件上调用`signalAll()`方法。

当另一个线程完成转账时，它应该调用

```java
sufficientFunds.signalAll();
```

这个调用会重新激活等待该条件的所有线程。当这些线程从等待集中移出时，它们再次变为可运行状态，调度器最终将再次激活它们。然后这些线程会尝试重新获取（条件对象关联的那个）锁。一旦锁可用，其中的某个线程将得到这个锁，并从`await()`调用返回。

此时，线程应当再次测试条件。不能保证现在一定满足条件——`signalAll()`方法仅仅是通知等待的线程：此时**可能满足条件**，但有必要再次检查。

注：线程调用`await()`时会自动释放锁，而不是通过`unlock()`显式释放的。同样地，被唤醒时会自动重新获取锁，而不是通过调用`lock()`。释放锁 → 进入等待状态 → 等待唤醒 → 重新获取锁，这整个过程都是在`await()`调用内部发生的，在此期间线程一直阻塞在`await()`调用。

注释：通常，`await()`调用应该放在如下形式的循环中

```java
while (!someCondition)
    condition.await();
```

至关重要的是，某个其他线程最终会调用`signalAll()`方法。当一个线程调用`await()`时，它没有办法重新激活自身，只能寄希望于其他线程。如果没有其他线程来重新激活这个等待的线程，它就再也不能运行了。这将导致死锁现象，程序将会挂起。

应该何时调用`signalAll()`？经验法则是：每当对象的状态发生可能有利于等待线程的变化（即可能使等待的条件满足）时，就调用`signalAll()`。在银行的例子中，每当完成转账时就调用`signalAll()`方法（因为账户余额增加了，“余额充足”这个条件可能会满足）：

```java
public void transfer(int from, int to, double amount) throws InterruptedException {
    bankLock.lock();
    try {
        while (accounts[from] < amount)
            sufficientFunds.await();
        // transfer funds
        sufficientFunds.signalAll();
    }
    finally {
        bankLock.unlock();
    }
}
```

注意，调用`signalAll()`不会立即激活一个等待的线程。它只是解除等待线程的阻塞，使其可以在当前线程释放锁之后竞争锁。

另一个方法`signal()`只随机选择等待集中的一个线程并解除其阻塞状态。这比`signalAll()`更高效，但也存在危险。如果随机选择的线程发现自己仍然不能运行，它就会再次阻塞。如果没有其他线程再次调用`signal()`，就会发生死锁。

警告：`Condition`必须与`Lock`配合使用。**只有当线程拥有一个条件对象的锁时，才能在这个条件上调用`await()`、`signalAll()`或`signal()`方法**（否则将抛出`IllegalMonitorStateException`）。
* 否则可能出现无限等待的情况。假设线程1判断条件为假，尚未调用`await()`，CPU时间片耗尽。线程2改变条件，调用`signalAll()`，实际上没有任何效果。线程1调用`await()`进入等待状态，但此时条件为真，因此不会有其他线程调用`signalAll()`唤醒线程1，线程1将无限等待。
* 根本原因：Java语言规范规定对等待集的操作必须是原子的，因此必须加锁。

如果运行程序清单12-4中的程序，你会注意到不再有任何错误。总余额永远是$100000，也没有账户会出现负余额。这个程序运行起来要慢一些——这是为实现同步机制所付出的代价。

[程序清单12-4 synch/Bank.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/synch/Bank.java)

[synch/SynchBankTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/synch/SynchBankTest.java)

在实际中，正确使用条件对象很有挑战性。应该首先考虑使用线程安全的集合（见12.5节）。

#### 小结
适合使用条件对象的场景：
* 线程1执行方法`workerThread()`，需要等待`someCondition`为`true`后才能运行。
* 线程2执行方法`notifierThread()`，其中执行的`changeCondition()` **可能**会使`someCondition`变为`true`。

条件对象的典型使用方式如下：

```java
public class ConditionExample {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void workerThread() throws InterruptedException {
        lock.lock();
        try {
            while (!someCondition) {
                condition.await();
            }
            doSomeWork();
        } finally {
            lock.unlock();
        }
    }

    public void notifierThread() {
        lock.lock();
        try {
            changeCondition();
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
```

### 12.4.5 synchronized关键字
在进一步深入之前，先对锁和条件对象的要点做一个总结：
* 锁用来保护代码段，一次只允许一个线程执行被保护的代码（临界区）。
* 锁管理试图进入临界区的线程。
* 一个锁可以有一个或多个关联的条件对象。
* 条件对象管理那些已经进入临界区但还不能运行的线程。

`Lock`和`Condition`接口为程序员提供了对加锁的高度控制。不过，大多数情况下并不需要那样的控制，可以使用一种Java语言内置的机制。从1.0版开始，Java中的**每个对象**都有一个**内部锁**(intrinsic lock)。**如果一个方法用`synchronized`关键字声明，那么对象内部锁将保护整个方法。** 要调用这个方法，线程必须获得对象内部锁。换句话说，

```java
public synchronized void method() {
    // method body
}
```

等价于

```java
public void method() {
    this.intrinsicLock.lock();
    try {
        // method body
    }
    finally {
        this.intrinsicLock.unlock();
    }
}
```

例如，可以简单地将`Bank`类的`transfer()`方法声明为`synchronized`，而不必使用显式的锁。

**对象内部锁只有一个关联的条件。** `Object`类的`wait()`方法将一个线程添加到等待集中，`notifyAll()`/`notify()`方法解除等待线程的阻塞。换句话说，调用`wait()`和`notifyAll()`分别等价于`intrinsicCondition.await()`和`intrinsicCondition.signalAll()`。

注释：`wait()`、`notifyAll()`和`notify()`是`Object`类的`final`方法。`Condition`方法必须命名为`await()`、`signalAll()`和`signal()`，从而不会与那些方法冲突。

注：
* 与`Condition`类似，**`wait()`、`notifyAll()`和`notify()`方法只能在同步方法或同步块内调用**，即调用线程必须拥有对象的内部锁。
* 对象内部锁是可重入的。

例如，可以如下实现`Bank`类：

```java
public class Bank {
    private double[] accounts;

    public synchronized void transfer(int from, int to, double amount) throws InterruptedException {
        while (accounts[from] < amount)
            wait(); // wait on intrinsic object lock's single condition
        accounts[from] -= amount;
        accounts[to] += amount;
        notifyAll(); // notify all threads waiting on the condition
    }

    public synchronized double getTotalBalance() {
        ...
    }
}
```

可以看到，使用`synchronized`关键字得到的代码要简洁得多。

提示：同步方法相当简单。但在使用`wait()`/`notifyAll()`之前，应该考虑使用线程安全的集合（见12.5节）。

将静态方法声明为`synchronized`也是合法的。如果调用这种方法，它会获得类对象的内部锁。

内部锁和条件对象存在一些限制。包括：
* 不能中断正在尝试获得锁的线程。
* 尝试获得锁时不能指定超时时间（但`Lock.tryLock()`可以）。
* 内部锁只有一个条件对象，这很低效。

在代码中应该使用`Lock`/`Condition`对象还是同步方法？下面是一些建议：
* 最好二者都不使用。在很多情况下，可以使用`java.util.concurrent`包中的某种机制，它会为你处理所有的加锁（例如12.5和12.6节）。还应当研究并行流（参见卷II第1章）。
* 如果`synchronized`关键字适合你的程序，就尽量使用它。这样可以减少编写的代码量和出错的概率。
* 如果确实需要`Lock`/`Condition`结构提供的额外能力，才使用它们。

程序清单12-5给出了使用同步方法实现的银行示例。

[程序清单12-5 synch2/Bank.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/synch2/Bank.java)

[synch2/SynchBankTest2.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/synch2/SynchBankTest2.java)

### 12.4.6 同步块
除了同步方法，还有另一种机制可以获得对象内部锁：**同步块**(synchronized block)。当线程进入如下形式的块时，就会获得`obj`的内部锁。

```java
synchronized (obj) {
    // critical section
}
```

有时会看到一些“专用”(ad hoc)锁，例如：

```java
public class Bank {
    private double[] accounts;
    private Object lock = new Object();
    ...
    public void transfer(int from, int to, double amount) {
        synchronized (lock) { // an ad-hoc lock
            accounts[from] -= amount;
            accounts[to] += amount;
        }
        System.out.println(...);
    }
}
```

在这里，创建`lock`对象只是为了使用其内部锁。

警告：
* 使用同步块时，不要使用字符串字面值作为锁：

```java
private final String lock = "LOCK";
...
synchronized (lock) { ... } // Don't lock on string literal!
```

如果这段代码在同一个程序中出现两次，锁将是同一个对象，因为字符串字面值会共享。这可能导致死锁。

* 另外，也不要使用基本类型包装器作为锁：

```java
private final Integer lock = new Integer(42); // Don't lock on wrappers
```

如果使用同一个数字两次，锁会意外地共享（小整数缓存，见5.4节）。

* 如果需要修改静态字段，锁定特定的类，而不是`getClass()`的返回值：

```java
synchronized (MyClass.class) { staticCounter++; } // OK
synchronized (getClass()) { staticCounter++; } // Don't
```

如果从子类调用包含这段代码的方法，`getClass()`会返回不同的`Class`对象。

* 一般来说，如果必须使用同步块，一定要了解你的锁对象！必须对所有受保护的访问路径使用相同的锁，而且别人不能使用你的锁。

注释：Java虚拟机为同步方法提供了内置支持。但是，同步块会被编译为很长的字节码序列来管理内部锁。

### 12.4.7 监视器概念
多年来，研究人员一直在寻找一种不要求程序员显式加锁就可以保证多线程安全性的方法。最成功的解决方案之一是**监视器**(monitor)概念。用Java的术语来讲，监视器有如下属性：
* 监视器是只包含私有字段的类。
* 该类的每个对象有一个关联的锁。
* 所有方法都由这个锁锁定。换句话说，如果调用`obj.method()`，那么在方法调用开始时会自动获得`obj`的锁，并在方法返回时自动释放这个锁。因为所有字段都是私有的，这可以确保在一个线程操作字段时，没有其他线程能够访问这些字段。
* 锁可以有任意多个关联的条件对象。

Java设计者以不太严格的方式适配了监视器概念。在Java中，监视器是通过对象内部锁和wait/notify机制实现的。如果一个方法用`synchronized`关键字声明，它表现得就像是一个监视器方法。

然而，Java对象在以下三个重要方面不同于监视器，这削弱了线程安全性：
* 字段不要求是`private`。
* 方法不要求是`synchronized`。
* 内部锁对客户端是可用的（通过同步块）。

### 12.4.8 volatile字段
有时，仅仅为了读写一两个实例字段就使用同步好像开销太大。遗憾的是，使用现代的处理器和编译器，出错的可能性很大。
* 有多处理器的计算机能够**在寄存器或缓存中暂时保存内存值**。其结果是，运行在不同处理器上的线程可能在同一个内存位置看到不同的值。
* 编译器可能**重新排序指令**以获得最大的吞吐量。指令重排不会改变代码语义，但是编译器假定只有当代码中有显式的修改指令时内存值才会改变。但是，内存值有可能被另一个线程改变。

如果使用锁来保护可能被多个线程访问的代码，那么不存在这些问题。编译器必须遵守锁的要求，在必要时刷新本地缓存，并且不能不适当地重排指令。详细的解释见JSR 133的Java内存模型和线程规范(<https://www.jcp.org/en/jsr/detail?id=133>)。更易懂的概述文章：[Fixing the Java Memory Model, Part 1](https://web.archive.org/web/20200704133829/https://www.ibm.com/developerworks/library/j-jtp02244/)。

注释：Brian Goetz创造了以下“同步格言”：“如果写入的变量接下来可能会被另一个线程读取，或者读取的变量可能已经被另一个线程写入，那么必须使用同步。”

`volatile`关键字为实例字段的同步访问提供了一种无锁机制。如果将一个字段声明为`volatile`（“易变的”），那么编译器和虚拟机就会考虑到该字段可能被另一个线程并发更新。

注：`volatile`只能修饰类字段，提供两方面的保证：
* 可见性：强制从主内存中读取和写入，确保一个线程对变量的修改对其他线程立即可见。
* 有序性：禁止指令重排，确保对该变量的读写操作按照代码顺序执行。

例如，假设有一个`boolean`标志`done`，由一个线程设置、另一个线程查询。你可以使用锁：

```java
private boolean done;
public synchronized boolean isDone() { return done; }
public synchronized void setDone() { done = true; }
```

但是对象内部锁不是只有这个字段才能使用。如果另一个线程已经对该对象加锁，`isDone()`和`setDone()`方法就会阻塞。也可以只为该字段使用一个单独的锁，但是这会很麻烦。

在这种情况下，将字段声明为`volatile`是合理的：

```java
private volatile boolean done;
public boolean isDone() { return done; }
public void setDone() { done = true; }
```

编译器会确保一个线程对`done`的修改对其他读取这个变量的线程是可见的。

警告：**`volatile`变量不能提供原子性。** 例如，方法

```java
public void flipDone() { done = !done; } // not atomic
```

不能保证翻转字段的值，因为无法确保读取、翻转和写入过程不被中断。

### 12.4.9 final变量
除了锁和`volatile`修饰符，还有一种情况可以安全地访问共享字段，即字段声明为`final`。考虑以下声明：

```java
final var accounts = new HashMap<String, Double>();
```

其他线程会在构造器完成之后才能看到`accounts`变量。如果不使用`final`，其他线程可能看到的是`null`。

当然，映射的操作并不是线程安全的。如果有多个线程读写这个映射，仍然需要同步。

### 12.4.10 原子性
`java.util.concurrent.atomic`包中有很多类使用高效的机器级指令（而不是锁）来保证对共享变量操作的**原子性**(atomicity)。例如，`AtomicLong`类提供了方法`incrementAndGet()`和`decrementAndGet()`，分别以原子方式将一个整数自增或自减。例如，可以如下安全地生成数值序列：

```java
public static AtomicLong nextNumber = new AtomicLong();

public void someThread() {
    long id = nextNumber.incrementAndGet();
    ...
}
```

`incrementAndGet()`方法以原子方式将`AtomicLong`自增，并返回自增后的值。也就是说，获得值、加1、设置值和生成新值的操作**不会被中断**。即使多个线程并发地访问同一个实例，也可以保证计算并返回正确的值。

注：
* 使用`long`共享变量的`++`运算符不是线程安全的，因为`++nextNumber`是非原子操作。
* Java中原子类的底层实现基于**CAS** (Compare-And-Swap)机制和`sun.misc.Unsafe`类。

此外还有以原子方式设置和增减值的方法。但是如果希望完成更复杂的更新，就必须使用`compareAndSet()`方法。例如，假设希望跟踪不同线程观察到的最大值，下面的代码是不可行的：

```java
public static AtomicLong largest = new AtomicLong();

public void someThread() {
    ...
    largest.set(Math.max(largest.get(), observed)); // ERROR--race condition!
}
```

这个更新不是原子的（由`get`、`max`和`set`三步组成）。而应当在一个循环中计算新值并使用`compareAndSet()`：

```java
do {
    var oldValue = largest.get();
    var newValue = Math.max(oldValue, observed);
} while (!largest.compareAndSet(oldValue, newValue));
```

`compareAndSet()`方法会比较`largest`的当前值与期望值(`oldValue`)是否相等。如果相等，就用新值替换旧值并返回`true`；否则返回`false`（这说明另一个线程在并发更新`largest`），在这种情况下，循环会**重试**，直到成功设置新值。这听上去有些麻烦，不过`compareAndSet()`方法会映射到一个处理器操作，比使用锁更快。

从Java 8起，可以提供一个lambda表达式来更新变量：

```java
largest.updateAndGet(x -> Math.max(x, observed));
```

或

```java
largest.accumulateAndGet(observed, Math::max);
```

注：`AtomicLong`类提供的方法如下表所示（方法名中`get`在前的返回旧值，`get`在后的返回新值）。

| 方法 | 描述 |
| --- | --- |
| `get()` | 返回当前值 |
| `set(v)` | 设置新值 |
| `getAndSet(v)` | 设置新值，并返回之前的值 |
| `compareAndSet(e, v)` | 如果当前值等于期望值`e`，则将值设置为`v`并返回`true`；否则返回`false` |
| `getAndAdd(d)` | 以原子方式完成`set(get() + d)`，返回旧值 |
| `getAndIncrement()` | 等价于`getAndAdd(1)` |
| `getAndDecrement()` | 等价于`getAndAdd(-1)` |
| `addAndGet(d)` | 以原子方式完成`set(get() + d)`，返回新值 |
| `incrementAndGet()` | 等价于`addAndGet(1)` |
| `decrementAndGet()` | 等价于`addAndGet(-1)` |
| `getAndUpdate(f)` | 以原子方式完成`set(f(get())`，返回旧值 |
| `updateAndGet(f)` | 以原子方式完成`set(f(get())`，返回新值 |
| `getAndAccumulate(x, f)` | 以原子方式完成`set(f(get(), x))`，返回旧值 |
| `accumulateAndGet(x, f)` | 以原子方式完成`set(f(get(), x))`，返回新值 |

注释：`AtomicInteger`、`AtomicIntegerArray`、`AtomicReference`等原子类也提供了这些方法。

注：乐观锁与悲观锁：

| 特性 | 乐观锁(optimistic locking) | 悲观锁(pessimistic locking) |
| --- | --- | --- |
| 核心思想 | 假设冲突很少发生，提交时检查冲突 | 假设冲突很可能发生，先加锁 |
| 实现方式 | CAS操作 | `synchronized`、`ReentrantLock`等 |
| 适用场景 | 并发冲突较少、读多写少 | 并发冲突频繁、写操作多 |
| 性能 | 高（不加锁） | 低（加锁开销） |
| 冲突处理 | 需要处理冲突（重试） | 直接避免冲突 |
| 实现复杂度 | 较高 | 较低 |
| 死锁风险 | 无 | 有 |

如果有大量线程访问同一个原子值，性能会受到影响，因为乐观更新（CAS操作）需要太多重试。`LongAdder`和`LongAccumulator`类解决了这个问题。

`LongAdder`由多个变量（加数）组成，其总和为当前值。对于只有当所有工作都完成之后才需要总和的值的情况，这种方法很高效，性能会有显著提升。如果预料会有大量竞争，就应该使用`LongAdder`而不是`AtomicLong`。调用`increment()`加1，`add()`增加指定值，`sum()`获取总和。

`LongAccumulator`将这种思想推广到任意的累加操作。在构造器中提供这个操作及其零元素，调用`accumulate()`加入新值，调用`get()`来获得当前值。下面的代码与`LongAdder`有同样的效果：

```java
var adder = new LongAccumulator(Long::sum, 0);
// in some thread...
adder.accumulate(value);
```

也可以选择不同的操作，例如计算最大值或最小值。一般来说，这个操作必须满足交换律和结合律。这意味着最终结果必须与中间值的组合顺序无关。

另外，`DoubleAdder`和`DoubleAccumulator`做法也相同，只不过处理的是`double`值。

### 12.4.11 死锁
锁和条件不能解决多线程中可能出现的所有问题。考虑下面的情况：
* 账户1：$200，账户2：$300
* 线程1：从账户1转账$300到账户2
* 线程2：从账户2转账$400到账户1

如下图所示，线程1和2都被阻塞，因为账户1和2中的余额都不足。这种情况称为**死锁**(deadlock)。

![死锁的情况](/assets/images/java-note-v1ch12-concurrency/死锁的情况.png)

在`SynchBankTest`程序中，死锁不会发生。因为每次至多转账$1000，而所有账户的初始余额都是$1000，因此在任意时刻，至少有一个账户的余额大于等于$1000。

但是如果把$1000的转账限制去掉，很快就会发生死锁。试试看，将`NACCOUNTS`设置为10、`MAX_AMOUNT`设置为`2 * INITIAL_BALANCE`，然后运行程序。程序运行一段时间后就会挂起。

提示：当程序挂起时，按Ctrl+\，将得到一个线程转储(thread dump)，它会列出所有线程。每个线程有一个栈轨迹，告诉你当前阻塞在哪里。另外，也可以运行`jconsole`并查看Threads面板。

导致死锁的另一种方式是让第i个线程负责向第i个账户存款，而不是从中取款。这样一来，有可能所有线程都集中到一个账户上，每个线程都试图从这个账户中取出大于该账户余额的钱。试试看，在`SynchBankTest`程序中，调用`transfer()`时交换`fromAccount`和`toAccount`。运行程序，会看到它几乎立即死锁。

还有一种很容易导致死锁的情况：在`SynchBankTest`程序中，将`signalAll()`方法改为`signal()`，会发现程序最终会挂起（将`NACCOUNTS`改小可以更快地看到结果）。与`signalAll()`不同，`signal()`方法只解除一个线程的阻塞。如果该线程不能继续运行，所有的线程可能都被阻塞。考虑下面发生死锁的场景：
* 账户1：$1990，所有其他账户：每个$990
* 线程1：从账户1转账$995到账户2（成功）
* 所有其他线程：从对应的账户转账$995到另一个账户（都被阻塞）
* 此时账户1：$995，账户2：$1985，所有其他账户：每个$990
* 线程1：调用`signal()`，随机选择了线程3将其唤醒
* 线程3：发现账户3余额不足，再次调用`await()`
* 线程1：从账户1转账$997到账户2，调用`await()`

现在，所有线程都被阻塞，系统死锁。这里的罪魁祸首是`signal()`调用，因为此时唯一能继续运行的是线程2，但它选择了线程3。

遗憾的是，**Java语言中没有任何特性可以避免或打破死锁。** 必须仔细设计程序，以确保不会出现死锁。

### 12.4.12 为什么弃用stop和suspend方法
Java的最初版本定义了`stop()`方法来停止一个线程，`suspend()`方法阻塞一个线程直到另一个线程调用`resume()`。`stop()`和`suspend()`方法有一个共同点：都试图在没有线程“配合”的情况下控制线程的行为。本节将介绍这些方法为什么有问题，以及怎样避免这些问题。

这三个方法已经被废弃。`stop()`方法天生就不安全，经验表明`suspend()`方法经常导致死锁。

`stop()`方法会终止所有未完成的方法，包括`run()`。当线程被停止时，会立即释放所有已锁定对象的锁。这会导致对象处于不一致的状态。例如，假设转账线程在取款和存款中间被停止，现在银行对象就被破坏了（取出的钱“不翼而飞”了）。因为锁已经释放，其他未停止的线程也可以观察到这种破坏。

当一个线程想要停止另一个线程时，它无法知道什么时候调用`stop()`方法是安全的，什么时候会导致对象被破坏。因此，这个方法被废弃了。希望停止一个线程的时候应该中断该线程，被中断的线程可以在安全的时候停止（见12.3.1节）。

与`stop()`不同，`suspend()`不会破坏对象。如果暂停一个持有锁的线程，那么在这个线程继续运行之前这个锁是不可用的。如果调用`suspend()`方法的线程获得同一个锁，程序就会死锁：被暂停的线程等待继续，将其暂停的线程等待锁。图形用户界面中经常出现这种情况。

如果想安全地暂停线程，可以引入一个变量`suspendRequested`并在`run()`方法的某个安全的地方测试它——安全的地方是指该线程没有锁定其他线程需要的对象。当线程发现`suspendRequested`变量为`true`，就应该等待直到它变为`false`。

### 12.4.13 按需初始化
有时候，你希望第一次需要某个数据结构时才进行初始化，而且希望确保这种初始化只发生一次。虚拟机会在第一次使用类时执行一个静态初始化器，而且只执行一次。虚拟机利用锁来确保这一点，所以不需要自己编程实现。

```java
public class OnDemandData {
    // private constructor to ensure only one object is constructed
    private OnDemandData() {
        ...
    }
    
    public static OnDemandData getInstance() {
        return Holder.INSTANCE;
    }
    
    // only initialized on first use, i.e. in the first call to getInstance
    private static class Holder {
        // VM guarantees that this happens at most once
        static final OnDemandData INSTANCE = new OnDemandData();
    }
}
```

警告：要采用这种方法，必须确保构造器不会抛出任何异常。虚拟机不会做第二次尝试来初始化`Holder`类。

### 12.4.14 线程局部变量
前面讨论了在线程之间共享变量的风险。有时可以使用`ThreadLocal`类为每个线程提供各自的实例来避免共享，即**线程局部变量**(thread-local variable)（注：是“线程局部 变量”，不是“线程 局部变量”）。例如，`SimpleDateFormat`类不是线程安全的，假设有一个静态变量：

```java
public static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
```

如果两个线程同时执行以下操作：

```java
String dateStamp = dateFormat.format(new Date());
```

结果可能是混乱的，因为`dateFormat`使用的内部数据结构可能会被并发访问所破坏。当然可以使用同步，但这样开销很大；或者也可以在需要时构造一个局部`SimpleDateFormat`对象，不过这也很浪费。

要为每个线程构造一个实例，可以使用以下代码：

```java
public static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```

要访问具体对象，调用

```java
String dateStamp = dateFormat.get().format(new Date());
```

在一个线程中首次调用`get()`时，会调用构造器中的lambda表达式。在此之后，`get()`方法会返回属于当前线程的那个实例。

一个类似的问题是在多个线程中生成随机数。`java.util.Random`类是线程安全的，但是多个线程等待一个共享的随机数生成器会很低效。可以使用`ThreadLocal`类为每个线程提供一个单独的生成器。不过Java 7提供了一个便利类`ThreadLocalRandom`，其`current()`方法返回当前线程独有的随机数生成器实例。

线程局部变量有时也用于使线程调用的所有方法都可以直接访问对象，而不必通过参数传递。

注：`ThreadLocal`与普通局部变量的对比如下

| 特性 | `ThreadLocal` | 局部变量 |
| --- | --- | --- |
| 作用域 | 跨方法、跨类，线程内全局可见 | 仅限于方法内部 |
| 生命周期 | 线程生命周期 | 方法调用期间 |
| 线程安全性 | 线程安全 | 线程安全 |
| 存储位置 | 线程的`ThreadLocalMap` | 栈内存 |
| 适用场景 | 线程上下文共享数据，避免参数传递 | 方法内部的临时变量 |

### 其他同步器
`java.util.concurrent`包提供了几个能帮助管理相互协作的线程的类：`CyclicBarrier`、`Phaser`、`CountDownLatch`、`Exchanger`、`Semaphore`和`SynchronousQueue`。详见API文档。

## 12.5 线程安全的集合
如果多个线程并发地修改一个数据结构（如散列表），那么很容易破坏这个数据结构。例如，一个线程在调整散列表桶的链接的过程中被抢占。如果另一个线程开始遍历同一个散列表，可能会使用无效的链接并造成混乱（抛出异常或者陷入无限循环）。

可以通过锁来保护共享的数据结构，但是选择线程安全的实现通常更容易。下面几节将讨论Java库提供的线程安全的集合。

### 12.5.1 阻塞队列
许多线程问题可以通过使用一个或多个队列来优雅且安全地解决。使用队列可以安全地从一个线程向另一个线程传递数据。生产者线程向队列插入元素，消费者线程取出元素（生产者-消费者模式）。例如，考虑银行转账程序，转账线程可以将转账指令插入一个队列（而不是直接访问银行对象），另一个线程从队列中取出指令并完成转账。只有这个线程可以访问银行对象的内部，因此不需要同步（当然，线程安全的队列类的实现者必须考虑同步）。

**阻塞队列**(blocking queue)是一种线程安全的队列，如果试图在队列已满时插入元素或者在队列为空时删除元素，将导致线程阻塞。`BlockingQueue`接口提供了以下方法：

| 操作 | 抛出异常 | 返回特殊值 | 阻塞 | 超时 |
| --- | --- | --- | --- | --- |
| 插入 | `add(e)` | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| 删除 | `remove()` | `poll()` | `take()` | `poll(time, unit)` |
| 查看 | `element()` | `peek()` | | |

例如，以下调用

```java
boolean success = q.offer(x, 100, TimeUnit.MILLISECONDS);
```

尝试在100毫秒内将元素插入队尾。如果成功则返回`true`，如果超时则返回`false`。

`java.util.concurrent`包提供了阻塞队列的几个变体，如下图所示。

![阻塞队列类](/assets/images/java-note-v1ch12-concurrency/阻塞队列类.png)

程序清单12-6中的程序展示了如何使用阻塞队列来控制一组线程。程序搜索一个目录及其子目录中的所有文件，打印出包含指定关键词的行。

[程序清单12-6 blockingQueue/BlockingQueueTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/blockingQueue/BlockingQueueTest.java)

程序启动了一个枚举线程（生产者），负责枚举所有子目录中的所有文件，并将其放到一个阻塞队列中。还启动了大量搜索线程（消费者），每个搜索线程从队列中取出一个文件，打开它，打印所有包含指定关键词的行，然后取出下一个文件。这里使用了一个技巧来终止程序：枚举线程会在队列中放置一个虚拟(dummy)对象作为完成信号，当搜索线程取到这个虚拟对象时将其放回并终止。

注意，这里不需要显式的线程同步。在这个程序中，使用阻塞队列作为一种同步机制。

### 12.5.2 并发映射、集和队列
`java.util.concurrent`包提供了并发映射、有序集和队列的高效实现：`ConcurrentHashMap`、`ConcurrentSkipListMap`、`ConcurrentSkipListSet`和`ConcurrentLinkedQueue`。

这些集合使用复杂的算法，通过允许并发访问数据结构的不同部分来最小化竞争。

与大多数集合不同，这些类的`size()`方法不一定在常量时间内完成。确定这些集合的当前大小通常需要遍历。

注释：并发散列映射的`size()`方法只能返回`int`。如果映射包含超过20亿个条目，就应该使用`mappingCount()`，该方法返回`long`。

并发集合返回**弱一致性**(weakly consistent)的迭代器。这意味着迭代器不一定能反映出它们被构造之后的所有修改，但是不会将同一个值返回两次，也不会抛出`ConcurrentModificationException`（相反，对于`java.util`包中的集合，如果在迭代器构造之后修改集合，迭代器会抛出`ConcurrentModificationException`）。

并发散列映射可以高效地支持大量读线程和有限数量的写线程。

注：`ConcurrentHashMap`的实现原理见[【Java】HashMap源代码解读]({% post_url 2021-03-17-java-hash-map-source-code %})。

### 12.5.3 映射条目的原子更新
假设用多个线程统计单词频率。如果使用`ConcurrentHashMap<String, Long>`，下面的代码显然不是线程安全的：

```java
Long oldValue = map.get(word);
Long newValue = oldValue == null ? 1 : oldValue + 1;
map.put(word, newValue); // ERROR--might not replace oldValue
```

可能会有另一个线程在同时更新同一个计数。

注释：`ConcurrentHashMap`本身是线程安全的，因此`get()`和`put()`永远不会破坏内部结构。但是，**这个操作序列不是原子的**，所以结果不可预知。

在旧版本的Java中，必须使用`replace()`方法（继承自`Map`接口），它会以原子方式将旧值替换为新值，前提是在此之前没有其他线程将旧值替换为其他值。必须一直重试，直到替换成功（这就是CAS的基本思想）：

```java
do {
    oldValue = map.get(word);
    newValue = oldValue == null ? 1 : oldValue + 1;
} while (!map.replace(word, oldValue, newValue));
```

另一种方法是使用`ConcurrentHashMap<String, AtomicLong>`：

```java
map.putIfAbsent(word, new AtomicLong());
map.get(word).incrementAndGet();
```

然而，这会为每次自增都构造一个新的`AtomicLong`，不管是否需要。（注：`computeIfAbsent()`方法可以解决这个问题，如下）

如今，Java API提供了一些可以更方便地完成原子更新的方法。`compute()`方法接受一个键和一个函数，这个函数根据键和旧值来计算新值。例如，可以如下更新计数器映射：

```java
map.compute(word, (k, v) -> v == null ? 1 : v + 1);
```

注释：`ConcurrentHashMap`中不允许有`null`值，因为很多方法都使用`null`值来指示给定的键不存在。

另外还有`computeIfPresent()`和`computeIfAbsent()`方法，分别只在键存在和不存在时计算新值。例如，可以如下更新一个`LongAdder`计数器映射：

```java
map.computeIfAbsent(word, k -> new LongAdder()).increment();
```

这与之前看到的`putIfAbsent()`调用几乎一样，不过只在确实需要一个新的计数器时才会调用`LongAdder`构造器。

首次添加一个键时通常需要做些特殊处理。利用`merge()`方法可以非常方便地做到。如果键不存在则使用给定的初始值，否则调用提供的函数来组合旧值与初始值。

```java
map.merge(word, 1L, Long::sum);
```

再不能比这更简洁了。

注释：如果传入`compute()`或`merge()`的函数返回`null`，将从映射中删除现有条目。

警告：使用`compute()`或`merge()`时，要记住提供的函数不能做太多工作，否则可能会阻塞对映射的其他更新。当然，这个函数也不应该更新映射的其他部分。

注：本节介绍的更新方法都继承自`Map`接口（见9.4.2节），但`ConcurrentHashMap`提供了原子实现。

程序清单12-7中的程序使用并发散列映射来统计一个目录中所有Java文件的单词计数。

[程序清单12-7 concurrentHashMap/ConcurrentHashMapDemo.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/concurrentHashMap/ConcurrentHashMapDemo.java)

### 12.5.4 并发散列映射的批操作
`ConcurrentHashMap`提供了批操作(bulk operation)：遍历映射，并对过程中找到的元素进行操作。批操作不会冻结映射的当前快照。除非你知道执行批操作时映射不会被修改，否则应该把结果看作映射状态的近似。

有三种批操作：
* `search`对每个键和/或值应用一个函数，直到函数生成非`null`的结果。然后搜索终止，返回这个函数的结果。
* `reduce`组合所有键和/或值，使用提供的累加函数。
* `forEach`对所有键和/或值应用一个函数。

每种操作都有四个版本：
* `opKeys`：操作键
* `opValues`：操作值
* `op`：操作键和值
* `opEntries`：操作`Map.Entry`对象

对于每个操作，需要指定一个**并行度阈值**(parallelism threshold)（批大小，即每个线程处理大致多少个元素）。如果映射包含的元素多于这个阈值，批操作就会并行完成。如果希望批操作在单个线程中运行，就使用阈值`Long.MAX_VALUE`；如果希望用尽可能多的线程运行批操作，就使用阈值1。

注：这些批操作都是`ConcurrentHashMap`类提供的，而不是接口。每种操作都有一个对应的内部类，例如`searchKeys()`对应`ConcurrentHashMap.SearchKeysTask`。底层是用fork-join框架实现的（见12.6.4节），其`compute()`方法实现具体操作。

先来看`search`方法，有以下版本：

```java
U searchKeys(long threshold, Function<? super K, ? extends U> f)
U searchValues(long threshold, Function<? super V, ? extends U> f)
U search(long threshold, BiFunction<? super K, ? super V, ? extends U> f)
U searchEntries(long threshold, Function<Map.Entry<K, V>, ? extends U> f)
```

例如，假设希望找出第一个出现次数超过1000的单词，需要搜索键和值：

```java
String result = map.search(threshold, (k, v) -> v > 1000 ? k : null);
```

`result`会被设置为第一个匹配的单词，如果未找到则为`null`。

`forEach()`方法有两种形式。第一种形式对每个键值对应用一个**消费者**(consumer)函数。例如：

```java
map.forEach(threshold, (k, v) -> System.out.println(k + " -> " + v));
```

第二种形式还接受一个**转换器**(transformer)参数，先应用这个函数，再将其结果传递给消费者：

```java
map.forEach(threshold,
    (k, v) -> k + " -> " + v, // transformer
    System.out::println); // consumer
```

转换器可以用作过滤器：如果转换器返回`null`，这个值就会被跳过。例如，只打印值大于1000的条目：

```java
map.forEach(threshold,
    (k, v) -> v > 1000 ? k + " -> " + v : null, // filter and transformer
    System.out::println); // the nulls are not passed to the consumer
```

`reduce`操作用一个累加函数组合其输入。例如，可以如下计算所有值的总和：

```java
Long sum = map.reduceValues(threshold, Long::sum);
```

与`forEach`一样，也可以提供一个转换器函数。可以如下计算最长的键的长度：

```java
Integer maxlength = map.reduceKeys(threshold,
    String::length, // transformer
    Integer::max); // accumulator
```

转换器也可以作为过滤器，通过返回`null`来排除不想要的输入。下面统计值大于1000的条目数量：

```java
Long count = map.reduceValues(threshold,
    v -> v > 1000 ? 1L : null,
    Long::sum);
```

注释：如果映射为空，或者所有条目都被过滤掉，则`reduce`操作返回`null`。如果只有一个元素，则返回其转换结果，不会应用累加器。

对于`int`、`long`和`double`输出还有相应的特殊化操作，分别有后缀`ToInt`、`ToLong`和`ToDouble`。需要将输入转换为基本类型值，并指定默认值和累加器函数。映射为空时返回默认值。

```java
long sum = map.reduceValuesToLong(threshold,
    Long::longValue, // transformer to primitive type
    0, // default value for empty map
    Long::sum); // primitive type accumulator
```

警告：当只有一个元素时，这些特殊化版本与对象版本的行为有所不同。不是直接返回（唯一元素的）转换结果，而是将其与默认值累加。因此，默认值必须是累加器的零元素。

### 12.5.5 并发集视图
如果你想要线程安全的集而不是映射，并没有`ConcurrentHashSet`类，而且你肯定不想自己创建这样一个类。

`ConcurrentHashMap`类的静态方法`newKeySet()`会生成一个`Set<K>`，这实际上是`ConcurrentHashMap<K, Boolean>`的一个包装器（所有值都是`Boolean.TRUE`，但无需关心）。

```java
Set<String> words = ConcurrentHashMap.<String>newKeySet();
```

如果已有一个映射，`keySet()`方法可以生成键集。键集是可变的，可以删除元素，但不能插入元素（就像普通映射一样，见9.4.3节）。

还有一个带默认值参数的`keySet()`方法，可用于向键集中添加元素：

```java
Set<String> words = map.keySet(1L);
words.add("Java");
```

如果`"Java"`在`words`中不存在，则现在的值为1。

注：`ConcurrentHashMap.<K>newKeySet()`大致等价于`new ConcurrentHashMap<K, Boolean>().keySet(Boolean.TRUE)`。

### 12.5.6 写时拷贝数组
`CopyOnWriteArrayList`和`CopyOnWriteArraySet`是线程安全的集合，其所有修改器方法（如`add()`）会创建底层数组的一个副本。如果读取线程远多于写入线程，这会很有用。迭代器包含当前数组的引用。如果数组后来被修改了，集合的数组会被替换，但迭代器仍然引用原来的数组。其结果是，迭代器可以访问一致的（但可能过时的）视图，而没有任何同步开销。

### 12.5.7 并行数组算法
`Arrays`类提供了一些并行化操作，基本类型数组和对象数组都有相应的版本。

`parallelSort()`方法对一个数组排序。例如：

```java
var contents = new String(Files.readAllBytes(
    Path.of("alice.txt")), StandardCharsets.UTF_8); // read file into string
String[] words = contents.split("[\\P{L}]+"); // split along nonletters
Arrays.parallelSort(words);
```

对对象排序时，可以提供一个`Comparator`：

```java
Arrays.parallelSort(words, Comparator.comparing(String::length));
```

也可以指定范围边界，例如：

```java
Arrays.parallelSort(words, words.length / 2, words.length); // sort the upper half
```

`parallelSetAll()`方法会用由一个函数计算得到的值填充数组。这个函数接受元素索引，并计算相应位置上的值。

```java
Arrays.parallelSetAll(values, i -> i % 10);
    // fills values with 0 1 2 3 4 5 6 7 8 9 0 1 2 ...
```

最后，`parallelPrefix()`方法使用给定结合操作的前缀累加结果替换各个数组元素（注：类似于Python的`itertools.accumulate()`）。例如，考虑数组`[1, 2, 3, 4, 5, 6, 7, 8]`和乘法操作。执行`Arrays.parallelPrefix(values, (x, y) -> x * y)`之后，数组将包含阶乘值

```
[1, 1*2, 1*2*3, 1*2*3*4, 1*2*3*4*5, 1*2*3*4*5*6, 1*2*3*4*5*6*7, 1*2*3*4*5*6*7*8]
```

令人惊讶的时，这个计算确实可以并行化。首先，结合相邻元素，如下所示：

```
[1, 1*2, 3, 3*4, 5, 5*6, 7, 7*8]
    ~~~     ~~~     ~~~     ~~~
```

显然，可以在数组的不同区域中并行完成这个计算。下一步，更新波浪线指示的元素，将其与前面一个或两个位置上的元素相乘：

```
[1, 1*2, 1*2*3, 1*2*3*4, 5, 5*6, 5*6*7, 5*6*7*8]
         ~~~~~  ~~~~~~~          ~~~~~  ~~~~~~~
```

这同样可以并行完成。经过log(n)步之后，这个过程完成。如果有足够多的处理器，这会远胜过简单的线性计算。

### 12.5.8 早期的线程安全集合
从Java的最初版本开始，`Vector`和`Hashtable`类就提供了动态数组和散列表的线程安全的实现。现在这些类被认为已经过时，被`ArrayList`和`HashMap`类所取代。这些类不是线程安全的，而集合库提供了一种不同的机制。任何集合类都可以通过**同步包装器**(synchronization wrapper)变成线程安全的（见9.5.5节）：

```java
List<E> synchArrayList = Collections.synchronizedList(new ArrayList<E>());
Map<K, V> synchHashMap = Collections.synchronizedMap(new HashMap<K, V>());
```

所得到的集合的方法用一个锁保护，可以提供线程安全的访问。应该确保没有线程通过原始的非同步方法访问数据结构。最简单的方法是不要保存原始对象的引用（正如示例所做的那样）。

如果希望对一个集合进行迭代，同时另一个线程有机会修改这个集合，那么仍然需要使用同步：

```java
synchronized (synchHashMap) {
    for (K key : synchHashMap.keySet())
        // ...
}
```

通常最好使用`java.util.concurrent`包中的集合，而不是同步包装器。特别是，`ConcurrentHashMap`经过了精心实现，使得多个线程访问不同的桶不会相互阻塞。经常更改的数组列表是一个例外，在这种情况下，同步的`ArrayList`要胜过`CopyOnWriteArrayList`。

## 12.6 任务和线程池
后面几节将介绍Java并发框架为协调并发任务提供的一些工具。

### 12.6.1 Callable和Future
`Runnable`封装了一个异步运行的任务，可以把它想象成一个没有参数和返回值的异步方法。`Callable`与`Runnable`类似，但是有返回值。

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

类型参数是返回值的类型。例如，`Callable<Integer>`表示一个最终返回`Integer`对象的异步计算。

`Future`保存异步计算的结果（注：这个名字的含义为“可以在**未来**某个时间点获取的结果”）。可以启动一个计算，并得到`Future`对象。当计算完成时，就可以通过`Future`对象获得结果。

`Future<V>`接口有以下方法：

```java
V get()
V get(long timeout, TimeUnit unit)
boolean isDone()
void cancel(boolean mayInterrupt)
boolean isCancelled()
```

调用第一个`get()`方法会阻塞直到计算完成。第二个`get()`方法也会阻塞，但如果在计算完成之前超时，就会抛出`TimeoutException`。如果运行该计算的线程被中断，这两个方法都会抛出`InterruptedException`。如果计算已经完成，`get()`会立即返回。

如果计算还在进行，则`isDone()`方法返回`false`；如果已经完成则返回`true`。

可以用`cancel()`方法取消计算。如果计算还没有开始，就会被取消且不再开始。如果计算正在进行，当`mayInterrupt`参数为`true`时计算会被中断。

警告：取消任务包括两个步骤：找到并中断底层线程。任务实现（`call()`方法）必须感知中断并放弃工作（见12.3.1节）。如果`Future`对象不知道任务在哪个线程中执行，或者任务没有监控执行线程的中断状态，那么取消任务没有任何效果。

执行`Callable`的一种方法是使用`FutureTask`，它实现了`Future`和`Runnable`接口，所以可以构造一个线程来运行：

```java
Callable<Integer> task = ...;
var futureTask = new FutureTask<Integer>(task);
var t = new Thread(futureTask); // it's a Runnable
t.start();
...
Integer result = futureTask.get(); // it's a Future
```

注：`Thread`只接受`Runnable`，不接受`Callable`。

[future/FutureTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/future/FutureTest.java)

更常见的情况是将`Callable`传递给执行器，这将在下一节介绍。

### 12.6.2 执行器
构造一个新的线程开销有点大，因为这涉及与操作系统的交互。如果程序创建了大量的生存期很短的线程，就不应该把每个任务映射到一个单独的线程，而应该使用**线程池**(thread pool)。线程池包含许多准备运行的线程。向线程池提交一个`Runnable`，就会有一个线程调用其`run()`方法。当`run()`方法退出时，这个线程不会死亡，而是等待处理下一个请求。

`Executors`类有一些用来构造线程池的静态工厂方法，如下表所示。

| 方法 | 描述 |
| --- | --- |
| `newCachedThreadPool()` | 必要时创建新线程、优先复用已有线程的线程池，空闲线程会保留60秒 |
| `newFixedThreadPool()` | 包含固定数量线程的线程池，空闲线程会一直保留 |
| `newWorkStealingPool()` | 适合fork-join任务的线程池，复杂任务被分解为简单任务，空闲线程“窃取”简单任务 |
| `newSingleThreadExecutor()` | 只有一个线程的线程池，顺序地执行提交的任务 |
| `newScheduledThreadPool()` | 用于调度执行的固定线程池 |
| `newSingleThreadScheduledExecutor()` | 用于调度执行的单线程池 |

`newCachedThreadPool()`、`newFixedThreadPool()`和`newSingleThreadExecutor()`返回一个实现了`ExecutorService`接口的`ThreadPoolExecutor`类的对象。

如果线程生存期很短或者大量时间都在阻塞，就可以使用缓存线程池。

如果线程在努力工作而并不阻塞，你不希望同时运行大量线程。为了得到最优的速度，并发线程数应该等于处理器核数。在这种情况下就应该使用固定线程池。

单线程池对于性能分析很有用。如果临时用单线程池替换缓存或固定线程池，就可以测量在没有并发的帮助下应用的运行速度会慢多少。

注：线程池的实现原理见[【Java】线程池]({% post_url 2021-03-15-java-thread-pool %})。

可以用下面的方法之一向`ExecutorService`提交一个`Runnable`或`Callable`：

```java
Future<?> submit(Runnable task)
Future<T> submit(Runnable task, T result)
Future<T> submit(Callable<T> task)
```

线程池会尽早运行提交的任务。调用`submit()`时，会得到一个`Future`对象，可用来得到结果或取消任务。

使用完线程池后，调用`shutdown()`。被关闭的线程池不再接受新的任务。当所有任务都完成时，线程池中的线程死亡。另一种方法是调用`shutdownNow()`，线程池会取消所有尚未开始的任务。

下面总结了线程池的用法：
1. 调用`Executors`类的静态方法`newCachedThreadPool()`或`newFixedThreadPool()`创建线程池。
2. 调用`submit()`提交`Runnable`或`Callable`对象。
3. 保留返回的`Future`对象，以便得到结果或取消任务。
4. 不再提交任务时，调用`shutdown()`。

上一节的示例程序创建了大量生存期很短的线程（每个目录一个线程）。下面的程序使用线程池来实现相同的功能。

[threadPool/ThreadPoolTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/threadPool/ThreadPoolTest.java)

`ScheduledExecutorService`接口用于调度执行或重复执行的任务。这是对`java.util.Timer`的泛化，可以支持线程池。`Executors`类的`newScheduledThreadPool()`和`newSingleThreadScheduledExecutor()`方法返回实现了该接口的对象。可以调度任务在初始延迟后运行一次，也可以调度定期运行，详见API文档。

### 12.6.3 控制任务组
`ExecutorService`也可以用于控制一组相关的任务。

`invokeAny()`方法提交一个`Callable`对象集合，并返回**某个**已完成任务的结果（往往是最快完成的那个任务）。对于搜索问题，如果愿意接受任何一种解，就可以使用这个方法。例如，假设需要对一个大整数进行因数分解。可以提交很多任务，每个任务尝试使用不同范围内的数进行分解。只要其中一个任务得到了答案，计算就可以停止了。

`invokeAll()`方法也提交一个`Callable`对象集合，会阻塞直到所有任务都完成，并返回一个表示所有任务结果的`Future`对象列表（其中每个对象的`isDone()`都为`true`）。例如：

```java
List<Callable<T>> tasks = ...;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result : results)
    processFurther(result.get());
```

如果要按完成顺序获得结果，可以使用`ExecutorCompletionService`。这个类管理一个`Future`对象的阻塞队列，包含已完成任务的结果。因此，对于前面的计算，更高效的组织方式如下：

```java
var service = new ExecutorCompletionService<T>(executor);
for (Callable<T> task : tasks)
    service.submit(task);
for (int i = 0; i < tasks.size(); i++)
    processFurther(service.take().get());
```

程序清单12-8中的程序展示了如何使用`Callable`和执行器。程序的第一部分使用`invokeAll()`统计一个目录中包含给定单词的文件数。程序还会显示搜索过程花费的时间。尝试将缓存线程池替换为单线程池并再次运行，看看并发计算是否更快。

第二部分使用`invokeAny()`搜索包含给定单词的第一个文件。注意，一旦任何任务返回，`invokeAny()`方法就会终止。所以不能让搜索任务返回一个`boolean`来指示成功或失败，而是让失败的任务抛出`NoSuchElementException`。另外，当一个任务成功时，其他任务就要取消，因此要监控中断状态。

[程序清单12-8 executors/ExecutorDemo.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/executors/ExecutorDemo.java)

提示：在自己的程序中，应当使用`ExecutorService`来管理线程，而不是单独启动线程。

### 12.6.4 Fork-Join框架
Java 7引入了fork-join框架，用来支持计算密集型任务。假设有一个任务，它可以很自然地分解为子任务，如下所示：

```java
if (problemSize < threshold){
    // solve problem directly
}
else {
    // break problem into subproblems
    // recursively solve each subproblem
    // combine the results
}
```

图像处理就是这样一个例子。这里讨论一个更简单的例子：假设想统计一个数组中有多少个元素满足特定的条件。可以将这个数组一分为二，分别统计这两部分，再将结果相加。

要使递归计算具有框架可用的形式，需要提供一个扩展`RecursiveTask<T>`（如果计算生成`T`类型的结果）或`RecursiveAction`（如果不生成结果）的类。覆盖`compute()`方法来生成并调用子任务，然后合并其结果。

```java
class Counter extends RecursiveTask<Integer> {
    ...
    protected Integer compute() {
        if (to - from < THRESHOLD) {
            // solve problem directly
        }
        else {
            int mid = from + (to - from) / 2;
            var first = new Counter(values, from, mid, filter);
            var second = new Counter(values, mid, to, filter);
            invokeAll(first, second);
            return first.join() + second.join();
        }
    }
}
```

最后，使用`ForkJoinPool`来执行整个任务。程序清单12-9给出了完整的示例代码。

[程序清单12-9 forkJoin/ForkJoinTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/forkJoin/ForkJoinTest.java)

在幕后，fork-join框架使用了一种称为**工作窃取**(work stealing)的有效启发式方法来平衡可用线程之间的工作负载。每个工作线程都有一个任务的双端队列。工作线程将子任务压入自己的双端队列的队头。当工作线程空闲时，它会从另一个双端队列的队尾“窃取”一个任务。由于大的子任务都在队尾，这种窃取很少见。

警告：`ForkJoinPool`是针对非阻塞工作负载优化的。如果向其中添加很多阻塞任务，会让它无法有效工作。

## 12.7 异步计算框架
到目前为止，并发计算的方法都是先分解一个任务，然后等待所有部分完成。下面几节将介绍如何实现无等待的**异步**(asynchronous)计算。

### 12.7.1 可完成Future
`CompletableFuture`类实现了`Future`接口，它提供了另一种获得结果的机制。需要注册一个**回调**(callback)，一旦结果可用，就会（在某个线程中）使用结果调用它。

```java
CompletableFuture<String> f = ...;
f.thenAccept(s -> /* Process the result string s */);
```

有一些API方法会返回`CompletableFuture`对象。例如，可以用`HttpClient`类（将在卷II第4章介绍）异步地获取一个网页：

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder(URI.create(urlString)).GET().build();
CompletableFuture<HttpResponse<String>> f = client.sendAsync(request, BodyHandlers.ofString());
```

但是大多数情况下，你需要自己创建`CompletableFuture`。要想异步运行任务并得到`CompletableFuture`，不能直接把它提交给`ExecutorService`，而应当调用静态方法`CompletableFuture.supplyAsync()`。在不使用`HttpClient`类的情况下读取网页的方式如下：

```java
public CompletableFuture<String> readPage(URL url) {
    return CompletableFuture.supplyAsync(() -> {
        try {
            return new String(url.openStream().readAllBytes(), "UTF-8");
        }
        catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }, executor);
}
```

如果省略执行器参数，任务会在一个默认执行器(`ForkJoinPool.commonPool()`)上运行。通常你并不希望这么做。

警告：`supplyAsync()`方法的第一个参数是`Supplier<T>`而不是`Callable<T>`。这两个接口都描述了无参数、返回值类型为`T`的函数，但`Supplier`不能抛出检查型异常。

`CompletableFuture`可能以两种方式完成：得到一个结果，或者一个未捕获的异常。要处理这两种情况，可以使用`whenComplete()`方法：

```java
f.whenComplete((result, throwable) -> {
    if (throwable == null) {
        // Process the result
    }
    else {
        // Process the throwable
    }
});
```

`CompletableFuture`之所以被称为“可完成的”，是因为可以（使用`complete()`）手动设置完成值（在其他并发库中，这种对象称为**承诺**(promise)）。要用一个异常完成future，调用`completeExceptionally()`。

注释：在多个线程中对同一个future调用`complete()`或`completeExceptionally()`是安全的。如果这个future已经完成，这些调用没有任何作用。

警告：与普通的`Future`不同，调用`cancel()`方法时`CompletableFuture`的计算不会被中断，而是将其设置为以异常`CancellationException`完成。

### 12.7.2 组合可完成Future
非阻塞调用通过回调来实现。然而，组合多个步骤的回调会使程序逻辑过于分散（“[回调地狱](http://callbackhell.com/)”）、难以理解。

`CompletableFuture`提供了一种将异步任务**组合**为处理流水线的机制来解决这个问题。

例如，假设希望从一个网页中抽取所有图像，`readPage()`方法异步读取网页，`getImageURLs()`方法生成HTML页面中所有图像的URL：

```java
public CompletableFuture<String> readPage(URL url)
public List<URL> getImageURLs(String page)
```

可以让`getImageURLs()`在页面可用时被调用：

```java
CompletableFuture<String> contents = readPage(url);
CompletableFuture<List<URL>> imageURLs = contents.thenApply(this::getImageURLs);
```

`thenApply()`方法不会阻塞，而是返回另一个future。当第一个future完成时，其结果会提供给`getImageURLs()`方法，这个方法的返回值就是最终结果。

利用可完成future，你只需指定希望完成的操作以及执行顺序。虽然不会立即执行，但重要的是所有代码都放在一处。

有很多不同的方法来组合可完成future。首先来看处理单个future的方法（如下表所示）（这里把`Function<? super T, ? extends U>`简写为`T -> U`）。对于列出的每个方法，还有两个`Async`形式（这里没有列出），其中一种形式使用公共的`ForkJoinPool`，另一种形式有一个`Executor`参数。

| 方法 | 参数 | 描述 |
| --- | --- | --- |
| `thenApply` | `T -> U` | 对结果应用一个函数 |
| `thenAccept` | `T -> void` | 类似于`thenApply`，但没有结果 |
| `thenCompose` | `T -> CompletableFuture<U>` | 对结果调用函数并执行返回的future |
| `thenRun` | `Runnable` | 执行`Runnable` |
| `handle` | `(T, Throwable) -> U` | 处理结果或错误，生成一个新结果 |
| `whenComplete` | `(T, Throwable) -> void` | 类似于`handle`，但没有结果 |
| `exceptionally` | `Throwable -> T` | 从错误计算一个结果 |
| `exceptionallyCompose` | `Throwable -> CompletableFuture<T>` | 对异常调用函数并执行返回的future |
| `completeOnTimeout` | `T, long, TimeUnit` | 如果超时则将给定值作为结果 |
| `orTimeout` | `long, TimeUnit` | 如果超时则抛出`TimeoutException` |

前面已经见过`thenApply()`方法。

`thenCompose()`方法接受一个将`T`映射到`CompletableFuture<U>`（而不是`U`）的函数。假设有两个函数`T -> CompletableFuture<U>`和`U -> CompletableFuture<V>`，如果在第一个函数完成时调用第二个函数，它们就组合为一个函数`T -> CompletableFuture<V>`。这正是`thenCompose()`所做的。

假设除了前面的`readPage()`方法，还有一个从用户输入获得URL的方法：

```java
public CompletableFuture<URL> getURLInput(String prompt)
```

就可以使用`thenCompose()`将它们组合起来：

```java
CompletableFuture<String> contents = getURLInput(prompt)
    .thenCompose(this::readPage);
```

在上一节中已经看到`whenComplete()`方法用于处理异常。还有一个`handle()`方法会计算一个新结果。在很多情况下，调用`exceptionally()`方法会更简单。这个方法在出现异常时计算一个虚拟值(dummy value)：

```java
CompletableFuture<List<URL>> imageURLs = readPage(url)
    .exceptionally(ex -> "<html></html>")
    .thenApply(this::getImageURLs);
```

可以采用同样的方式处理超时：

```java
CompletableFuture<List<URL>> imageURLs = readPage(url)
    .completeOnTimeout("<html></html>", 30, TimeUnit.SECONDS)
    .thenApply(this::getImageURLs);
```

上表中结果为`void`的方法通常在处理流水线的最后使用。

注：这些方法仅仅指定要执行的操作，只有当调用`get()`时才会实际执行。

下面来看组合多个future的方法（如下表所示）。

| 方法 | 参数 | 描述 |
| --- | --- | --- |
| `thenCombine` | `CompletableFuture<U>, (T, U) -> V` | 执行两个future并用给定的函数组合结果 |
| `thenAcceptBoth` | `CompletableFuture<U>, (T, U) -> void` | 类似于`thenCombine`，但没有结果 |
| `runAfterBoth` | `CompletableFuture<?>, Runnable` | 两个都完成后执行`Runnable` |
| `applyToEither` | `CompletableFuture<T>, T -> U` | 其中一个完成后对结果调用给定的函数 |
| `acceptEither` | `CompletableFuture<T>, T -> void` | 类似于`applyToEither`，但没有结果 |
| `runAfterEither` | `CompletableFuture<?>, Runnable` | 其中一个完成后执行`Runnable` |
| `static allOf` | `CompletableFuture<?>...` | 所有future都完成后则完成，结果为`void` |
| `static anyOf` | `CompletableFuture<?>...` | 任意future完成后则完成，生成其结果 |

前三个方法并发运行一个`CompletableFuture<T>`和`CompletableFuture<U>`动作，并组合结果。

接下来三个方法并发运行两个`CompletableFuture<T>`动作。一旦其中一个完成就传递其结果，并忽略另一个结果。

静态方法`allOf()`和`anyOf()`接受一组`CompletableFuture`并生成一个`CompletableFuture`，在所有/任意future完成时完成。`allOf()`方法不生成任何结果。`anyOf()`方法不会终止剩余任务。

注释：严格来说，本节介绍的方法接受`CompletionStage`类型的参数，而不是`CompletableFuture`。`CompletableFuture`同时实现了`CompletionStage`和`Future`。

程序清单12-10给出了一个完成的程序（网络爬虫），它会读取一个网页，抽取其中的图像，下载图像并保存到本地。

[程序清单12-10 completableFutures/CompletableFutureDemo.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/completableFutures/CompletableFutureDemo.java)

### 12.7.3 用户界面回调中的长时间运行任务
使用线程的原因之一是为了提高程序的响应性。对于有用户界面的应用，这一点尤其重要。当程序需要做某些耗时的工作时，不能在UI线程中做这些工作，否则用户界面会冻结（卡住），而应该启动另一个工作线程。

不过，不要在工作线程中更新用户界面。Swing、JavaFX或Android等用户界面都不是线程安全的，不能从多个线程操作用户界面元素。因此，需要让所有UI更新都在UI线程中执行。每个用户界面库都提供了调度一个`Runnable`在UI线程中执行的机制。例如，在Swing中调用`EventQueue.invokeLater()`。

在工作线程中实现用户反馈很烦琐，所以每个用户界面库都提供了某种辅助类来管理有关细节，如Swing中的`SwingWorker`、JavaFX中的`Task`以及Android中的`AsyncTask`。

程序清单12-11中的程序提供了加载文本文件和取消加载过程的命令。应该用一个长文件来测试这个程序，例如《基督山伯爵》(The Count of Monte Cristo)的全文([crsto10.txt](https://github.com/ZZy979/Core-Java-code/blob/main/gutenberg/crsto10.txt))。文件在一个单独的线程中加载。在读取文件的过程中，Open菜单项被禁用，Cancel项启用（如下图所示）。每读取一行后，状态栏中的行计数器会更新。读取完成后，Open菜单项会重新启用，Cancel项被禁用，状态栏文本设置为Done。

![在单独的线程中加载文件](/assets/images/java-note-v1ch12-concurrency/在单独的线程中加载文件.png)

这个例子展示了后台任务的典型UI活动：
* 每个工作单元完成后，更新UI来显示进度。
* 整个工作完成后，对UI做最后的修改。

利用这个简单的技术，就能在执行耗时任务的同时保证用户界面仍能响应。

[程序清单12-11 swingWorker/SwingWorkerTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/swingWorker/SwingWorkerTest.java)

## 12.8 进程
到目前为止，已经了解了如何在同一个程序的不同线程中执行Java代码。有时你还需要执行另一个程序。为此，可以使用`ProcessBuilder`和`Process`类。`Process`类在一个单独的操作系统**进程**(process)中执行一个命令，并允许与其标准输入、输出和错误流交互。`ProcessBuilder`类用于配置`Process`对象。

注释：`ProcessBuilder`类可以替代`Runtime.exec()`调用，而且更灵活。

注：Python标准库模块`subprocess`实现了类似的功能。

### 12.8.1 创建进程
首先指定想要执行的命令。可以提供一个`List<String>`（命令和每个参数分开），或者直接提供命令字符串（命令和参数用空格分隔）。

```java
var builder = new ProcessBuilder("gcc", "myapp.c");
```

警告：第一个字符串必须是可执行命令，而不是shell内置命令。例如，要在Windows上运行`dir`命令（相当于Linux的`ls`），需要用字符串`"cmd.exe", "/C", "dir"`来创建进程。

每个进程都有一个**工作目录**(working directory)，用来解析相对路径。默认情况下，进程的工作目录与虚拟机相同，通常是启动`java`程序的目录。可以用`directory()`方法改变工作目录：

```java
builder = builder.directory(path.toFile());
```

注释：`ProcessBuilder`的各个方法都返回其自身，所以可以链式调用：

```java
Process p = new ProcessBuilder(command)
    .directory(file)
    .start();
```

可以用以下方法访问进程的标准输入、输出和错误流：

```java
OutputStream processIn = p.getOutputStream(); // stdin
InputStream processOut = p.getInputStream();  // stdout
InputStream processErr = p.getErrorStream();  // stderr
```

注意，进程的输入流在当前程序中是一个输出流，写入的内容会成为进程的输入。反过来，程序会读取进程的输出和错误流，对于当前程序来说它们都是输入流。（但是把方法名也反过来太容易误导了吧……）

要指定进程的三个流与JVM相同（即控制台），调用`builder.inheritIO()`。如果只想继承某个流，可以把`ProcessBuilder.Redirect.INHERIT`传递给`redirectInput()`、`redirectOutput()`或`redirectError()`方法，例如：

```java
builder.redirectOutput(ProcessBuilder.Redirect.INHERIT);
```

通过提供`File`对象，可以将进程的流重定向到文件：

```java
builder.redirectInput(inputFile)
    .redirectOutput(outputFile)
    .redirectError(errorFile);
```

进程启动时，会创建或清空输出和错误文件。要追加到现有的文件，可以使用

```java
builder.redirectOutput(ProcessBuilder.Redirect.appendTo(outputFile));
```

要合并输出和错误流，调用`builder.redirectErrorStream(true)`。如果这样做，就不能再调用`builder.redirectError()`或`p.getErrorStream()`。

还可以修改进程的环境变量：

```java
Map<String, String> env = builder.environment();
env.put("LANG", "fr_FR");
env.remove("JAVA_HOME");
Process p = builder.start();
```

如果要将一个进程的输出通过管道传递到另一个进程的输入（类似于shell中的`|`运算符），Java 9提供了`startPipeline()`方法。传入一个`ProcessBuilder`列表，并从最后一个进程读取结果。下面的例子枚举目录中所有文件不同的扩展名：

```java
List<Process> processes = ProcessBuilder.startPipeline(List.of(
    new ProcessBuilder("find", "/opt/jdk-17"),
    new ProcessBuilder("grep", "-o", "\\.[^./]*$"),
    new ProcessBuilder("sort"),
    new ProcessBuilder("uniq")
));
Process last = processes.get(processes.size() - 1);
var result = new String(last.getInputStream().readAllBytes());
```

注：这等价于shell命令

```shell
find /opt/jdk-17 | grep -o "\.[^./]*$" | sort | uniq
```

### 12.8.2 运行进程
配置完成后，调用builder的`start()`方法启动进程。例如：

```java
Process process = new ProcessBuilder("/bin/ls", "-l")
    .directory(Path.of("/tmp").toFile())
    .start();
try (var in = new Scanner(process.getInputStream())) {
    while (in.hasNextLine())
        System.out.println(in.nextLine());
}
```

[process/ReadDir.java](https://github.com/ZZy979/Core-Java-code/blob/main/v1ch12/process/ReadDir.java)

警告：进程流的缓冲空间有限。不要写入太多输入，而且要及时读取输出。如果有大量输入和输出，可能需要在单独的线程中生产和消费。

要等待进程完成，调用

```java
int result = process.waitFor();
```

也可以指定超时时间：

```java
long delay = ...;
if (process.waitFor(delay, TimeUnit.SECONDS)) {
    int result = process.exitValue();
    ...
}
else {
    process.destroyForcibly();
}
```

第一个`waitFor()`返回进程的退出值（按照惯例，0表示成功，非0表示错误）。如果进程没有超时，第二个调用返回`true`，然后调用`exitValue()`方法获取退出值。

可以调用`isAlive()`来查看进程是否仍存活。要杀死进程，调用`destroy()`或`destroyForcibly()`，这二者的区别取决于平台。在UNIX上，前者会以`SIGTERM`终止进程（相当于`kill`），后者会以`SIGKILL`终止进程（相当于`kill -9`）。

最后，可以在进程完成时接收到一个异步通知。调用`process.onExit()`会生成一个`CompletableFuture<Process>`，可以用来调度任何动作（注册回调，见12.7.2节）。

```java
process.onExit()
    .thenAccept(p -> System.out.println("Exit value: " + p.exitValue()));
```

### 12.8.3 进程句柄
要获得一个进程的更多信息，可以使用`ProcessHandle`接口。有四种方式获得进程句柄：
1. 给定`Process`对象`p`，调用`p.toHandle()`。
2. 给定操作系统进程ID，调用`ProcessHandle.of(id)`。
3. `ProcessHandle.current()`是当前JVM进程的句柄。
4. `ProcessHandle.allProcesses()`生成一个`Stream<ProcessHandle>`，包含对当前进程可见的所有操作系统进程。

给定一个进程句柄，可以得到其进程ID、父进程、子进程和后代：

```java
long pid = handle.pid();
Optional<ProcessHandle> parent = handle.parent();
Stream<ProcessHandle> children = handle.children();
Stream<ProcessHandle> descendants = handle.descendants();
```

注释：`allProcesses()`、`children()`和`descendants()`方法返回的`Stream<ProcessHandle>`只是当时的快照。其中一些进程可能已经终止，另外又启动了其他进程。

`info()`方法生成一个`ProcessHandle.Info`对象，可用来获得进程的有关信息，详见API文档。

与`Process`类一样，`ProcessHandle`接口也有`isAlive()`、`destroy()`、`destroyForcibly()`和`onExit()`方法。不过，没有`waitFor()`方法。
