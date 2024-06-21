---
title: 《Python基础教程》笔记 第24章 项目5：虚拟茶话会
date: 2024-06-10 17:20:55 +0800
categories: [Python, Beginning Python]
tags: [python, socket, asynchronous]
---
在这个项目中，我们将做些正式的网络编程。我们将编写一个聊天服务器，让人们能够通过网络实时聊天。使用Python创建这种程序的方式有很多。在本章中，将使用标准库中的异步网络模块。

值得注意的是，本章使用的`asyncore`和`asynchat`模块已在Python 3.6中弃用、在3.12中移除。如果愿意，可以尝试使用第14章讨论的其他方法（例如分叉或线程），甚至使用新的`asyncio`模块重写这个项目。

## 24.1 问题描述
我们将编写一个相对低层次的在线聊天服务器。假设有如下需求：
* 服务器应该能接受来自不同用户的多个连接。
* 应该允许用户**并行**操作。
* 应该能够解释命令（例如`say`或`logout`）。
* 服务器应该易于扩展。

其中网络连接和程序的异步特性需要使用特殊工具来实现。

## 24.2 有用的工具
在这个项目中，需要的新工具只有标准库模块`asyncore`及其相关的模块`asynchat`。第14章讨论过，网络程序的基本组件是**套接字**。`asyncore`框架让你能够处理多个同时连接的用户。编写聊天服务器时，关键就是允许多个用户同时连接，不然用户之间如何聊天呢？

`asyncore`框架基于的底层机制（14.3.2节讨论的`select()`函数）让服务器能够以“碎片化”的方式为所有连接的用户提供服务：不是读取来自一个用户的**所有**数据后再读取下一个用户，而是只读取**部分**数据。另外，服务器只读取**有数据可读取的**套接字。这种操作是在循环中反复进行的。对写入的处理与此类似。你可以使用`socket`和`select`模块自己实现这种功能，但`asyncore`和`asynchat`提供了一个很有用的框架，可以替你处理这些细节。（实现并行用户连接的其他方式参见14.3节。）

### 工作原理
通过阅读标准库源代码了解`asyncore`和`asynchat`模块的工作原理。

#### asyncore
`asyncore`模块提供的主要接口是`dispatcher`类和`loop()`函数。
* `dispatcher`类是对`socket`对象的简单包装，提供了基本的套接字方法（`accept()`、`recv()`、`send()`等）。另外还提供了事件处理器方法（例如`handle_accept()`、`handle_read()`、`handle_write()`等）供子类覆盖，这些方法将被`loop()`自动调用。
* `dispatcher`对象在创建套接字时，会将自身添加到一个全局映射，键为套接字的文件描述符：`socket_map[sock.fileno()] = self`。
* `loop()`函数是一个轮询循环，不断地遍历上述全局映射，基于`select.select()`函数获取准备好读和写的`dispatcher`对象，并分别调用其`handle_read_event()`和`handle_write_event()`方法。而这两个方法根据套接字的当前状态调用`handle_accept()`、`handle_connect()`、`handle_read()`或`handle_write()`方法。（即`loop()`函数相当于代码清单14-6中的主循环。）

#### asynchat
`asynchat`模块提供的主要接口是`async_chat`类。
* `async_chat`是`asyncore.dispatcher`的子类，在其基础上实现了按行接收数据的逻辑（类似于Twisted的`LineReceiver`）。
* 这个类覆盖了`handle_read()`和`handle_write()`方法：
  * `handle_read()`方法使用`recv()`读取一部分数据，如果遇到行结束符则调用`found_terminator()`，否则调用`collect_incoming_data()`。使用`set_terminator()`设置行结束符。
  * `handle_write()`方法调用`initiate_send()`，而该方法从一个队列(producer)中获取数据，并调用`send()`发送数据。
  * 要在子类中写入数据，应该调用`push(data)`。该方法将数据压入队列，并调用`initiate_send()`。
* 要使用这个类，必须继承它，并实现`collect_incoming_data()`和`found_terminator()`两个方法。

## 24.3 准备工作
首先，你需要一台连接到网络（比如互联网）的计算机，否则别人将无法连接到你的聊天服务器。要连接到聊天服务器，用户必须知道服务器的地址（机器名或IP地址）和端口号。在本章的代码中，使用（随便选择的）端口号5005。

为了测试聊天服务器，需要一个**客户端**——位于用户侧的程序。一个简单的这种程序是Telnet（基本上能够让你连接到任何套接字服务器）。在UNIX中，可以从命令行执行这个程序。

```shell
$ telnet some.host.name 5005
```

这个命令连接到机器some.host.name的5005端口。要连接到当前机器，只需使用机器名localhost。

在Windows中，可以使用系统自带的Telnet客户端：控制面板→程序→启用或关闭Windows功能→Telnet客户端，开启后可以在CMD中执行`telnet`命令。

![Windows启用Telnet客户端](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/Windows启用Telnet客户端.png)

## 24.4 初次实现
下面将程序稍做分解。需要创建两个主要的类：一个表示聊天服务器，另一个表示聊天会话（连接的用户）。

### 24.4.1 ChatServer类
要创建`ChatServer`类，可以继承`asyncore`模块的`dispatcher`类。`dispatcher`基本上就是一个套接字对象，但有一些额外的事件处理特性，稍后就会用到。代码清单24-1是一个基本的聊天服务器（几乎什么都没做）。

[代码清单24-1 极简服务器程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/minimal_chat_server.py)

如果运行这个程序，什么都不会发生。要让服务器做点有趣的事情，需要调用其`create_socket()`方法来创建一个套接字，以及`bind()`和`listen()`方法将套接字绑定到特定的端口并监听连接（毕竟这是服务器要做的事情）。另外，还需要覆盖`handle_accept()`方法，在服务器接受客户端连接时做些事情。最后调用`asyncore.loop()`启动服务器的监听循环。修改后的程序如代码清单24-2所示。

[代码清单24-2 能够接受连接的服务器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/chat_server_accept_connections.py)

`handle_accept()`方法调用`self.accept()`，接受客户端连接并返回一个连接（客户端套接字）和一个地址，`addr[0]`是客户端的IP地址。`handle_accept()`方法没有使用返回的连接做任何有用的事情（读写数据），而只是打印了一条客户端试图连接的消息。

尝试运行这个服务器，然后使用客户端连接到它。可以使用`telnet`命令：

```shell
$ telnet localhost 5005
```

也可以使用Python标准库模块`telnetlib`：

```shell
$ python -m telnetlib localhost 5005
```

客户端会立即断开连接，而服务器将打印以下内容：

```
Connection attempt from 127.0.0.1
```

要停止服务器，只需使用键盘中断：在UNIX中为Ctrl+C，在Windows中为Ctrl+Break（或Ctrl+C）（注：按下后可能需要等待一段时间才会停止）。

使用键盘中断关闭服务器会打印栈跟踪(`KeyboardInterrupt`)。为了避免这种情况，可以将`loop()`放在`try/except`语句中。添加一些清理代码后，这个基本服务器如代码清单24-3所示。

[代码清单24-3 包含清理代码的基本服务器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/chat_server_with_cleanup.py)

添加的`set_reuse_addr()`调用让你能够重用相同的端口号，即使服务器没有正确关闭（否则可能需要等待一段时间才能再次启动服务器）。

### 24.4.2 ChatSession类
基本的`ChatServer`类不是很有用，应该为每个连接创建一个新的`dispatcher`对象。然而，这些对象的行为与用作主服务器的对象不同。它们不在端口监听连接，而是已经连接到一个客户端。它们的主要任务是收集来自客户端的数据（文本）并做出响应（注：二者的区别相当于代码清单14-1中的服务器套接字`s`和`accept()`返回的客户端套接字`c`）。你可以通过继承`dispatcher`并覆盖各种方法来自己实现这种功能，但所幸有一个模块已经完成了大部分工作：`asynchat`。

尽管叫这个名字，`asynchat`并不是专门为我们正在做的聊天应用设计的（名字中的chat指的是“聊天式”或者命令-响应协议）。`asynchat`模块中有一个`async_chat`类，其好处是隐藏了大部分基本的套接字读写操作。为了让它发挥作用，只需覆盖两个方法：`collect_incoming_data()`和`found_terminator()`。前者在每次从套接字读取一些文本后调用，后者在读取到**结束符**时调用。你需要在初始化时调用`set_terminator()`设置结束符（在这里是换行符）。

更新后带有`ChatSession`类的程序如代码清单24-4所示。

[代码清单24-4 包含ChatSession类的服务器程序](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/chat_server_with_session.py)

对于这个新版本有几点值得注意。
* `set_terminator()`方法将行结束符设置为`b'\r\n'`，这是网络协议中常用的行结束符。
* `ChatSession`对象将已读取的数据存储在`bytes`列表`data`中。当读取到更多数据时，`collect_incoming_data()`将被自动调用，它只是将数据添加到列表中。使用字符串（或`bytes`）列表存储数据、之后使用`join()`连接是一个常见的习惯用法。（历史上这比`+=`更高效，而现在完全可以使用`+=`）
* `found_terminator()`方法在遇到结束符时被调用。当前的实现将数据拼接为一行（并解码），并将`self.data`重置为空列表。然而，目前还未使用这行做任何有用的事，只是将其打印出来。
* `ChatServer`保存了一个会话列表，`handle_accept()`方法创建一个新的`ChatSession`，并将其添加到会话列表。

尝试运行这个服务器，并同时使用两个或多个客户端连接到它。在客户端输入的每一行都将在运行服务器的终端打印出来。这意味着服务器能够同时处理多个连接。现在唯一缺少的功能是让客户端能够看到其他人的发言。

注意：
* 在Python 3中，通过网络发送和接收的数据都是`bytes`而不是字符串！因此`async_chat`类的`collect_incoming_data()`和`set_terminator()`等方法的参数都是`bytes`类型。
* 在Windows和Linux中，Telnet客户端（`telnet`命令）使用的换行符都是`\r\n`；而Python的`telnetlib`模块使用的换行符是`\n`（因此如果将行结束符设置为`\r\n`，行将永远不会结束）。
* Windows Telnet客户端默认编码是GBK，而Linux是UTF-8。
* Windows Telnet客户端在发送中文数据时会先发送第一个字节，再发送其他字节。因此必须先收集全部数据，在`found_terminator()`拼接后再解码（否则可能解码失败），如下图所示。

![Windows Telnet客户端发送中文数据](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/Windows Telnet客户端发送中文数据.png)

### 24.4.3 整合起来
要让原型成为功能完整（虽然简单）的聊天服务器，还缺少一项主要功能：将用户的发言（输入的每一行）广播给其他用户。要实现这种功能，可以在服务器中使用一个简单的`for`循环遍历会话列表，并将内容行写入每个会话。要将数据写入`async_chat`对象，使用`push()`方法。这种广播行为也带来了一个问题：客户端断开连接后，必须确保将其从会话列表中删除。为此，可以覆盖事件处理方法`handle_close()`。

第一个原型的最终版本如代码清单24-5所示。

[代码清单24-5 简单的聊天服务器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/simple_chat.py)

注：建立连接和聊天时的交互关系图如下。

![交互关系图-建立连接](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/交互关系图-建立连接.png)

![交互关系图-聊天](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/交互关系图-聊天.png)

原型程序的运行结果如下图所示（其中红框内为当前客户端发送的内容）。

![原型运行结果-Alice](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/原型运行结果-Alice.png)

![原型运行结果-Bob](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/原型运行结果-Bob.png)

## 24.5 再次实现
第一个原型算是个功能齐全的聊天服务器，但其功能很有限。最明显的限制是无法辨别谁说了什么。另外，它也不能解释命令（例如`say`或`logout`），而这是最初的规范所要求的。因此，需要添加对身份（每个用户一个唯一的名字）和命令解释的支持，同时必须让每个会话的行为都依赖于其所处的状态（已连接、已登录等）。所有这些都以一种易于扩展的方式完成。

### 24.5.1 基本命令解释
下面将演示如何模仿标准库模块`cmd`中`Cmd`类的命令解释功能。你需要一个可以处理（用户输入的）单行文本的方法，它应该截取第一个单词（命令），并根据这个单词调用相应的方法。例如，这一行：

```
say Hello, world!
```

会调用

```python
do_say('Hello, world!')
```

可能还会将会话本身作为参数（从而`do_say()`知道是谁在说话）。

下面是一种简单的实现，其中还包含一个处理未知命令的方法。

```python
class CommandHandler:

    def unknown(self, session, cmd):
        session.push('Unknown command: {}\r\n'.format(cmd).encode())

    def handle(self, session, line):
        if not line.strip():
            return
        parts = line.split(' ', 1)
        cmd = parts[0]
        try:
            line = parts[1].strip()
        except IndexError:
            line = ''
        meth = getattr(self, 'do_' + cmd, None)
        try:
            meth(session, line)
        except TypeError:
            self.unknown(session, cmd)
```

在这个类中`getattr()`的用法类似于第20章的标记项目。实现了基本的命令处理功能后，需要定义一些真正的命令。有哪些命令可用（以及它们做什么）应该取决于会话的当前状态。那么如何表示会话的状态呢？

### 24.5.2 聊天室
每种状态都可以用一个自定义的命令处理器表示。每个聊天室(room)都是一个支持特定命令的`CommandHandler`。另外，它还应该记录聊天室内有哪些用户（会话）。下面是所有聊天室的通用超类：

```python
class EndSession(Exception):
    pass

class Room(CommandHandler):

    def __init__(self, server):
        self.server = server
        self.sessions = []

    def add(self, session):
        self.sessions.append(session)

    def remove(self, session):
        self.sessions.remove(session)

    def broadcast(self, line):
        for session in self.sessions:
            session.push(line.encode())

    def do_logout(self, session, line):
        raise EndSession
```

除了基本的`add()`和`remove()`方法外，还包含`broadcast()`方法，该方法对聊天室内的所有用户（会话）调用`push()`。这个类还定义了一个`logout`命令（`do_logout()`方法），它会引发`EndSession`异常，而该异常将在较高的层次（`found_terminator()`中）处理。

### 24.5.3 登录和退出聊天室
除了表示常规聊天室（这个项目中只有一个这样的聊天室）之外，`Room`的子类还可以表示其他状态，这正是我们的意图。例如，当用户连接到服务器时，将进入专用的`LoginRoom`（没有其他用户）。`LoginRoom`在用户进入时打印一条欢迎消息（在`add()`中）。它还覆盖了`unknown()`方法，让用户登录。它只响应`login`命令，这个命令检查用户名是否可接受（不是空字符串，且未被其他用户使用）。

`LogoutRoom`要简单得多。它唯一的职责是将用户的名字从服务器中删除（服务器包含存储会话的字典`users`）。如果用户名不存在（因为用户从未登录），将忽略因此引发的`KeyError`。

这两个类的源代码参见代码清单24-6。

注意：尽管服务器的`users`字典存储了用户名对应的会话，但并没有从中获取会话，而只用于记录哪些用户名被占用。然而，我没有将用户名关联到随便选择的值（例如`True`），而是将其关联到对应的会话。虽然现在没有用到，但在以后的程序版本中可能发挥作用（例如让用户能够发私信）。

### 24.5.4 主聊天室
主聊天室`ChatRoom`也覆盖了`add()`和`remove()`方法。`add()`方法广播一条消息（有用户进入），并将用户的名字添加到服务器的`users`字典中。`remove()`方法广播一条消息（有用户离开）。

除了这些方法外，`ChatRoom`类还实现了三个命令：
* `say`命令广播一行内容，并在开头指出是哪个用户说的。
* `look`命令告诉用户聊天室内当前有哪些用户。
* `who`命令告诉用户当前有哪些用户已登录。在这个简单的服务器中，`look`和`who`是等价的，但如果你将其扩展为包含多个房间，这两个命令的功能就不同了。

这个类的源代码参见代码清单24-6。

### 24.5.5 新的服务器
至此已经介绍了大部分功能。对于`ChatSession`和`ChatServer`的主要改进如下：
* `ChatSession`新增了`enter()`方法，用于进入新的聊天室。
* `ChatSession`的构造函数使用了`LoginRoom`，`handle_close()`方法使用了`LogoutRoom`。
* `ChatServer`的构造函数新增了字典属性`users`和`ChatRoom`属性`main_room`。

注意，`handle_accept()`不再将新的`ChatSession`添加到会话列表，因为现在会话由聊天室管理。

注意：一般而言，如果你只实例化一个对象，而不将其赋给变量或添加到容器（就像`handle_accept()`中的`ChatSession`），它将丢失并可能被垃圾收集。由于所有的`dispatcher`都由`asyncore`处理（引用），而`async_chat`是`dispatcher`的子类，因此在这里不是问题。（注：见24.2节中的“工作原理”）

聊天服务器的最终版本如代码清单24-6所示。

[代码清单24-6 稍复杂些的聊天服务器](https://github.com/ZZy979/Beginning-Python-code/blob/main/ch24/chatserver.py)

下表列出了聊天服务器支持的命令：

| 命令 | 可用于 | 描述 |
| --- | --- | --- |
| `login name` | 登录房间 | 用于登录服务器 |
| `logout` | 所有房间 | 用于退出服务器 |
| `say statement` | 聊天室 | 用于发言 |
| `look` | 聊天室 | 用于查看谁在同一个聊天室 |
| `who` | 聊天室 | 用于查看谁登录到了服务器 |

注：一个客户端在连接-聊天-退出过程中的状态转移图如下。

![状态转移图](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/状态转移图.png)

聊天会话示例如下图所示。

![运行结果-magnus](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/运行结果-magnus.png)

![运行结果-dilbert](/assets/images/python-note-ch24-project-5-a-virtual-tea-party/运行结果-dilbert.png)

## 24.6 进一步探索
对于本章介绍的基本服务器，可以在很多方面进行扩展和改进。
* 可以创建包含多个聊天室的版本，还可以按自己的想法扩展命令集。
* 可以让程序只能识别某些命令（例如`login`或`logout`），并将其他文本都视为聊天内容，这样就不需要`say`命令了。
* 可以在所有命令前添加一个特殊字符（例如斜杠，让命令类似于`/login`或`/logout`），并将不以特殊字符开头的内容都视为聊天内容。
* 你可能想创建自己的GUI客户端，但这比想像的要难一些。GUI工具包有一个事件循环，而与服务器通信可能需要另一个循环。为了让它们协作，你可能需要使用线程。有关如何使用线程的简单示例（各个线程不能直接访问其他线程的数据），参见第28章。
