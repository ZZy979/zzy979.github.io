---
title: 【Python】基于协程和asyncio的并发编程
date: 2025-11-30 11:14:50 +0800
categories: [Python]
tags: [python, coroutine, concurrency, asynchronous, event loop, await expression, iterator, for statement, context manager, with statement, future, crawler]
---
**协程**(coroutine)是Python 3.5中引入的一种实现并发编程的方式。它比线程更轻量级，可以在单个线程内实现多个任务的切换执行。协程可以在执行过程中暂停，并在需要时恢复执行。

本文将介绍Python协程的基本概念和实现原理，以及如何使用`async/await`语法和标准库`asyncio`模块编写并发代码。最后使用协程实现一个完整的网络爬虫应用。

## 1.基本概念
### 1.1 协程
**协程函数**(coroutine function)（也称为**异步函数**(asynchronous function)）是一种特殊的函数，可以在执行过程中暂停，并在稍后恢复执行。协程函数通过`async def`关键字定义，例如：

```python
async def hello():
    print('Hello ...')
    await asyncio.sleep(1)
    print('... World!')
```

调用协程函数不会立即执行函数体，而是创建并返回一个**协程对象**(coroutine object)：

```python
>>> hello()
<coroutine object hello at 0x000002360C51A980>
```

要运行协程对象，可以使用`asyncio.run()`函数。该函数会创建一个事件循环，执行给定的协程对象并返回结果。

```python
asyncio.run(hello())
```

运行该程序，会在控制台打印 "Hello ..." ，一秒后再打印 "... World!" 。当然，对于这个Hello World程序来说，使用`time.sleep()`也可以实现同样的效果。协程真正的强大之处在于：等待一个作业完成的同时可以执行其他作业，并且避免了线程创建和切换的开销。

基于协程的异步程序代码框架如下：

```python
import asyncio

async def main():
    # Perform asynchronous instructions
    ...

if __name__ == '__main__':
    asyncio.run(main())  # Block until the coroutine main() finishes
```

注意：术语“协程函数”和“协程对象”经常被统称为“协程”，这可能会引起混淆。在本文中，“协程”特指协程对象。

协程可以在函数体的不同位置暂停和恢复，这使得异步行为成为可能。稍后将介绍如何实现这种异步行为。

### 1.2 事件循环
**事件循环**(event loop)是`asyncio`的核心组件，负责调度和执行协程。事件循环包含一个待运行作业的“队列”，每次从队列中取出一个作业并给予它控制权，然后该作业就会运行。一旦该作业暂停或完成，就将控制权返回给事件循环，然后事件循环将选择另一个作业。这个过程将无限重复。（可以将事件循环大致类比为线程池的调度线程。）

`asyncio.run()`函数会自动创建一个新的事件循环来运行给定的协程。也可以使用`asyncio.new_event_loop()`函数手动创建事件循环。同一个线程中不能同时运行两个事件循环。

异步程序的高效执行依赖于作业之间的共享和合作。如果一个作业长时间独占控制权，会让其他作业陷入饥饿，从而使整个事件循环机制毫无用处。

### 1.3 任务
**任务**(task)是对协程的包装，用于在事件循环中运行协程。可以通过`asyncio.create_task()`函数创建任务，该函数会自动调度给定协程的执行，返回一个`asyncio.Task`对象。不应该手动创建任务对象。

```python
hello_task = asyncio.create_task(hello())
```

可以在其他协程中使用`await`表达式等待任务执行完成（将在下一节介绍）。如果协程等待一个任务则会暂停执行（此时事件循环可以执行其他协程）；当任务完成时，该协程将恢复执行。例如：

```python
async def main():
    hello_task = asyncio.create_task(hello())
    await hello_task
```

可以将这个过程类比为向线程池中提交一个作业并返回一个future对象，通过该对象等待作业完成并返回结果：

```python
with ThreadPoolExecutor() as executor:
    future = executor.submit(my_job)
    res = future.result()
```

事件循环相当于线程池，协程相当于提交的作业，任务相当于future。实际上，任务就是一种future对象，`asyncio.Task`继承了`asyncio.Future`类。

任务还维护了一个回调函数列表，其重要性将在下一节中讨论。需要注意的是，任务本身不会被添加到事件循环中，只有任务的回调函数才会被添加到事件循环中。

如果创建的任务对象在被事件循环调用之前就被垃圾回收了，这可能会产生问题。例如：

```python
async def main():
    asyncio.create_task(hello())
    # Other asynchronous instructions which run for a while and cede control to the event loop...
    ...
```

当事件循环最终尝试运行该任务时，可能会失败并发现该任务对象不存在。为了避免这一问题，应该使用变量引用`asyncio.create_task()`返回的对象。

要取消任务，可以调用`cancel()`方法。当任务被取消时，会引发`asyncio.CancelledError`异常。协程函数应该使用`try`块执行清理逻辑并**重新引发该异常**。例如：

```python
import asyncio

async def hello():
    try:
        print('Hello ...')
        await asyncio.sleep(100)
        print('... World!')
    except asyncio.CancelledError:
        print('Task cancelled')
        # perform clean-up logic
        raise  # propagate CancelledError

async def main():
    hello_task = asyncio.create_task(hello())
    await asyncio.sleep(1)
    hello_task.cancel()

asyncio.run(main())
```

### 1.4 可等待对象和await表达式
[可等待](https://docs.python.org/3/reference/datamodel.html#awaitable-objects)(awaitable)对象是提供了`__await__()`方法的对象，该方法必须返回一个迭代器，用于实现控制权转移（将在2.1节中介绍）。主要有三类可等待对象：协程、任务和future。

关键字`await`用于暂停当前协程的执行，以等待一个可等待对象执行完成并返回结果。表达式`await obj`会调用`obj.__await__()`方法，只能在协程函数内部使用。

`await`的行为取决于被等待的对象类型：
* 等待任务（或future）会将控制权从当前协程交还给事件循环。
* 等待协程不会将控制权交还给事件循环。

再考虑上一节中的示例：

```python
async def main():
    hello_task = asyncio.create_task(hello())
    await hello_task
```

假设事件循环已经将控制权交给了协程`main()`。`await hello_task`这条语句会向`hello_task`对象的回调函数列表中添加一个回调函数（用于恢复`main()`的执行），随后将控制权交还给事件循环。一段时间后，事件循环会将控制权交给`hello_task`，该任务会完成其工作（执行协程`hello()`）。当该任务结束时，会将其回调函数添加到事件循环中（在这里是恢复`main()`的执行）。

一般来说，**当被等待的任务完成时，原先的协程将被添加回事件循环以便恢复运行**。这是一个基本但可靠的思维模型。

与任务不同，**等待协程并不会将控制权交还给事件循环**。`await coroutine`的行为实际上与调用普通函数相同。考虑下面的程序：

```python
import asyncio

async def coro_a():
    print("I am coro_a(). Hi!")

async def coro_b():
    print("I am coro_b(). I sure hope no one hogs the event loop...")

async def main():
    task_b = asyncio.create_task(coro_b())
    num_repeats = 3
    for _ in range(num_repeats):
        await coro_a()
    await task_b

asyncio.run(main())
```

协程`main()`中的第一条语句创建了`task_b`并通过事件循环调度其执行。然后重复地等待`coro_a()`，在这个过程中控制权从未交还给事件循环。直到执行`await task_b`才会暂停`main()`的执行，然后事件循环才会切换到`coro_b()`。程序的输出如下：

```
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_b(). I sure hope no one hogs the event loop...
```

在这个示例中，控制权转移过程如下图所示。

![控制权转移过程时序图1](/assets/images/python-coroutine-and-asyncio/控制权转移过程时序图1.png)

如果将`await coro_a()`改为`await asyncio.create_task(coro_a())`，行为就会发生变化。协程`main()`会通过该语句将控制权交还给事件循环。然后事件循环会继续处理待执行的作业，先调用`task_b`，然后调用包装`coro_a()`的任务，最后恢复协程`main()`。

```
I am coro_b(). I sure hope no one hogs the event loop...
I am coro_a(). Hi!
I am coro_a(). Hi!
I am coro_a(). Hi!
```

![控制权转移过程时序图2](/assets/images/python-coroutine-and-asyncio/控制权转移过程时序图2.png)

这个例子说明了仅使用`await coroutine`可能会独占控制权并阻滞事件循环。`asyncio.run()`可以通过关键字参数`debug=True`来检测这种情况，这将启用[调试模式](https://docs.python.org/3/library/asyncio-dev.html#asyncio-debug-mode)。

### 1.5 异步迭代器和async for语句
[异步迭代器](https://docs.python.org/3/reference/datamodel.html#asynchronous-iterators)(asynchronous iterator)是提供了`__anext__()`方法的对象。该方法必须用`async def`定义（可以在其中调用异步代码），并返回一个可等待对象，其结果是迭代器的下一个值，当迭代结束时应该引发`StopAsyncIteration`。

异步可迭代对象(asynchronous iterable)是提供了`__aiter__()`方法的对象，该方法必须返回一个异步迭代器。异步迭代器也是异步可迭代对象，其`__aiter__()`方法返回自身。异步可迭代对象可以用于`async for`语句。

下面是一个异步迭代器的示例：

```python
class Reader:
    async def readline(self):
        ...

    def __aiter__(self):
        return self

    async def __anext__(self):
        val = await self.readline()
        if val == b'':
            raise StopAsyncIteration
        return val
```

[异步生成器](https://docs.python.org/3/reference/datamodel.html#asynchronous-generator-functions)是使用`async def`定义且包含`yield`语句的函数或方法。调用它会返回一个异步迭代器对象。

`async for`语句可以方便地对异步可迭代对象进行迭代。语法如下：

```python
async for target in expr:
    block
```

在语义上大致等价于

```python
it = expr.__aiter__()
while True:
    try:
        target = await it.__anext__()
        block
    except StopAsyncIteration:
        break
```

`async for`语句只能在协程函数内使用。例如：

```python
async def main():
    reader = Reader(...)
    async for line in reader:
        # process line
```

### 1.6 异步上下文管理器和async with语句
[异步上下文管理器](https://docs.python.org/3/reference/datamodel.html#asynchronous-context-managers)(asynchronous context manager)是提供了`__aenter__()`和`__aexit__()`方法的对象，这两个方法必须使用`async def`定义，并返回一个可等待对象。异步上下文管理器可以用于`async with`语句。

`async with`语句的语法如下：

```python
async with expr as target:
    block
```

在语义上大致等价于

```python
manager = expr
hit_except = False

try:
    target = await manager.__aenter__()
    block
except:
    hit_except = True
    if not await manager.__aexit__(*sys.exc_info()):
        raise
finally:
    if not hit_except:
        await manager.__aexit__(None, None, None)
```

`async with`语句只能在协程函数内使用。

## 2.工作原理
本节将介绍协程的控制权转移机制工作原理，以及如何自定义异步运算符。

### 2.1 控制权转移
前面已经提到，可等待对象的`__await__()`方法返回一个迭代器。协程的执行可以通过这个迭代器来控制。[协程对象](https://docs.python.org/3/reference/datamodel.html#coroutine-objects)具有以下与控制执行相关的方法（类似于[生成器方法](https://docs.python.org/3/reference/expressions.html#generator-iterator-methods)，详见[《Python基础教程》笔记 第9章]({% post_url 2024-02-21-python-note-ch09-magic-methods-properties-and-iterators %}) 9.7.4节）:
* `send(value)`：开始或恢复协程的执行。如果协程是首次执行，`value`必须为`None`；如果是恢复执行，则`value`将作为使协程暂停的（可等待对象的`__await__()`方法中）`yield`语句的返回值。该方法返回下一个`yield`语句产生的值，如果协程结束则引发`StopIteration`异常，并将返回值保存在其`value`属性中（如果有）。
* `throw(exc)`：在协程内引发指定的异常。如果该异常未在协程内被捕获，则传播给调用者。
* `close()`：使协程清理并退出。

假设`coro`是一个协程对象，`obj`是一个可等待对象，控制协程执行的各个阶段及转移控制权的方式如下：
* 开始：调用`coro.send(None)`，将控制权交给协程`coro`。
* 暂停：在协程内调用`await obj` → 调用`obj.__await__()`，执行到`yield value` → 沿着调用链向上传播 → `coro.send(None)`返回`value`，控制权交还给调用者。
* 恢复：调用`coro.send(value2)`，将控制权交给协程`coro` → `obj.__await__()`方法中的`yield value`语句返回`value2`，执行到`return value3` → 协程中的`await obj`返回`value3` → 协程继续执行。
* 结束：协程执行完成，返回`value4` → `coro.send(value2)`抛出`StopIteration(value4)`，控制权交还给调用者。

下面看一个例子，这个程序手动模拟了一个简单的事件循环：

```python
class Rock:
    def __await__(self):
        value_sent_in = yield 7
        print(f"Rock.__await__ resuming with value: {value_sent_in}.")
        return value_sent_in

async def main():
    print("Beginning coroutine main().")
    rock = Rock()
    print("Awaiting rock...")
    value_from_rock = await rock
    print(f"Coroutine received value: {value_from_rock} from rock.")
    return 23

coroutine = main()
intermediate_result = coroutine.send(None)
print(f"Coroutine paused and returned intermediate value: {intermediate_result}.")

print(f"Resuming coroutine and sending in value: 42.")
try:
    coroutine.send(42)
except StopIteration as e:
    returned_value = e.value
    print(f"Coroutine main() finished and provided value: {returned_value}.")
```

这个程序的执行过程如下：
1. 第15行：调用协程函数`main()`，创建协程对象`coroutine`。
2. 第16行：对协程对象调用`send(None)`，将控制权交给协程，协程**开始**执行。
3. 第11行：等待`rock`对象，调用其`__await__()`方法。
4. 第3行：语句`yield 7` **暂停**当前协程的执行，并沿着调用链向上传播。
5. 第16行：`send(None)`返回，并将`intermediate_result`赋值为7（第3行`yield`语句的参数），控制权交还给主程序。
6. 第21行：对协程对象调用`send(42)`，将控制权交给协程，协程**恢复**执行。
7. 第3行：`yield`语句返回，并将`value_sent_in`赋值为42（第21行`send()`方法的参数）。
8. 第5行：`__await__()`方法返回42。
9. 第11行：`await rock`返回，并将`value_from_rock`赋值为42（第5行`__await__()`方法的返回值），协程继续执行。
10. 第13行：协程执行**完成**，返回23。
11. 第23行：`send(42)`引发`StopIteration`异常，其`value`属性为23（第13行协程函数的返回值），将其赋值给`returned_value`，控制权交还给主程序。

程序将产生以下输出：

```
Beginning coroutine main().
Awaiting rock...
Coroutine paused and returned intermediate value: 7.
Resuming coroutine and sending in value: 42.
Rock.__await__ resuming with value: 42.
Coroutine received value: 42 from rock.
Coroutine main() finished and provided value: 23.
```

注：
* `Rock.__await__()`实际上就是一个普通的生成器函数。协程对象的`send()`、`throw()`和`close()`方法会委托给这个生成器的相应方法。
* `__await__()`方法可以包含多个`yield`语句。`await`表达式会调用被等待对象的`__await__()`方法获得一个生成器，每次调用协程对象的`send()`方法都会消费一个值，期间协程一直暂停在`await`，直到迭代结束协程才会继续执行。
* 自定义可等待对象的`__await__()`方法应该遵循这一规则：**当等待的条件不满足时`yield`，当条件满足时返回**。“等待的条件”取决于可等待对象的语义，示例见2.3节。
* 虽然[文档](https://docs.python.org/3/reference/datamodel.html#awaitable-objects)中只要求`__await__()`方法返回一个迭代器，但实际上必须是生成器。如果将上面程序中的`Rock.__await__()`方法改为返回`iter([7])`，`await rock`会引发异常 "AttributeError: 'list_iterator' object has no attribute 'send'" 。

### 2.2 Future
[Future对象](https://docs.python.org/3/library/asyncio-future.html#future-object)表示异步计算的结果（“可以在**未来**某个时间点获取的结果”）。Future对象有几个重要属性。一个是**状态**，可以是待处理(pending)、已取消(cancelled)或已完成(done)。另一个是**结果**，当状态为已完成时可以获取结果。

`asyncio.Task`继承了`asyncio.Future`。1.3节中提到的任务的回调函数列表实际上继承自`Future`类。

`asyncio.Future`的API类似于`concurrent.futures.Future`，但前者用于协程，而后者用于线程和进程。可以使用`set_result()`方法设置结果并将future标记为已完成，或者使用`set_exception()`方法设置异常。`result()`方法返回future的结果或引发设置的异常。`done()`和`cancelled()`检查future的状态是否是已完成或已取消。

Future对象是与事件循环绑定的，默认为当前正在运行的事件循环(`asyncio.get_running_loop()`)，可以在构造函数中通过关键字参数`loop`指定事件循环。也可以通过事件循环对象的`create_future()`方法创建future对象。

Future是可等待对象，协程可以等待future对象直到完成或取消。一个future可被多次等待，最终结果相同。

Future通常用于实现底层API和高层API的交互。经验法则是永远不要在面向用户的API中暴露future对象。

#### 源代码解析
前面提到过，协程、任务和future都是可等待对象。下面通过Python源代码解析各自的实现原理。

（1）Future

`asyncio.Future`类的`__await__()`方法简化的定义如下（源代码见[Lib/asyncio/futures.py](https://github.com/python/cpython/blob/3.14/Lib/asyncio/futures.py#L292)）：

```python
def __await__(self):
    if not self.done():
        yield self
    return self.result()
```

可以看到，等待一个future会检查其状态：如果状态是已完成则直接返回结果，否则`yield`使协程暂停，后续恢复时再获取结果（只有当future已完成时事件循环才会恢复那些等待它的协程）。也就是说，future等待的条件是**状态变为已完成**。

（2）任务

Future通常由其他协程调用`set_result()`来转入已完成状态，而任务在其包装的协程结束时将**自己**标记为已完成。`asyncio.Task`类的核心方法是`__step()`，简化的定义如下（源代码见[Lib/asyncio/tasks.py](https://github.com/python/cpython/blob/3.14/Lib/asyncio/tasks.py#L266)）：

```python
class Task(Future):
    def __init__(self, coro, loop=None):
        super().__init__(loop=loop)
        self.coro = coro
        loop.call_soon(self.__step)

    def __step(self):
        try:
            future = self.coro.send(None)
        except StopIteration as e:
            # coroutine finished
            super().set_result(e.value)
        except CancelledError:
            super().cancel()
        except BaseException as e:
            super().set_exception(e)
        else:
            # coroutine still awaiting future
            future.add_done_callback(self.__step)
```

`__step()`方法的核心逻辑如下：
* 对包装的协程`self.coro`调用`send(None)`，使其开始或恢复执行。
* 如果协程等待一个future对象，则`send()`返回这个对象（上面已经看到`Future.__await__()`方法会`yield`自身），并在`else`子句中将`self.__step`添加到其回调函数列表。
* 如果协程等待的future已完成，会通过其回调函数再次调用`__step()`方法。
* 如果协程执行结束，则`send()`会引发`StopIteration`，此时调用`set_result()`将自己标记为已完成。
* 如果任务被取消，则调用`cancel()`。
* 如果发生其他异常，则调用`set_exception()`，同样将自己标记为已完成。

`Task`直接继承了`Future`类的`__await__()`方法。因此，任务等待的条件是**包装的协程执行结束**。

（3）协程

协程对象的`__await__()`方法是用C代码实现的（源代码见[Objects/genobject.c](https://github.com/python/cpython/blob/3.14/Objects/genobject.c#L1116)）：

```c
static PyObject *
coro_await(PyObject *coro)
{
    PyCoroWrapper *cw = PyObject_GC_New(PyCoroWrapper, &_PyCoroWrapper_Type);
    cw->cw_coroutine = (PyCoroObject*)Py_NewRef(coro);
    return (PyObject *)cw;
}
```

该方法创建了一个协程包装类型(`PyCoroWrapper`)的对象，这个类型也有`send()`、`throw()`和`close()`方法，分别直接调用底层协程的对应方法。例如：

```c
static PyObject *
coro_wrapper_send(PyObject *self, PyObject *arg)
{
    PyCoroWrapper *cw = _PyCoroWrapper_CAST(self);
    return gen_send((PyObject *)cw->cw_coroutine, arg);
}
```

其中`gen_send()`就是协程对象`send()`方法的C代码。也就是说，当`await`一个协程时，调用其`__await__()`返回对象的`send()`方法等价于调用这个协程的`send()`方法。由于协程的`__await__()`方法不包含`yield`，因此不会使当前协程暂停。

小结：
* `await future`：暂停协程，等待future的状态变为已完成。
* `await task`：暂停协程，等待任务包装的协程执行结束。
* `await coroutine`：不暂停协程，直接调用另一个协程。

### 2.3 实现asyncio.sleep()
本节通过一个例子来说明如何利用future自己实现异步休眠功能（类似于`asyncio.sleep()`）。

`asyncio.sleep()`函数的作用是使当前协程暂停指定的秒数，同时不影响其他协程的运行。在协程中通过`await asyncio.sleep(n)`放弃控制权，并在（至少）n秒后重新获得控制权。

例如，协程`display_date()`每秒显示一次当前日期时间，持续5秒（间隔并非精确的1秒）：

```python
import asyncio
import datetime

async def display_date():
    loop = asyncio.get_running_loop()
    end_time = loop.time() + 5
    while True:
        print(datetime.datetime.now())
        if loop.time() + 1 >= end_time:
            break
        await asyncio.sleep(1)

asyncio.run(display_date())
```

使协程放弃控制权的唯一方式是`await`一个在其`__await__()`方法中`yield`的对象。换句话说，需要实现这样一个可等待对象：其`__await__()`方法在指定的结束时间之前始终`yield`（即使事件循环恢复当前协程也立即放弃控制权）；到达结束时间后立即返回，使当前协程可以继续执行。

下面的`AsyncSleep`类实现了这个功能：

```python
class AsyncSleep:
    def __init__(self, seconds):
        self.seconds = seconds

    def __await__(self):
        time_to_wake = time.time() + self.seconds
        while time.time() < time_to_wake:
            yield
```

现在将上面示例程序中的`await asyncio.sleep(1)`替换为`await AsyncSleep(1)`也可以实现同样的效果。

另外，也可以使用future实现：

```python
async def async_sleep(seconds):
    future = asyncio.Future()
    time_to_wake = time.time() + seconds
    # Add the watcher-task to the event loop.
    watcher_task = asyncio.create_task(_sleep_watcher(future, time_to_wake))
    # Block until the future is marked as done.
    await future

async def _sleep_watcher(future, time_to_wake):
    while time.time() < time_to_wake:
        await YieldToEventLoop()
    # This marks the future as done.
    future.set_result(None)

class YieldToEventLoop:
    def __await__(self):
        yield
```

运行协程`_sleep_watcher()`的任务`watcher_task`会在事件循环的每个完整周期中被调用一次。每次恢复时检查时间，如果还未经过足够的时间，则再次暂停并将控制权交还给事件循环。一旦经过了足够的时间，`_sleep_watcher()`会退出无限循环并将future标记为已完成。由于这个辅助任务在事件循环的每个周期中只会被调用一次，因此这个异步休眠会休眠**至少**一秒，而不是恰好一秒。`asyncio.sleep()`也是如此。

这个示例也可以不用future实现，如下所示：

```python
async def async_sleep(seconds):
    time_to_wake = time.time() + seconds
    while time.time() < time_to_wake:
        await YieldToEventLoop()
```

标准库`asyncio.sleep()`是基于future实现的，简化的定义如下（源代码见[Lib/asyncio/tasks.py](https://github.com/python/cpython/blob/3.14/Lib/asyncio/tasks.py#L687)）：

```python
async def sleep(seconds):
    loop = asyncio.get_running_loop()
    future = loop.create_future()
    loop.call_later(seconds, asyncio.Future.set_result, future, None)
    await future
```

`loop.call_later()`方法会使事件循环在指定的秒数后调用给定的函数。这种方式比前面几种实现的优势在于避免了不必要的恢复。

## 3.asyncio模块
Python标准库模块`asyncio`是用于编写异步代码的库，特别适合于I/O密集型操作以及构建高性能异步框架，例如网络请求、Web服务器、文件I/O、数据库连接、实时消息队列处理等。

官方文档：<https://docs.python.org/3/library/asyncio.html>

`asyncio`模块提供了一组[高层级API](https://docs.python.org/3/library/asyncio-api-index.html)，例如：
* [Runner](https://docs.python.org/3/library/asyncio-runner.html)和[任务](https://docs.python.org/3/library/asyncio-task.html)：运行协程
* [流](https://docs.python.org/3/library/asyncio-stream.html)：网络I/O
* [子进程](https://docs.python.org/3/library/asyncio-subprocess.html)：创建和管理子进程
* [队列](https://docs.python.org/3/library/asyncio-queue.html)：实现任务分配
* [同步原语](https://docs.python.org/3/library/asyncio-sync.html)：同步并发代码

此外，还有一些为库和框架开发者提供的[低层级API](https://docs.python.org/3/library/asyncio-llapi-index.html)，例如：
* [事件循环](https://docs.python.org/3/library/asyncio-eventloop.html)
* [Future](https://docs.python.org/3/library/asyncio-future.html)
* [传输和协议](https://docs.python.org/3/library/asyncio-protocol.html)

### 3.1 运行协程

| 函数 | 描述 | 示例 |
| --- | --- | --- |
| `run(coro)` | 运行一个协程 | `asyncio.run(main())` |
| `create_task(coro)` | 创建一个任务来调度协程执行 | `task = asyncio.create_task(fetch_data())` |
| `gather(*aws)` | 并发运行多个任务 | `await asyncio.gather(task1, task2)` |
| `sleep(delay)` | 异步休眠 | `await asyncio.sleep(1)` |
| `wait_for(aw, timeout)` | 带超时的等待 | `await asyncio.wait_for(long_task(), timeout=5)` |
| `to_thread(func, *args)` | 在单独的线程中运行函数 | `await asyncio.to_thread(blocking_io, arg1, arg2)` |

示例：

```python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number + 1):
        print(f"Task {name}: Compute factorial({number}), currently i={i}...")
        await asyncio.sleep(1)
        f *= i
    print(f"Task {name}: factorial({number}) = {f}")
    return f

async def main():
    # Schedule three calls *concurrently*:
    results = await asyncio.gather(
        factorial("A", 2),
        factorial("B", 3),
        factorial("C", 4),
    )
    print(results)

asyncio.run(main())
```

程序的输出如下：

```
Task A: Compute factorial(2), currently i=2...
Task B: Compute factorial(3), currently i=2...
Task C: Compute factorial(4), currently i=2...
Task A: factorial(2) = 2
Task B: Compute factorial(3), currently i=3...
Task C: Compute factorial(4), currently i=3...
Task B: factorial(3) = 6
Task C: Compute factorial(4), currently i=4...
Task C: factorial(4) = 24
[2, 6, 24]
```

在这个示例中，任务A、B、C同时运行，总运行时间约为3 s（如下图所示）。

![gather并发运行多个任务](/assets/images/python-coroutine-and-asyncio/gather并发运行多个任务.png)

假设`tasks`是一个任务对象的列表，`await asyncio.gather(*tasks)`的效果等价于`[await task for task in tasks]`。如果直接向`gather()`传递协程对象，会自动创建一个任务对其进行包装。因此上面的`main()`函数等价于

```python
async def main():
    tasks = [
        asyncio.create_task(factorial("A", 2)),
        asyncio.create_task(factorial("B", 3)),
        asyncio.create_task(factorial("C", 4)),
    ]
    results = [await task for task in tasks]
    print(results)
```

### 3.2 网络I/O

| 函数 | 描述 | 示例 |
| --- | --- | --- |
| `open_connection(host, port)` | 建立TCP连接 | `reader, writer = await asyncio.open_connection('127.0.0.1', 8888)` |
| `start_server(cb, host, port)` | 启动TCP服务器 | `server = await asyncio.start_server(echo_handle, '0.0.0.0', 8888)` |

例如，使用`open_connection()`实现的TCP echo客户端：

```python
import asyncio

async def tcp_echo_client(message):
    reader, writer = await asyncio.open_connection('127.0.0.1', 8888)

    print(f'Send: {message!r}')
    writer.write(message.encode())
    await writer.drain()

    data = await reader.read(100)
    print(f'Received: {data.decode()!r}')

    print('Close the connection')
    writer.close()
    await writer.wait_closed()

asyncio.run(tcp_echo_client('Hello World!'))
```

使用`start_server()`实现的TCP echo服务器：

```python
import asyncio

async def handle_echo(reader, writer):
    data = await reader.read(100)
    message = data.decode()
    addr = writer.get_extra_info('peername')

    print(f"Received {message!r} from {addr!r}")

    print(f"Send: {message!r}")
    writer.write(data)
    await writer.drain()

    print("Close the connection")
    writer.close()
    await writer.wait_closed()

async def main():
    server = await asyncio.start_server(handle_echo, '0.0.0.0', 8888)

    addrs = ', '.join(str(sock.getsockname()) for sock in server.sockets)
    print(f'Serving on {addrs}')

    async with server:
        await server.serve_forever()

asyncio.run(main())
```

### 3.3 子进程

| 函数 | 描述 | 示例 |
| --- | --- | --- |
| `create_subprocess_exec(prog, *args)` | 创建子进程 | `proc = await asyncio.create_subprocess_exec('python', '-V)` |
| `create_subprocess_shell(cmd)` | 运行shell命令 | `proc = await asyncio.create_subprocess_shell('ls -l')` |

这两个函数都返回一个`asyncio.subprocess.Process`对象用于控制子进程，使用`StreamReader`类读取其标准输出。

例如，下面的程序使用`create_subprocess_exec()`函数在子进程中调用`python -c`命令执行Python代码，打印当前日期：

```python
import asyncio
import sys

async def get_date():
    code = 'import datetime; print(datetime.datetime.now())'

    # Create the subprocess; redirect the standard output into a pipe.
    proc = await asyncio.create_subprocess_exec(
        sys.executable, '-c', code,
        stdout=asyncio.subprocess.PIPE)

    # Read one line of output.
    data = await proc.stdout.readline()
    line = data.decode().rstrip()

    # Wait for the subprocess exit.
    await proc.wait()
    return line

date = asyncio.run(get_date())
print(f"Current date: {date}")
```

### 3.4 队列
`asyncio.Queue`的API类似于`queue.Queue`，但前者用于协程，而后者用于线程。

队列可用于在多个协程之间分配工作，实现连接池以及发布/订阅模式或生产者/消费者模式。下面的程序使用队列向多个并发的协程分配工作：

```python
import asyncio
import random
import time


async def worker(name, queue):
    while True:
        # Get a "work item" out of the queue.
        sleep_for = await queue.get()

        # Sleep for the "sleep_for" seconds.
        await asyncio.sleep(sleep_for)

        # Notify the queue that the "work item" has been processed.
        queue.task_done()

        print(f'{name} has slept for {sleep_for:.2f} seconds')


async def main():
    # Create a queue that we will use to store our "workload".
    queue = asyncio.Queue()

    # Generate random timings and put them into the queue.
    total_sleep_time = 0
    for _ in range(20):
        sleep_for = random.uniform(0.05, 1.0)
        total_sleep_time += sleep_for
        queue.put_nowait(sleep_for)

    # Create three worker tasks to process the queue concurrently.
    tasks = []
    for i in range(3):
        task = asyncio.create_task(worker(f'worker-{i}', queue))
        tasks.append(task)

    # Wait until the queue is fully processed.
    started_at = time.monotonic()
    await queue.join()
    total_slept_for = time.monotonic() - started_at

    # Cancel our worker tasks.
    for task in tasks:
        task.cancel()
    # Wait until all worker tasks are cancelled.
    await asyncio.gather(*tasks, return_exceptions=True)

    print('====')
    print(f'3 workers slept in parallel for {total_slept_for:.2f} seconds')
    print(f'total expected sleep time: {total_sleep_time:.2f} seconds')


asyncio.run(main())
```

这个示例使用`sleep()`模拟工作耗时，一次运行的输出如下。可以看到，所有任务需要的总耗时为10.72 s，而3个协程并发执行这些任务只用了3.94 s。

```
3 workers slept in parallel for 3.94 seconds
total expected sleep time: 10.72 seconds
```

### 3.5 同步原语

| 类 | 描述 | 示例 |
| --- | --- | --- |
| `Lock` | 互斥锁 | 创建：`lock = asyncio.Lock()`<br>加锁：`async with lock:` |
| `Event` | 事件通知 | 创建：`event = asyncio.Event()`<br>等待者：`await event.wait()`<br>通知者：`event.set()` |
| `Condition` | 条件变量 | 创建：`cond = asyncio.Condition()`<br>等待者：`async with cond: await cond.wait()`<br>通知者：`async with cond: cond.notify()` |
| `Semaphore` | 信号量 | 创建：`sem = asyncio.Semaphore(10)`<br>使用：`async with sem:` |

这些同步原语与`threading`模块中的类似。有些类是异步上下文管理器，可以通过`async with`语句使用。例如：

```python
lock = asyncio.Lock()

# ... later
async with lock:
    # access shared state
```

等价于

```python
lock = asyncio.Lock()

# ... later
await lock.acquire()
try:
    # access shared state
finally:
    lock.release()
```

### 3.6 事件循环

| 函数 | 描述 |
| --- | --- |
| `get_running_loop()` | 获取当前运行的事件循环 |
| `new_event_loop()` | 创建一个新的事件循环 |

事件循环的主要方法：

| 方法 | 描述 | 示例 |
| --- | --- | --- |
| `run_until_complete(future)` | 运行可等待对象直到完成 | `loop.run_until_complete(main())` |
| `run_forever()` | 永久运行事件循环（直到调用`stop()`） | |
| `stop()` | 停止事件循环 | |
| `close()` | 关闭事件循环 | |
| `call_soon(func, *args)` | 尽快调用函数 | `loop.call_soon(print, 'Hello')` |
| `call_later(delay, func, *args)` | 在`delay`秒后调用函数 | `loop.call_later(5, print, 'Hello')` |
| `call_at(when, func, *args)` | 在绝对时间戳`when`调用函数 | `loop.call_at(loop.time() + 5, print, 'Hello')` |
| `create_future()` | 创建一个附加到事件循环的future对象 | |
| `create_task(coro)` | 创建一个任务对象，将协程加入事件循环 | |
| `run_in_executor(executor, func, *args)` | 在线程池或进程池中运行函数 | [Executing code in thread or process pools](https://docs.python.org/3/library/asyncio-eventloop.html#executing-code-in-thread-or-process-pools) |

`asyncio.run()`函数底层就是使用`get_running_loop()`、`loop.create_task()`和`loop.run_until_complete()`实现的。

### 3.7 第三方库
有很多基于`asyncio`构建的异步I/O库，参见 <https://github.com/timofurrer/awesome-asyncio> 。

## 4.最佳实践

1. 不要阻塞事件循环，例如在协程中调用`time.sleep()`或耗时的同步操作。
2. 尽量使用原生支持异步的库（如aiohttp）。如果必须使用同步库（如requests），则使用`to_thread()`或`loop.run_in_executor()`在线程池中执行。
3. 适用于I/O密集型操作。对于CPU密集型操作，使用`loop.run_in_executor()`在进程池中执行（由于[GIL](https://docs.python.org/3/glossary.html#term-global-interpreter-lock)）。

## 5.实践：网络爬虫
最后，使用`asyncio`和aiohttp库编写一个网络爬虫程序。

[aiohttp](https://docs.aiohttp.org/en/stable/)是一个异步HTTP库。与requests不同，这个库提供的API都是异步的（例如发送HTTP请求和获取响应体），可以和`asyncio`模块一起使用。使用以下命令安装：

```shell
pip install aiohttp
```

下面的程序从 <http://quotes.toscrape.com/> 爬取名人语录并保存到文件。

async_quotes_crawler.py

```python
import asyncio
import re
import time

import aiohttp


def parse(text):
    quote_pattern = re.compile(r'<span class="text" itemprop="text">(.+?)</span>')
    author_pattern = re.compile(r'<small class="author" itemprop="author">(.+?)</small>')
    quotes = quote_pattern.findall(text)
    authors = author_pattern.findall(text)
    return list(zip(quotes, authors))


async def fetch_url(session, url):
    async with session.get(url) as response:
        text = await response.text()
        return parse(text)


async def main():
    url = 'http://quotes.toscrape.com/page/{:d}/'
    start_time = time.time()

    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url.format(page)) for page in range(1, 6)]
        results = await asyncio.gather(*tasks)

    end_time = time.time()
    print('Total time: {:.2f} s'.format(end_time - start_time))

    with open('quotes.txt', 'w', encoding='utf-8') as f:
        for result in results:
            f.writelines([f'{quote} by {author}\n' for quote, author in result])
    print(f'Results saved to {f.name}')


if __name__ == '__main__':
    asyncio.run(main())
```

## 参考
* [PEP 492](https://peps.python.org/pep-0492/)：协程的`async/await`语法
* [PEP 3156](https://peps.python.org/pep-3156/)：`asyncio`模块
* [Python HOWTOs - A Conceptual Overview of asyncio](https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html)
* [Python标准库 - asyncio模块](https://docs.python.org/3/library/asyncio.html)
* [Python 异步协程：从 async/await 到 asyncio 再到 async with](https://www.cnblogs.com/piperliu/articles/18625027)
* [A Web Crawler With asyncio Coroutines](https://aosabook.org/en/500L/a-web-crawler-with-asyncio-coroutines.html)
