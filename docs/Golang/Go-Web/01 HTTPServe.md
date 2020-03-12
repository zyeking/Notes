
# 01：建立HTTP服务器的多种方法

## 建立服务器
```go
package mian

import (
    "log"
    "net/http"
)

func main() {
    http.HadnleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("hello world")
    }
    log.Println("starting HTTP server..")
    log.Fatal(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/11/4pTASLnM5DOZceq.jpg)

1. `http.HandleFunc(pattern string, handler func(ResponseWriter, *Request)` 根据一个==路由规则==绑定一个执行函数, 当用户访问到指定路由时执行.
2. `http.HandleFunc`的第二个参数必须符合函数签名`   func(http.ResponseWriter, *http.Request)`, 第一个参数是请求所对应的响应对象`http.ResponseWriter`,包含响应码、响应头和响应体, 在这里通过调用`响应对象`的`Write`方法像响应体写入字符串. 第二个参数是请求所对应的请求对象`*http.Request`,包含请求头、请求体等.
3. `http.ListenAndServe`启动HTTP服务器,监听`指定地址`和`端口号`的HTTP请求
4. `http.HandleFunc`将传入的`绑定函数`转换为类型`http.HandleFunc`(一个HTTP请求处理器对象),该对象类型实现`http.Handler`接口,接口方法调用自己
```golang
// net/http/server.go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers. If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler that calls f.
type HandlerFunc func(ResponseWriter, *Request)
 
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
![流程图](https://i.loli.net/2019/12/20/Ny9p6MBJrWzL7Hd.png)

## 自定义Handler
```go
package main
import (
    "log"
    "net/http"
)

type customHandler {}

// 将ServeHTTP方法绑定到customHandler上
func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request){
    w.Write([]byte("hello custom Handler")
}

func main() {
    http.Handle("/", &cutsomerHandler{})
    log.Println("starting HTTP server...")
    log.Fatl(http.ListenAndServe(":8000", nil))
}
```
![](https://i.loli.net/2019/11/12/goP7jr2Rxik1wTs.jpg)

1. 自定义类型`type customHandler{}`
2. 类型绑定`ServerHTTP(w http.ResponseWriter, r *http.Request)`方法
3. `http.Handle`调用, 通过`&customHandler{}`传入该自定义类型的地址

![流程图](https://i.loli.net/2019/12/20/St4qUOoM2Q3RrPa.png)
> 少了将绑定函数转换为`type HandleFunc`的步骤

## 服务复用器(ServeMux)
```go
// ListenAndServe listens on the TCP network address addr and then calls
// Serve with handler to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
//
// The handler is typically nil, in which case the DefaultServeMux is used.
//
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
//*****************************//
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
1. 实例中`http.ListenAndServe`的nil替代了实现了`http.Handler`接口的对象
2. `http.Handler`实现`ServeHTTP`接口
3. ==缺陷== 该方法不能像之前调用的`http.HandleFunc`和`http.Handle`为不同路由规则绑定不同的函数处理
```go
package main

import (
	"log"
	"net/http"
)

type customHandler struct{}

func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello custom server mux's ServeHTTP"))
}
func main() {
	log.Println("starting HTTP server... ")
	log.Fatal(http.ListenAndServe(":8000", &customHandler{}))
}
```
![](https://i.loli.net/2019/11/12/PAH98iMlEQqIokG.jpg)

### 自定义Serve Mux
```go
// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
//*****************************//
// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = &defaultServeMux

var defaultServeMux ServeMux
//*****************************//
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}

```

1. `handle.Handle`调用`DefaultServeMux`,该`DefaultMux`是`http.ServeMux`的封装
2. `http.ServeMux`带有基本路由功能的服务复用器(Serve Multiplexer)
3. 通过`http.NewServeMux`操作`http.ServeMux`对象, 调用`http.NewServeMux`的`.Handle`方法
```go
package main

import (
	"log"
	"net/http"
)

type customHandler struct{}

func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello new custom server mux"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &customHandler{})
	log.Println("starting HTTP serve...")
	log.Fatal(http.ListenAndServe(":8000", mux))
}
```
![](https://i.loli.net/2019/11/12/ZdqBuKPvrohxOGg.jpg) 

## 服务器对象(Server)
```go
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
} 
//***************************//
type Server struct {
    Addr string
    Handler Handler
 ...
}
```
### 自定义server1
1. 调用`http.ListenAndServe`的时候创建了另一个`http.Serve`对象
```go
package main

import (
	"log"
	"net/http"
)

type customHandler struct{}

func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello custom server mux"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &customHandler{})

	serve := &http.Server{
		Addr:    ":8000",
		Handler: mux,
	}
	log.Println("starting HTTP serve...")
	log.Fatal(serve.ListenAndServe())
}
```
![](https://i.loli.net/2019/11/12/PlSst4cpn8RxgUZ.jpg)
### 自定义server2

```go
// custome server 2
package main

import (
	"log"
	"net/http"
	"time"
)

type customHandler struct{}

func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello custom server mux2"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &customHandler{})
	mux.HandleFunc("/timeout", func(w http.ResponseWriter, r *http.Request) {
	    // 超时
		time.Sleep(2 * time.Second)
		w.Write([]byte("Timeout"))
	})

	server := &http.Server{
		Addr:         ":8000",
		Handler:      mux,
		WriteTimeout: 2 * time.Second,
	}
	log.Println("starting HTTP server...")
	log.Fatal(server.ListenAndServe())
}
```
1. 无法访问到`localhost:8000/timeout`
2. 执行函数休眠2秒, 被`http.Serve`对象认为已经超时,提前关闭与客户端之间的连接, 后面无法像响应体写入任何信息

![](https://i.loli.net/2019/11/12/iV1bDI2KU3RMQdz.jpg)
![](https://i.loli.net/2019/11/12/K9H3UB2DZ8xVSIT.jpg)

## 优雅地停止服务
1. 通过捕捉系统信号(Signal)、goroutine和通道(Channel)实现
2. 捕捉`os.Interrupt`信号(ctrl+c)然后调用`server.Shutdown`方法g告知服务器停止接受新请求
3. `http.ErrServerClosed`根据该错误类型判断服务器是否正常关闭

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
)

type customHandler struct{}

func (_ *customHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello custom server mux2"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &customHandler{})

	server := &http.Server{
		Addr:    ":8000",
		Handler: mux,
	}

	// 创建系统信号接收器
	quit := make(chan os.Signal)
	signal.Notify(quit, os.Interrupt)
	go func() {
		<-quit

		if err := server.Shutdown(context.Background()); err != nil {
			log.Fatal("Shutdown server:", err)
		}
	}()

	log.Println("start HTTP server...")
	err := server.ListenAndServe()
	if err != nil {
		if err == http.ErrServerClosed {
			log.Print("Server Closed under request")
		} else {
			log.Fatal("Server closed unexpected")
		}
	}
}
```
![](https://i.loli.net/2019/11/12/3vZwu7Xce61xfyd.jpg)