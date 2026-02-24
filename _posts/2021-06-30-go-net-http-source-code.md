---
title: 【Go】net/http包源码解读
date: 2021-06-30 20:04:12 +0800
categories: [Go]
tags: [go, http]
---
## 1.Web服务器
使用`net/http`包编写一个最简单的Web服务器：

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", index)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, world!")
}
```

![运行截图](/assets/images/go-net-http-source-code/运行截图.png)

Web服务器一次请求的流程如下：

客户端→request→多路器(multiplexer)/路由器(router)→handler→response→客户端

这一过程的核心是路由，即使用多路器找到URL模式对应的处理函数，因此`net/http`包中最重要的概念就是多路器和handler，分别对应`ServeMux`结构体和`Handler`接口

`main()`函数的第1行调用`http.HandleFunc()`函数进行**路由注册**，第2行调用`http.ListenAndServe()`函数监听端口启动服务

`http.HandleFunc()`函数调用默认多路器的同名方法：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

`http.ListenAndServe()`函数创建了一个`Server`实例，之后调用该实例的同名方法：

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}

type Server struct {
	Addr string
	Handler Handler
	// ...
}
```

## 2.Handler接口
`http.ListenAndServe()`的第二个参数和`Server`的第二个字段都是`Handler`类型

`Handler`是一个接口，只有一个`ServeHTTP()`方法：

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

`Handler`接口描述的是“**给定请求能够返回响应**的对象”，即执行实际的业务逻辑，handler函数就是这样的对象

示例代码中的`index()`就是一个handler函数，该函数的参数类型与`ServeHTTP()`方法相同，但函数名不同

为了能够将handler函数包装为`Handler`接口值，`net/http`包定义了一个`HandlerFunc`类型：

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

`HandlerFunc`是一个函数类型，且实现了`Handler`接口，其`ServeHTTP()`方法就是调用函数本身，因此`HandlerFunc(index)`是一个`Handler`类型的值，其`ServeHTTP()`方法就是调用`index()`函数

`http.HandleFunc()`函数就利用了这种转换，`http.HandleFunc("/", index)`等价于`http.Handle("/", http.HandlerFunc(index))`

## 3.多路器ServeMux
`ServeMux`结构体定义如下：

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry
	hosts bool
}

type muxEntry struct {
	h       Handler
	pattern string
}
```

可以看到`ServeMux`本质上就是**URL模式到handler的映射**，存储在其`m`字段中

可以使用`http.NewServeMux()`函数创建一个新的`ServeMux`，也可以直接使用默认的`DefaultServeMux`，`http.HandleFunc()`就使用了`DefaultServeMux`

```go
func NewServeMux() *ServeMux { return new(ServeMux) }

var DefaultServeMux = &defaultServeMux
var defaultServeMux ServeMux
```

示例代码中执行完`main()`函数第1行后`http.DefaultServeMux`的值如下：

![DefaultServMux的值](/assets/images/go-net-http-source-code/DefaultServMux的值.png)

`ServeMux`的两大功能是路由注册和路由查找

### 3.1 路由注册
路由注册由`Handle()`和`HandleFunc()`方法实现：

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	// ...
	e := muxEntry{h: handler, pattern: pattern}
	mux.m[pattern] = e
	// ...
}

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	// ...
	mux.Handle(pattern, HandlerFunc(handler))
}
```

可以看到`Handle()`方法就是将指定的URL模式和handler写入映射，而`HandleFunc()`方法只是一个快捷操作

由于`http.Handle()`和`http.HandleFunc()`分别调用了`DefaultServeMux.Handle()`和`DefaultServeMux.HandleFunc()`，因此

```go
http.HandleFunc("/", index)
log.Fatal(http.ListenAndServe("localhost:8000", nil))
```

等价于

```go
mux := http.NewServeMux()
mux.HandleFunc("/", index)
log.Fatal(http.ListenAndServe("localhost:8000", mux))
```

`http.ListenAndServe()`的第二个参数是用于处理**所有**请求的handler，如果是`nil`则使用`DefaultServeMux`

### 3.2 路由查找
上面最后一行代码将`mux`作为`http.ListenAndServe()`的第二个参数，因为`ServeMux`也实现了`Handler`接口

```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	// ...
	h, _ := mux.Handler(r)
	h.ServeHTTP(w, r)
}
```

其中`mux.Handler(r)`可以理解为`mux.m[r.URL.Path]`，即在映射中查找request的URL对应的handler

上面提到，`Handler`接口描述的是“给定请求能够返回响应的对象”，`ServeMux`实现这一功能的方式就是**根据请求的URL在映射中找到handler并调用该handler**，这也是`ServeMux`的路由查找功能

## 4.处理客户端请求
注册好路由后还需要启动服务器监听端口

`http.ListenAndServe()`创建了一个`Server`实例并调用其同名方法

`(*Server).ListenAndServe()`调用`net.Listen()`监听端口，并使用其返回的listener调用自己的`Serve()`方法

```go
func (srv *Server) ListenAndServe() error {
	// ...
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```

`(*Server).Serve()`遵循套接字编程标准：主体是一个无限循环，每次循环调用`Listener.Accept()`接受客户端请求，返回一个连接对象，最后启动一个goroutine调用连接对象的`serve()`方法来处理客户端请求

```go
func (srv *Server) Serve(l net.Listener) error {
	// ...
	for {
		rw, err := l.Accept()
		// ...
		c := srv.newConn(rw)
		// ...
		go c.serve(connCtx)
	}
}
```

`(*conn).serve()`方法的代码如下：

```go
func (c *conn) serve(ctx context.Context) {
	// ...
	for {
		w, err := c.readRequest(ctx)
		// ...
		if err != nil {
			// return
		}

		// ...
		serverHandler{c.server}.ServeHTTP(w, w.req)
		// ...
	}
}
```

该方法的代码很长，但核心逻辑很简单，主体也是一个无限循环（每个连接能处理多次请求），每次循环获取一次客户端请求，并调用对应的handler处理请求

`readRequest()`的第一个返回值是`response`指针，该结构体实现了`ResponseWriter`接口，并且有一个`*Request`成员`req`，因此`w`和`w.req`可以充当handler的两个参数

如果`readRequest()`返回了一个错误（例如客户端断开连接）则`serve()`方法返回，本次连接结束

调用handler时创建了一个`serverHandler`实例，该类型定义如下：

```go
type serverHandler struct {
	srv *Server
}

func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	// ...
	handler.ServeHTTP(rw, req)
}
```

该结构体只有一个`Server`指针字段`srv`，并且也实现了`Handler`接口，其`ServeHTTP()`方法就是调用`srv.Handler.ServeHTTP()`，其中`srv.Handler`就是一开始调用`http.ListenAndServe()`时的第二个参数，是一个多路器，如果是`nil`则使用`DefaultServeMux`（不是很懂为什么要定义这个类型）

以上就是`net/http`包的核心代码解读

整体流程图如下：

![整体流程图](/assets/images/go-net-http-source-code/整体流程图.png)
