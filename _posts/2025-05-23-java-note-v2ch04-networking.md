---
title: 《Java核心技术》笔记 卷II 第4章 网络
date: 2025-05-23 19:53:29 +0800
categories: [Java, Core Java]
tags: [java, computer network, socket, http]
---
本章首先回顾网络相关的基本概念，然后编写连接到网络服务的Java程序，并展示网络客户端和服务器是如何实现的。最后将介绍如何用Java程序发送E-mail，以及如何从Web服务器获取信息。

## 4.1 连接到服务器
### 4.1.1使用Telnet
Telnet程序是一个很好的网络编程调试工具，可以在命令行中输入`telnet`启动它。

注释：在Windows中，需要手动开启Telnet。打开控制面板→程序→启用或关闭Windows功能，勾选“Telnet客户端”复选框。

![Windows启用Telnet客户端](/assets/images/java-note-v2ch04-networking/Windows启用Telnet客户端.png)

你可以使用Telnet连接到远程计算机，也可以用它与因特网主机提供的其他服务进行通信。例如，在命令行输入

```shell
telnet time-a.nist.gov 13
```

应该得到像下面这样的一行：

```
57488 16-04-10 04:23:00 50 0 0 610.5 UTC(NIST) *
```

![时间服务的输出](/assets/images/java-note-v2ch04-networking/时间服务的输出.png)

刚刚连接的是由美国国家标准技术研究所(National Institute of Standards and Technology)的服务器提供的时间("time of day")服务，提供铯原子钟的计量时间。按照惯例，时间服务的端口号是13。

下面是另一个类似的实验。在命令行输入

```shell
telnet horstmann.com 80
```

然后输入以下内容（在末尾按两次Enter键）：

```
GET / HTTP/1.1
Host: horstmann.com

```

下图显示了响应结果。它是一个HTML格式的文本页面，即Cay Horstmann的主页。这与Web浏览器获取网页的过程完全相同，它使用HTTP向服务器请求网页。

注：这里得到的实际上是一个重定向页面，而不是真正的主页。因为网站要求HTTPS连接，访问HTTP端口(80)会被重定向到HTTPS，但Telnet无法直接连接到HTTPS端口(443)。

![使用Telnet访问HTTP端口](/assets/images/java-note-v2ch04-networking/使用Telnet访问HTTP端口.png)

### 4.1.2 用Java连接到服务器
程序清单4-1是第一个网络程序，它与上一节使用Telnet所做的事相同——连接到端口并打印出响应内容。

[程序清单4-1 socket/SocketTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/socket/SocketTest.java)

下面是这个简单程序的关键代码：

```java
var s = new Socket("time-a.nist.gov", 13);
var in = new Scanner(s.getInputStream(), StandardCharsets.UTF_8);
```

第一行代码打开一个**套接字**(socket)，它是网络中的一个抽象概念，允许程序与外部通信。将远程地址和端口号传给套接字的构造器，如果连接失败则抛出`UnknownHostException`，如果发生其他问题则抛出`IOException`。

一旦打开了套接字，就可以使用`getInputStream()`方法获得一个输入流，用于读取远程服务的响应。该程序直接把每一行打印到标准输出，直到服务器断开连接。

这个程序只适用于非常简单的服务器（例如时间服务）。在更复杂的网络程序中，客户端向服务器发送请求数据，服务器可能不会在响应结束时立即断开连接。

`Socket`类非常简单易用，因为Java库隐藏了建立网络连接和通过连接发送数据的复杂性。`java.net`包提供的编程接口与操作文件使用的接口基本相同。

### 4.1.3 套接字超时
读取套接字会阻塞直到数据可用。如果主机不可达，程序将会一直等待，直到底层操作系统最终超时。

可以调用`Socket`类的`setSoTimeout()`方法设置超时时间（单位毫秒）。

```java
var s = new Socket(...);
s.setSoTimeout(10000); // time out after 10 seconds
```

如果套接字设置了超时时间，后续读操作在超时时会抛出`SocketTimeoutException`。

```java
try {
    InputStream in = s.getInputStream(); // read from in
    ...
}
catch (SocketTimeoutException e) {
    // react to timeout
}
```

写操作没有超时时间。

还有一个超时问题需要解决：构造器`Socket(host, port)`会一直阻塞，直到建立了到主机的初始连接。为了解决这个问题，可以先（使用默认构造器）构造一个未连接的套接字，然后使用超时进行连接：

```java
var s = new Socket();
s.connect(new InetSocketAddress(host, port), timeout);
```

### 4.1.4 IP地址
因特网协议(Internet Protocol, IP)地址是用数字表示的主机地址，由4个字节组成（在IPv6中是16个字节），例如129.6.15.28。

静态方法`InetAddress.getByName()`返回主机名对应的一个IP地址。例如，

```java
InetAddress address = InetAddress.getByName("time-a.nist.gov"); // 129.6.15.28
```

IP地址可以表示为4个由点分隔的整数，每个整数的范围为0~255（刚好对应一个“无符号字节”）。因此IP地址也可以表示为长度为4的`byte[]`数组。可以使用`getAddress()`方法得到字节表示：

```java
byte[] addressBytes = address.getAddress(); // {-127, 6, 15, 28}
```

一些流量较大的主机名（如google.com）会对应多个IP地址，以实现负载均衡。当访问主机时，会随机选择其中的一个。可以使用`getAllByName()`方法获得主机名的所有IP地址。

```java
InetAddress[] addresses = InetAddress.getAllByName(host);
```

有时需要本地主机的IP地址。如果调用`getByName("localhost")`总是会得到本地回环地址127.0.0.1，其他程序无法用这个地址来连接到这台计算机。应该使用静态方法`getLocalHost()`：

```java
InetAddress address = InetAddress.getLocalHost();
```

程序清单4-2中的简单程序打印在命令行中指定的主机名的所有IP地址。例如：

```shell
$ java inetAddress/InetAddressTest www.horstmann.com
www.horstmann.com/68.66.226.114
```

[程序清单4-2 inetAddress/InetAddressTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/inetAddress/InetAddressTest.java)

## 4.2 实现服务器
上一节已经实现了一个从因特网接收数据的基本网络客户端。在这一节将编写一个可以向客户端发送信息的简单服务器。

### 4.2.1 服务器套接字
服务器程序启动后就会等待客户端连接到它的端口。在我们的示例程序中，选择端口号8189。`ServerSocket`类用于建立服务器套接字：

```java
var s = new ServerSocket(8189);
```

下面的命令告诉服务器程序一直等待，直到有客户端连接到这个端口：

```java
Socket incoming = s.accept();
```

一旦有人连接到这个端口，该方法就会返回一个表示已建立连接的`Socket`对象。可以使用这个对象得到输入和输出流：

```java
InputStream inStream = incoming.getInputStream();
OutputStream outStream = incoming.getOutputStream();
```

服务器发送给输出流的所有信息都会成为客户端的输入，同时来自客户端的所有输出都会出现在服务器输入流中。

在本章的所有示例中，我们都是通过套接字传输文本（而不是二进制数据）。因此将输入/输出流转换为scanner和writer。

```java
var in = new Scanner(inStream, StandardCharsets.UTF_8);
var out = new PrintWriter(new OutputStreamWriter(outStream, StandardCharsets.UTF_8), true);
```

在这个简单的服务器中，直接读取客户端输入，每次一行，然后回显(echo)（即原样写回，类似于Linux的`echo`命令）。

```java
String line = in.nextLine();
out.println("Echo: " + line);
if (line.strip().equals("BYE")) done = true;
```

在最后关闭客户端套接字：

```java
incoming.close();
```

这就是服务器处理单次请求的过程。每个服务器程序（例如HTTP Web服务器）都会执行下面这个循环：
1. 通过输入流从客户端接收命令。
2. 解码客户端命令。
3. 收集客户端请求的信息。
4. 通过输出流将信息发送给客户端。

程序清单4-3是程序的完整代码。

[程序清单4-3 server/EchoServer.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/server/EchoServer.java)

编译并运行这个程序，然后（在一个新的终端窗口）使用telnet连接到localhost的8189端口，将看到如下图所示的信息。输入任意内容并按回车键，然后可以在屏幕上看到回显。输入 "BYE" 可以断开连接，同时服务器程序也会终止。

![访问echo服务器](/assets/images/java-note-v2ch04-networking/访问echo服务器.png)

如果你直接连接到互联网（注：这意味着不能使用192.168开头的私有IP地址），那么世界上的任何人都可以访问你的echo服务器，只要他们知道你的IP地址和端口号。

### 4.2.2 服务多个客户端
前面例子中的简单服务器存在一个问题：同一时刻只有一个客户端能连接到服务器。我们希望允许多个客户端同时连接到服务器。可以使用线程来解决这个问题。

每当建立一个新的套接字连接，就启动一个新的线程来处理服务器和该客户端之间的连接，而主线程立即返回并等待下一个连接。为了实现这一点，服务器的主循环应该如下所示：

```java
while (true) {
    Socket incoming = s.accept();
    var r = new ThreadedEchoHandler(incoming);
    var t = new Thread(r);
    t.start();
}
```

`ThreadedEchoHandler`类实现了`Runnable`接口，其`run()`方法包含与客户端的通信循环：

```java
class ThreadedEchoHandler implements Runnable {
    ...
    public void run() {
        try (InputStream inStream = incoming.getInputStream();
             OutputStream outStream = incoming.getOutputStream()) {
            // Process input and send response
        }
        catch(IOException e) {
            // Handle exception
        }
    }
}
```

这样多个客户端就可以同时连接到服务器了。可以很容易地测试：
1. 编译并运行服务器程序（程序清单4-4）。
2. 打开几个telnet窗口（如下图所示）。
3. 在这些窗口之间切换并输入命令。注意你可以同时通过这些窗口进行通信。
4. 完成之后，切换到启动服务器程序的窗口，并按Ctrl+C将其终止。

![多个同时通信的telnet窗口](/assets/images/java-note-v2ch04-networking/多个同时通信的telnet窗口.png)

注释：在这个程序中，我们为每个连接创建一个单独的线程。这种方法对于高性能服务器来说并不令人满意。可以使用`java.nio`包中的特性实现更高的服务器吞吐量。详见[Merlin brings nonblocking I/O to the Java platform](https://web.archive.org/web/20170407234331/https://www.ibm.com/developerworks/java/library/j-javaio/)。

### 4.2.3 半关闭
**半关闭**(half-close)允许套接字连接的一端终止输出，同时仍然从另一端接收数据。

例如，假设向服务器传输数据，但是一开始并不知道有多少数据。如果在到达数据末尾时关闭套接字，则会立即与服务器断开连接，无法读取响应。半关闭可以解决这个问题。可以通过关闭套接字的输出流来告诉服务器请求数据已经结束，但保持输入流处于打开状态。

客户端代码如下：

```java
try (var socket = new Socket(host, port)) {
    var in = new Scanner(socket.getInputStream(), StandardCharsets.UTF_8);
    var writer = new PrintWriter(socket.getOutputStream());
    // send request data
    writer.print(...);
    writer.flush();
    socket.shutdownOutput();
    // now socket is half-closed
    // read response data
    while (in.hasNextLine()) {
        String line = in.nextLine();
        ...
    }
}
```

服务端只需读取输入，直到输入流结尾，然后发送响应即可。

该协议只适用于一次性的(one-shot)服务，例如HTTP（客户端连接服务器、发送请求、接收响应，然后断开连接）。（注：如果客户端需要进行多次请求-响应循环，则不适合使用半关闭）

### 4.2.4 可中断套接字
当连接到套接字或通过套接字读取数据时，当前线程会阻塞直到操作成功或超时（写数据没有超时时间）。

在交互式应用（如GUI）中，你可能希望允许用户取消无响应的套接字连接。但是，如果线程在无响应的套接字上阻塞，无法通过调用`interrupt()`来解除阻塞。

为了中断套接字操作，可以使用`java.nio`包的特性——`SocketChannel`。可以如下打开套接字通道：

```java
SocketChannel channel = SocketChannel.open(new InetSocketAddress(host, port));
```

与套接字不同，通道并没有关联的输入/输出流，而是具有使用`Buffer`对象的`read()`和`write()`方法（参见2.5.2节）。这些方法声明在`ReadableByteChannel`和`WritableByteChannel`接口中。

如果不想处理缓冲区，可以使用`Scanner`类读取`SocketChannel`，因为`Scanner`有一个带`ReadableByteChannel`参数的构造器：

```java
var in = new Scanner(channel, StandardCharsets.UTF_8);
```

为了将通道转换成输出流，可以使用静态方法`Channels.newOutputStream()`：

```java
OutputStream outStream = Channels.newOutputStream(channel);
```

当线程在打开、读取或写入操作期间被中断时，操作不会阻塞，而是以抛出异常而终止。

程序清单4-5对比了可中断套接字和阻塞套接字。服务器会发送数字，并假装在第十个数字之后卡住。点击Interruptible和Blocking按钮将启动一个线程，分别使用可中断套接字和阻塞套接字连接到服务器并打印输出。如果在第十个数字之前点击Cancel按钮，这两个线程都会中断。但是，在第十个数字之后，只能中断第一个线程。第二个线程会一直阻塞，直到服务器最终关闭连接。

[程序清单4-5 interruptible/InterruptibleSocketFrame.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/interruptible/InterruptibleSocketFrame.java)

![可中断套接字](/assets/images/java-note-v2ch04-networking/可中断套接字.png)

![阻塞套接字](/assets/images/java-note-v2ch04-networking/阻塞套接字.png)

## 4.3 获取Web数据
在下面几节中，将讨论Java库中用于访问Web服务器的类。

### 4.3.1 URL和URI
`URL`和`URLConnection`类封装了从远程站点获取信息的大部分复杂性。可以从字符串构造一个`URL`对象：

```java
var url = new URL("https://horstmann.com/corejava/index.html");
```

为了获取该资源的内容，可以使用`URL`类的`openStream()`方法。该方法返回一个`InputStream`对象，可以以通常的方式使用它，例如构造一个`Scanner`：

```java
InputStream inStream = url.openStream();
var in = new Scanner(inStream, StandardCharsets.UTF_8);
```

`java.net`包对**统一资源定位符**(uniform resource locator, URL)和**统一资源标识符**(uniform resource identifier, URI)作了有用了区分：
* URI用于**标识**(identify)资源，不一定是可访问的地址，包括URL和URN。例如`mailto:cay@horstmann.com`，这种不可访问的URI称为**统一资源名称**(uniform resource name, URN)。
* URL用于**定位**(locate)资源，必须是可访问的，是URI的子集。例如`https://horstmann.com/corejava/index.html`。

在Java库中，`URI`类唯一的作用是解析。相反，`URL`类可以打开连接到资源的流。

URI具有以下语法（详见[RFC 2396](https://www.ietf.org/rfc/rfc2396.txt)）：

```
[scheme:]schemeSpecificPart[#fragment]
```

其中，`[]`表示可选的部分。

包含`scheme:`部分的URI称为**绝对的**，否则称为**相对的**。

如果绝对URI的`schemeSpecificPart`不以`/`开头，则称为**不透明的**(opaque)。例如：

```
mailto:java-net@www.example.com
news:comp.lang.java
urn:isbn:096139210x
```

所有绝对透明URI和所有相对URI都是**分层的**(hierarchical)。例如：

```
http://example.com/languages/java/
sample/a/index.html#28
../../demo/b/index.html
file:///~/calendar
```

分层URI的`schemeSpecificPart`具有以下结构：

```
[//authority][path][?query]
```

对于基于服务器的URI，`authority`部分具有以下形式：

```
[user-info@]host[:port]
```

其中`port`必须是一个整数。

`URI`类的作用之一是解析标识符并将其分解成各个组成部分。可以使用`getScheme()`、`getSchemeSpecificPart()`等方法获取它们。例如：

```java
String urlString = "https://www.example.com:80/java/index.html?lang=en#section2";
URI uri = new URI(urlString);
String scheme = uri.getScheme();               // "https"
String specific = uri.getSchemeSpecificPart(); // "//www.example.com:80/java/index.html?lang=en"
String authority = uri.getAuthority();         // "www.example.com:80"
String userInfo = uri.getUserInfo();           // null
String host = uri.getHost();                   // "www.example.com"
int port = uri.getPort();                      // 80
String path = uri.getPath();                   // "/java/index.html"
String query = uri.getQuery();                 // "lang=en"
String fragment = uri.getFragment();           // "section2"
```

`URI`类的另一个作用是处理绝对和相对标识符。可以将一个绝对URI和一个相对URI组合成一个绝对URI，这个过程称为**解析**(resolving)。例如：

```java
var u = new URI("http://docs.mycompany.com/api/java/net/ServerSocket.html");
var r = new URI("../../java/net/Socket.html#Socket()");
var v = u.resolve(r);
  // http://docs.mycompany.com/api/java/net/Socket.html#Socket()
```

与此相反的过程称为**相对化**(relativization)。例如：

```java
var u = new URI("http://docs.mycompany.com/api");
var v = new URI("http://docs.mycompany.com/api/java/lang/String.html");
var r = u.relativize(v);
  // java/lang/String.html
```

注：
* URI的解析和相对化类似于`Path`类，见2.4.1节。
* 在上面的示例中，`u.resolve(r)`的结果是`http://docs.mycompany.com/java/lang/String.html`，而不是等于`v`。如果在`u`的结尾添加一个 "/" ，则`u.resolve(r)`等于`v`。

### 4.3.2 使用URLConnection获取信息
如果想获取Web资源的更多信息，应该使用`URLConnection`类，它比基本的`URL`类提供了更多的控制。

使用`URLConnection`包括以下步骤：
1. 调用`URL`类的`openConnection()`方法获得`URLConnection`对象。
2. 使用`setDoInput()`、`setDoOutput()`、`setRequestProperty()`等方法设置参数和请求属性。
3. 调用`connect()`方法连接远程资源。除了与服务器建立套接字连接外，该方法还会向服务器查询头信息(header information)。
4. 连接到服务器之后，可以使用`getHeaderFieldKey()`、`getHeaderField()`和`getHeaderFields()`方法查询头信息。
5. 最后，可以访问资源数据。使用`getInputStream()`方法获得用于读取信息的输入流（同`URL`类的`openStream()`方法返回的输入流）。

警告：`URLConnection`类的输入/输出流与`Socket`类并不完全相同。`URLConnection`类在底层做了许多工作，例如处理请求和响应消息头。因此，遵循上述步骤非常重要。

下面详细介绍一些`URLConnection`方法。有几个方法可以在连接到服务器之前设置连接属性，其中最重要的是`setDoInput()`和`setDoOutput()`。默认情况下，连接只生成用于从服务器读取数据的输入流，而没有用于写入的输出流。如果需要输出流（例如向Web服务器提交数据），则需要调用`connection.setDoOutput(true)`。

接下来，可能需要设置请求头(request header)。请求头是与请求命令一起发送到服务器的。例如，下面是一个HTTP请求，第一行包含方法(GET)、URI和版本号(1.0)，第2~9行是请求头。

```
GET www.server.com/index.html HTTP/1.0
Referer: http://www.somewhere.com/links.html
Proxy-Connection: Keep-Alive
User-Agent: Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.8.1.4)
Host: www.server.com
Accept: text/html, image/gif, image/jpeg, image/png, */*
Accept-Language: en
Accept-Charset: iso-8859-1,*,utf-8
Cookie: orangemilano=192218887821987
```

`setIfModifiedSince()`方法用于告诉连接你只对某个特定日期之后修改过的数据感兴趣。`setRequestProperty()`方法可以设置任何请求头键/值对。关于HTTP请求头的格式，参见[RFC 2616](https://www.ietf.org/rfc/rfc2616.txt) 5.3节。

如果想要访问一个有密码保护的网页，就必须执行以下操作：

```java
String input = username + ":" + password;
Base64.Encoder encoder = Base64.getEncoder();
String encoding = encoder.encodeToString(input.getBytes(StandardCharsets.UTF_8));
connection.setRequestProperty("Authorization", "Basic " + encoding);
```

提示：要通过FTP访问有密码保护的文件，需要使用一种完全不同的方法：构造具有以下格式的URL

```
ftp://username:password@ftp.yourserver.com/pub/file.txt
```

一旦调用了`connect()`方法，就可以查询响应头信息：

```java
String key = connection.getHeaderFieldKey(n);
String value = connection.getHeaderField(n);
```

分别获得第n个响应头的键和值，n从1开始！如果n为0或大于头字段总数则返回`null`（`getHeaderField(0)`除外）。没有返回头字段数量的方法。

`getHeaderFields()`方法返回一个包含所有响应头字段的映射：

```java
Map<String, List<String>> headerFields = connection.getHeaderFields();
```

下面是一组典型的HTTP响应头字段：

```
Date: Wed, 27 Aug 2008 00:15:48 GMT
Server: Apache/2.2.2 (Unix)
Last-Modified: Sun, 22 Jun 2008 20:53:38 GMT
Accept-Ranges: bytes
Content-Length: 4813
Connection: close
Content-Type: text/html
```

注释：可以通过`connection.getHeaderField(0)`或`headerFields.get(null)`获得响应状态行（例如 "HTTP/1.1 200 OK" ）。

为了简便起见，以下6个方法查询最常用的响应头。返回类型为`long`的方法返回自1970年1月1日GMT经过的秒数。

| 键名 | 方法名 | 返回类型 |
| --- | --- | --- |
| `Date` | `getDate` | `long` |
| `Expires` | `getExpiration` | `long` |
| `Last-Modified` | `getLastModified` | `long` |
| `Content-Length` | `getContentLength` | `int` |
| `Content-Type` | `getContentType` | `String` |
| `Content-Encoding` | `getContentEncoding` | `String` |

程序清单4-6中的程序可用于试验URL连接。在命令行中运行程序时，提供一个URL以及用户名和密码（可选）。程序将打印
* 响应头中的所有键和值
* 6个简便方法的返回值
* 所请求资源的前10行

[程序清单4-6 urlConnection/URLConnectionTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/urlConnection/URLConnectionTest.java)

### 4.3.3 提交表单数据
上一节介绍了如何从Web服务器读取数据。本节将介绍如何向Web服务器发送数据。

为了从浏览器将信息发送到Web服务器，用户需要填写类似下图所示的**表单**(form)。

![HTML表单](/assets/images/java-note-v2ch04-networking/HTML表单.png)

当用户点击提交按钮时，表单中的文本框、复选框、单选按钮和其他输入元素的内容将被发回Web服务器（即触发一次HTTP请求）。然后Web服务器调用处理用户输入的程序，并返回响应（如下图所示）。

![提交表单后的响应](/assets/images/java-note-v2ch04-networking/提交表单后的响应.png)

有许多技术可以让Web服务器调用程序。其中最著名的是Java Servlet、JavaServer Faces、微软的ASP (Active Server Pages)以及CGI (Common Gateway Interface)脚本。

服务器端程序处理表单数据，生成响应并将其发送回浏览器，然后浏览器展示响应页。这个过程如下图所示。

![执行服务器端程序的过程](/assets/images/java-note-v2ch04-networking/执行服务器端程序的过程.png)

本节只关注如何编写客户端程序并与已有的服务器端程序进行交互（即上图中的②和⑥）。

向Web服务器发送信息常用的**方法**(method)有两种：GET和POST。

GET方法直接将**查询参数**(query parameter)（对应表单字段）添加到URL结尾，例如：

```
https://www.google.com/maps?q=1+Market+Street+San+Francisco&hl=de
```

每个参数具有`name=value`的形式，参数之间用`&`分隔。参数值使用**URL编码**，遵循下面的规则：
* 字符A-Z、a-z、0-9和`. - ~ _`保持不变。
* 将空格替换为`+`。
* 将所有其他字符编码为UTF-8，每个字节表示为`%`后面跟着两位十六进制数字。

例如， "San Francisco, CA" 的URL编码结果为`San+Francisco%2c+CA`，十六进制数2c是字符 ',' 的UTF-8编码。

POST方法通常用于具有大量数据的表单。在POST请求中，不是将查询参数添加到URL，而是包含在消息体中（如下图所示）。仍然需要对值进行URL编码。

![HTTP POST请求](/assets/images/java-note-v2ch04-networking/HTTP POST请求.png)

注：
* 使用`URLConnection`类访问Web服务器时，请求行会自动生成，请求头通过`setRequestProperty()`方法设置，消息体通过输出流写入。`connect()`方法会完成请求消息的构造和发送。
* HTTP消息格式详见RFC 2616第4~6节和[HTTP messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages)。

向服务器端程序提交数据的详细过程如下：

```java
var url = new URL("http://host/path");
URLConnection connection = url.openConnection();
connection.setDoOutput(true);

// send data to the server
var out = new PrintWriter(connection.getOutputStream(), StandardCharsets.UTF_8);
out.print(name1 + "=" + URLEncoder.encode(value1, StandardCharsets.UTF_8) + "&");
out.print(name2 + "=" + URLEncoder.encode(value2, StandardCharsets.UTF_8));
out.close();

// read the server response
var inStream = connection.getInputStream();
```

下面来看一个实际的例子。网站 <https://tools.usps.com/zip-code-lookup.htm?byaddress> 包含一个用于查询街道地址的邮政编码的表单（见本节开头的图）。为了在Java程序中使用这个表单，需要知道请求方法、URL和参数。

可以通过查看表单的HTML代码得到这些信息，但是用网络监视器来“监视”发出的请求通常会更容易（注：实际上，这个表单的请求是在JavaScript中通过AJAX发送的，因此只能使用第二种方式）。大多数浏览器的开发者工具都包含网络监视器。例如，在Chrome浏览器中打开右上角菜单 → 更多工具 → 开发者工具（或者按快捷键Ctrl+Shift+I）。打开网络监视器后提交表单，可以发现提交的URL以及参数的名称和值，如下图所示。

![监视表单的提交](/assets/images/java-note-v2ch04-networking/监视表单的提交.png)

![监视表单的提交2](/assets/images/java-note-v2ch04-networking/监视表单的提交2.png)

在提交表单数据时，请求头包含内容类型和内容长度：

```
Content-Length: 124
Content-Type: application/x-www-form-urlencoded
```

程序清单4-7中的程序用于将POST表单数据发送给任何服务端程序。URL、请求头和参数放在一个.properties文件中，例如：

```properties
url=https://tools.usps.com/tools/app/ziplookup/zipByAddress
User-Agent=HTTPie/0.9.2
address1=1 Market Street
address2=
city=San Francisco
state=CA
companyName=
```

[程序清单4-7 post/PostTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/post/PostTest.java)

程序移除了.properties文件中的`url`和`User-Agent`项，并将其他内容传递给`doPost()`方法（作为查询参数）。

在`doPost()`方法中，首先打开连接并设置用户代理(user agent)（邮政编码服务对于包含字符串 "Java" 的默认用户代理无法工作，可能是因为屏蔽了程序化的请求）。然后调用`setDoOutput(true)`，打开输出流并写入查询参数。

当从写入请求切换到读取响应的任何部分时，就会发生与服务器的实际交互。在示例程序中，这种切换发生在调用`connection.getContentEncoding()`。

注：示例程序中并没有显式调用`connect()`，这是因为`getContentEncoding()`会自动调用该方法。调用过程如下：

```
PostTest.doPost()
  URLConnection.getContentEncoding()
    getHeaderField()
      getInputStream()
        getInputStream0()
          connect()
          writeRequests()  // 构造并发送请求消息
          sun.net.www.http.HttpClient.parseHTTP()  // 解析响应
          getResponseCode()
  HttpURLConnection.getInputStream()
```

在读取响应时，如果服务端出现错误，调用`connection.getInputStream()`就会抛出`IOException`。但是，服务器仍然会返回一个错误页面（例如随处可见的 "Error 404 - page not found" ）。为了捕获这个错误页面，可以调用`getErrorStream()`方法。

注释：`getErrorStream()`方法属于`URLConnection`的子类`HttpURLConnection`。如果请求的URL以`http://`或`https://`开头，就可以将连接对象强制转换为`HttpURLConnection`。

在请求Web服务器时，响应可能是**重定向**(redirect)：应该请求另一个URL来获得实际的信息。

注释：如果需要在重定向中包含cookie，可以像下面这样配置全局cookie处理器：

```java
CookieHandler.setDefault(new CookieManager(null, CookiePolicy.ACCEPT_ALL));
```

在大多数情况下，`HttpURLConnection`类可以自动处理重定向，但是有些情况下需要自己完成。因为安全原因不支持HTTP和HTTPS之间的自动重定向。另外在这个示例中，尽管可以在最初的请求中设置用户代理，自动重定向总是会发送包含字符串 "Java" 的用户代理。

在这种情况下，可以手动执行重定向。首先关闭自动重定向：

```java
connection.setInstanceFollowRedirects(false);
```

在发送请求之后，获取响应码：

```java
int responseCode = connection.getResponseCode();
```

如果响应码是301、302或303（对应`HttpURLConnection`类的常量`HTTP_MOVED_PERM`、`HTTP_MOVED_TEMP`和`HTTP_SEE_OTHER`），则从响应头`Location`获取重定向的URL。然后断开连接，并创建到新URL的连接：

```java
String location = connection.getHeaderField("Location");
if (location != null) {
    URL base = connection.getURL();
    connection.disconnect();
    connection = (HttpURLConnection) new URL(base, location).openConnection();
    ...
}
```

注：
* 目前即使设置了`User-Agent`，在Java程序中仍然无法正常访问邮政编码服务，而是返回一个 "This service is currently unavailable" 页面。可能是因为在表单中提交的请求会添加一些自动生成的请求头（例如`X-Jfuguzwb-A`、`X-Jfuguzwb-B`等）。
* 为了测试示例程序，可以使用[httpbin](https://httpbin.org/)工具。假设文件test.properties内容如下：

```properties
url=https://httpbin.org/post
User-Agent=HTTPie/0.9.2
name=Alice
email=alice@example.com
```

这个服务会以JSON格式将请求头和参数原样返回：

```shell
$ java post.PostTest post/test.properties
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "email": "alice@example.com", 
    "name": "Alice"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Content-Length": "36", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org",
    "User-Agent": "HTTPie/0.9.2",
    "X-Amzn-Trace-Id": "Root=1-6844583b-6e09136c30894ff256b77691"
  },
  "json": null,
  "origin": "221.218.159.190",
  "url": "https://httpbin.org/post"
}
```

## 4.4 HTTP客户端
`URLConnection`类提供了对多种协议的支持，但是对HTTP的支持有些笨重。

注：作为对比，下面是使用Python的[Requests](https://requests.readthedocs.io/en/latest/)库实现相同功能的代码。

```python
response = requests.post('https://httpbin.org/post',
    data={'name': 'Alice', 'email': 'alice@example.com'},
    headers={'User-Agent': 'HTTPie/0.9.2'})
```

`HttpClient`提供了更便捷的API和HTTP/2支持。在Java 9和10中，它位于`jdk.incubator.http`包。从Java 11起，它位于`java.net.http`包。

注释：在Java 9和10中，需要用命令行选项`--add-modules jdk.incubator.httpclient`来运行程序。

### 4.4.1 HttpClient类
`HttpClient`类可以发送HTTP请求并接收响应。可以通过以下调用获得一个客户端：

```java
HttpClient client = HttpClient.newHttpClient();
```

如果需要配置客户端，可以像下面这样使用建造者(builder) API：

```java
HttpClient client = HttpClient.newBuilder()
    .followRedirects(HttpClient.Redirect.ALWAYS)
    .build();
```

这是一种构建不可变对象的常见模式。

### 4.4.2 HttpRequest类和体发布器
定制请求也遵循建造者模式。下面是一个GET请求：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(new URI("http://horstmann.com"))
    .GET()
    .build();
```

注：也可以直接在`newBuilder()`方法中指定URI。

对于POST请求，需要一个“体发布器”(body publisher)（`HttpRequest.BodyPublisher`接口），用于将请求数据转换为要发送的数据。`HttpRequest.BodyPublishers`类提供了针对字符串、字节数组和文件的体发布器。例如，如果请求数据是JSON格式，只需将JSON字符串提供给字符串体发布器：

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(new URI(url))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(jsonString))
    .build();
```

（注：没有表单数据的体发布器，还是得手动拼接 "=" 和 "&" ……）

程序清单4-8中的示例程序提供了用于表单数据和文件上传的体发布器。

Java 16添加了一个`newBuilder()`方法，用于过滤已有请求的头（其他信息复制原请求）。需要提供该请求和一个函数，该函数接收头的名字和值，对于应该保留的头返回`true`。例如，下面修改了内容类型：

```java
HttpRequest request2 = HttpRequest.newBuilder(request,
    (name, value) -> !name.equalsIgnoreCase("Content-Type")) // Remove old content type
    .header("Content-Type", "application/xml") // Add new content type
    .build();
```

### 4.4.3 HttpResponse接口和体处理器
在发送请求时，需要告诉客户端如何处理响应：调用`HttpClient`类的`send()`方法并提供请求和体处理器（`HttpResponse.BodyHandler`接口，`HttpResponse.BodyHandlers`类提供了对应的静态工厂方法）。

如果只想将响应体作为字符串，可以使用`ofString()`：

```java
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

`HttpResponse`是一个泛型接口，其类型参数表示响应体的类型。可以直接获得响应体字符串：

```java
String bodyString = response.body();
```

还有其他的体处理器，可以将响应作为字节数组或输入流获取。`ofFile(filePath)`将响应保存到给定的文件；`ofFileDownload(directoryPath)`使用`Content-Disposition`头中的文件名将响应保存到给定的目录；`discarding()`直接丢弃响应。

处理响应的内容不在API的考虑范围内。例如，如果收到了JSON数据，就需要一个JSON库来解析其内容。

`HttpResponse`对象也可以获得状态码和响应头：

```java
int status = response.statusCode();
HttpHeaders responseHeaders = response.headers();
```

可以将`HttpHeaders`对象转换为映射：

```java
Map<String, List<String>> headerMap = responseHeaders.map();
```

映射的值是列表，因为在HTTP中消息头的每个键可以有多个值。

如果知道一个特定的键不会有多个值，可以调用`firstValue()`方法：

```java
Optional<String> lastModified = responseHeaders.firstValue("Last-Modified");
```

如果指定的键不存在则返回空的`Optional`。

### 4.4.4 异步处理
可以异步地处理响应。在构建客户端时提供一个执行器（参见卷I第12章 12.6.2节）：

```java
ExecutorService executor = Executors.newCachedThreadPool();
HttpClient client = HttpClient.newBuilder().executor(executor).build();
```

然后构建请求并调用客户端的`sendAsync()`方法。该方法返回一个`CompletableFuture<HttpResponse<T>>`，其中`T`是响应类型。之后就可以使用卷I第12章 12.7.1节介绍的`CompletableFuture` API：

```java
HttpRequest request = HttpRequest.newBuilder().uri(uri).GET().build();
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenAccept(response -> ...);
```

提示：要开启`HttpClient`的日志，将下面这行添加到JDK的net.properties文件（在$jdk/conf目录中）：

```properties
jdk.httpclient.HttpClient.log=all
```

除了`all`，还可以指定一个逗号分隔的列表，其中包含`headers`, `requests`, `content`, `errors`, `ssl`, `trace`和`frames`，后面可选地跟着`:control`, `:data`, `:window`或`:all`。不要使用空格。

然后，将名为`jdk.httpclient.HttpClient`的日志记录器的日志级别设置为`INFO`。例如，在JDK的logging.properties文件中添加下面这行（参见卷I第7章 7.5.3节）：

```properties
jdk.httpclient.HttpClient.level=INFO
```

[程序清单4-8 client/HttpClientTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/client/HttpClientTest.java)

## 4.5 发送E-mail
编写发送电子邮件的程序很简单：创建一个到邮件服务器端口25的套接字连接，然后发送邮件头，之后是邮件正文。简单邮件传输协议(Simple Mail Transport Protocol, SMTP)（参见[RFC 821](https://www.ietf.org/rfc/rfc821.txt)）描述了电子邮件消息格式。

详细过程如下：

1.打开一个到邮件服务器的套接字连接：

```java
var s = new Socket("mail.yourserver.com", 25); // 25 is SMTP
var out = new PrintWriter(s.getOutputStream(), StandardCharsets.UTF_8);
```

2.发送以下信息到输出流：

```
HELO sending host
MAIL FROM: sender e-mail address
RCPT TO: recipient e-mail address
DATA
Subject: subject
(blank line)
mail message (any number of lines)
.
QUIT
```

SMTP规范规定，每一行必须以`\r\n`结尾。

下面将展示如何使用[JavaMail](https://javaee.github.io/javamail/)在Java程序中发送电子邮件。

为了使用JavaMail，需要设置一些与邮件服务器相关的属性。例如，对于GMail，需要设置

```properties
mail.transport.protocol=smtps
mail.smtps.auth=true
mail.smtps.host=smtp.gmail.com
mail.smtps.user=accountname@gmail.com
```

我们的示例程序从属性文件读取这些值。出于安全原因，没有将密码放在属性文件中，而是提示用户输入。

首先读取属性文件，然后像这样获得一个邮件会话：

```java
Session mailSession = Session.getDefaultInstance(props);
```

创建消息，并指定发送人、接收人、主题和消息文本：

```java
var message = new MimeMessage(mailSession);
message.setFrom(new InternetAddress(from));
message.addRecipient(RecipientType.TO, new InternetAddress(to));
message.setSubject(subject);
message.setText(text);
```

然后发送消息：

```java
Transport tr = mailSession.getTransport();
tr.connect(null, password);
tr.sendMessage(message, message.getAllRecipients());
tr.close();
```

程序清单4-9中的程序从具有以下格式的文本文件中读取消息，并发送电子邮件。

```
Sender
Recipient
Subject
Message text (any number of lines)
```

要运行该程序，需要从 <https://javaee.github.io/javamail/> （或者[Maven](https://mvnrepository.com/artifact/com.sun.mail/javax.mail/1.6.2)）下载JavaMail实现。还需要从 <https://www.oracle.com/java/technologies/java-archive-downloads-java-plat-downloads.html#jaf-1.1.1-fcs-oth-JPR> （或者[Maven](https://mvnrepository.com/artifact/javax.activation/activation/1.1.1)）下载（JavaMail依赖的）Java激活框架的JAR文件。

然后运行

```shell
java -classpath .:javax.mail.jar:activation-1.1.1.jar path/to/mail.properties path/to/message.txt
```

提示：如果不明白邮件连接为什么无法正常工作，可以调用`mailSession.setDebug(true)`并检查消息。

[程序清单4-9 mail/MailTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch04/mail/MailTest.java)
