---
layout: post
title: Golang net/http 库
subtitle: Golang 每日一库
date: 2024-02-26 19:50:00 +0800
categories: Golang
author: 月梦
cover: 'https://s1.ax1x.com/2023/09/04/pPrSza4.jpg'
cover_author: 'Rutva Safi'
cover_author_link: 'https://www.softwebsolutions.com/author/rutva-safi'
tags: 
- Golang  
---

## 简介
`Golang` 的 `net/http` 库是构建 `web` 应用的标准库。  

## Example

首先从一个简单的例子开始吧，让我们启动一个简单的 `web` 服务器，对于请求返回 "hello,world!"。  

```go
package main

import "net/http"

func Hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello,world!"))
}

func main() {
	http.HandleFunc("/", Hello)
	http.ListenAndServe("localhost:8080", nil)
}
```

首先调用了 `http.HandleFunc` 函数，让我们来看看它的注释： 

>_HandleFunc registers the handler function for the given pattern in the DefaultServeMux. The documentation for ServeMux explains how patterns are matched._  

`HandleFunc` 在 `DefaultServeMux` 中为给定的模式注册 `handler function` ，这是它的实现。  

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

可以看到，它的前一个参数接收一个字符串，就是上面提到的模式，后一个参数接收一个 `handler function` ，是对应模式的处理函数。  

函数调用了 `DefaultServeMux` 的 `HandleFunc` 函数，我们看看 `DefaultServeMux` 是什么

`DefaultServeMux` 是类 `ServeMux` 的一个默认实例

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}

type muxEntry struct {
	h       Handler
	pattern string
}
```

>_ServeMux is an HTTP request multiplexer._  

它通过注册模式的列表匹配每一个请求的 `URL` ，并且调用对应的 `handler`   

它的 `HandleFunc` 方法定义如下：  

```go
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
	mux.Handle(pattern, HandlerFunc(handler))
}
```

其中首先判断传入的 `handler` 是否为空，随后调用自己的 `Handle` 方法，用于为指定的模式注册 `handler`    

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
	e := muxEntry{h: handler, pattern: pattern}
	mux.m[pattern] = e
	if pattern[len(pattern)-1] == '/' {
		mux.es = appendSorted(mux.es, e)
	}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}
```

在 `Handle` 方法中，首先判模式空，之后判 `handler` 空，之后判断模式对应的 `handler` 是否已经存在，若存在则 `panic` 

从这里可以看到 `ServeMux` 中的 `m` 存储的是 模式->`muxEntry` 键值对  

随后组装一个 `muxEntry` ，再添加到 `m` 中，这里按照模式串长度从大到小维护 `es`   

至此，`handler` 就注册好了，接下来启动 `http` 服务器  

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

`ListenAndServe` 函数首先实例化一个 `Server` 类，之后调用它的 `ListenAndServe` 函数  

```go
func (srv *Server) ListenAndServe() error {
	if srv.shuttingDown() {
		return ErrServerClosed
	}
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```

这里首先取出地址，也就是 `addr` ，随后基于 `TCP` 协议监听这个地址，最后再调用 `Server` 的 `Serve` 函数  

```go
func (srv *Server) Serve(l net.Listener) error {
	if fn := testHookServerServe; fn != nil {
		fn(srv, l) // call hook with unwrapped listener
	}

	origListener := l
	l = &onceCloseListener{Listener: l}
	defer l.Close()

	if err := srv.setupHTTP2_Serve(); err != nil {
		return err
	}

	if !srv.trackListener(&l, true) {
		return ErrServerClosed
	}
	defer srv.trackListener(&l, false)

	baseCtx := context.Background()
	if srv.BaseContext != nil {
		baseCtx = srv.BaseContext(origListener)
		if baseCtx == nil {
			panic("BaseContext returned a nil context")
		}
	}

	var tempDelay time.Duration // how long to sleep on accept failure

	ctx := context.WithValue(baseCtx, ServerContextKey, srv)
	for {
		rw, err := l.Accept()
		if err != nil {
			if srv.shuttingDown() {
				return ErrServerClosed
			}
			if ne, ok := err.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return err
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew, runHooks) // before Serve can return
		go c.serve(connCtx)
	}
}
```

>_Serve accepts incoming connections on the Listener l, creating a new service goroutine for each. The service goroutines read requests and then call srv.Handler to reply to them._  

`Serve` 为每个已建立的连接创建一个新的服务协程，服务协程读取请求并且调用 `handler` 来处理并回应

大体上来说，这里的操作有创建 `Context`、Accept 新的连接、基于 `net.Conn` 创建封装的连接对象 `conn` 以及调用 `conn` 的 `serve` 方法  

```go
func (c *conn) serve(ctx context.Context) {
	for {
        w, err := c.readRequest(ctx)
		···
		req := w.req
		···
		serverHandler{c.server}.ServeHTTP(w, w.req)
    }
}
```

`serve` 函数源码较为复杂，不过大体可以分为上述几个步骤，由于——  

>_HTTP cannot have multiple simultaneous active requests.[*]
Until the server replies to this request, it can't read another,
so we might as well run the handler in this goroutine._

所以在这个协程中完成了读取请求、调用 `handle` 并响应三步  

