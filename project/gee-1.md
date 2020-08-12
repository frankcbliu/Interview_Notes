# 1. Gee - 前置知识（http.Handler）

- 了解标准库`net/http`以及`http.Handler`接口。
- 搭建`Gee`的基本框架。

## 标准库启动 Web 服务

`net/http`是 Go 语言内置的`HTTP`网络编程基础库，我们所实现的`Gee-Web`框架就是基于这个库的，接下来我们用例子来了解这个库的使用方法。

```go
// gin-web/main.go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", indexHandler)
	http.HandleFunc("/hello", helloHandler)
	// 启动服务，监听 8080
	log.Fatal(http.ListenAndServe(":8080", nil))
}

// 输出 request 的 URL.Path
func indexHandler(w http.ResponseWriter, req *http.Request) {
	_, err := fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path) // 此处 %q 会在值左右加上双引号
  if err != nil {
    panic(err)
  }
}

// 输出 request 的 Header
func helloHandler(w http.ResponseWriter, req *http.Request) {
	for k, v := range req.Header {
		_, err := fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
    if err != nil {
      panic(err)
    }
	}
}
```

我们将路由`/`绑定`indexHandler`，将路由`/hello`绑定 `helloHandler`，当 HTTP 请求进来时，会根据不同的 路径调用不同的处理函数。

访问`/`：

```bash
$ curl http://localhost:8080/
URL.Path = "/"
```

访问`/hello`，得到请求头(header)中的键值对信息：
```bash
$ curl http://localhost:8080/hello
Header["User-Agent"] = ["curl/7.64.1"]
Header["Accept"] = ["*/*"]
```

除了`curl`，我们也可以直接在浏览器中打开 `http://127.0.0.1:8080/`。

回顾`main`函数，最后一行`log.Fatal(http.ListenAndServe(":8080", nil))`是用来启动 Web 服务的。

`http.ListenAndServe`方法的第一个参数是`host:port`，`:8080`表示在 `8080` 端口监听。

而第二个参数则代表处理所有的 HTTP 请求的实例，`nil` 代表使用标准库中的实例处理。后续我们将利用该参数实现对请求的统一处理。

## 实现 http.Handler 接口

通过查看`net/http`的源码可以发现，`Handler`是一个接口，需要实现方法 `ServeHTTP` ：

`net/http`中的`http/server.go`：

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

也就是说，只要传入任何实现了 `ServerHTTP` 接口的实例，所有的 HTTP 请求，都会交给该实例进行处理。

```go
// gin-web/main.go
package main

import (
	"fmt"
	"log"
	"net/http"
)

// 定义空结构体
type Engine struct{}

// 实现 net/http 中的 Handler 接口
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	switch req.URL.Path {
	case "/":
    _, err := fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path) // 此处 %q 会在值左右加上双引号
    if err != nil {
      panic(err)
    }
	case "/hello":
		for k, v := range req.Header {
			_, err := fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
      if err != nil {
        panic(err)
      }
		}
	default: // 处理默认情况，对于没有单独处理的路径，统一返回 404
		_, err := fmt.Fprintf(w, "404 NOT FOUND: %s\n", req.URL)
    if err != nil {
      panic(err)
    }
	}
}

func main() {
	handler := new(Engine)
	// 启动服务，监听 8080
	log.Fatal(http.ListenAndServe(":8080", handler))
}
```

- 我们定义了一个空的结构体`GeeHandler`，实现了方法`ServeHTTP`。这个方法有2个参数，第二个参数是 `Request` ，该对象包含了该 HTTP 请求的所有的信息，比如请求地址、Header 和 Body 等信息；第一个参数是 `ResponseWriter` ，利用 `ResponseWriter` 可以构造针对该请求的响应。
- 在 `main` 函数中，我们给 `ListenAndServe` 方法的第二个参数传入了刚才创建的`handler`实例。至此，我们走出了实现 Web 框架的第一步，即，将所有的 HTTP 请求转向了我们自己的处理逻辑。还记得吗，在实现`GeeHandler`之前，我们调用 `http.HandleFunc` 实现了路由和 Handler 的映射，也就是只能针对具体的路由写处理逻辑。比如`/hello`。但是在实现`GeeHandler`之后，我们拦截了所有的 HTTP 请求，拥有了统一的控制入口。在这里我们可以自由定义路由映射的规则，也可以统一添加一些处理逻辑，例如日志、异常处理等。
- 代码的运行结果与之前的是一致的。

## Gee-Web的基本框架

我们接下来重新组织上面的代码，搭建出整个框架的雏形。

最终的代码目录结构是这样的。

```
gee/
	|--go.mod
  |--gee.go
go.mod
main.go
```

首先按照上述目录结构，新增`gee`文件夹，然后创建`go.mod`：

```go
// gin-web/gee/go.mod
module gee

go 1.14
```

然后创建`gee.go`，具体文件内容稍后再讲。

### go.mod

这里在`main.go`同目录下创建`go.mod`：

```go
// gin-web/go.mod
module gee-web

go 1.14

require gee v0.0.0

replace gee => ./gee
```

- 在 `go.mod` 中使用 `replace` 将 gee 指向 `./gee`

> 从 go 1.11 版本开始，引用相对路径的 package 需要使用上述方式。

### main.go

```go
// gin-web/main.go
package main

import (
	"fmt"
	"net/http"

	"gee"
)

func main() {
	r := gee.New() // 在没有往 gee.go 写入内容时，这里会报错，可以先继续往下看，看完再回过来梳理。
	r.GET("/", func(w http.ResponseWriter, req *http.Request) {
    _, err := fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
    if err != nil {
      panic(err)
    }
	})
	r.GET("/hello", func(w http.ResponseWriter, req *http.Request) {
		for k, v := range req.Header {
			_, err := fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
      if err != nil {
        panic(err)
      }
		}
	})
	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

看到这里，如果你之前使用过`gin`的话（同样是基于 Go 实现的 Web 框架，应用非常广泛），肯定会觉得很亲切。因为我们的`gee`框架设计、API 设计均参考了`gin`。使用`New()`创建 gee 的实例，使用 `GET()`方法添加路由，最后使用`Run()`启动Web服务。当然，我们目前实现的路由只是静态路由，还不支持`/hello/:name`这样的动态路由，动态路由我们将在下一次实现。

### gee.go

```go
// gin-web/gee/gee.go
package gee

import (
	"fmt"
	"net/http"
)

// 定义一个处理函数
type HandlerFunc func(http.ResponseWriter, *http.Request)

type Engine struct {
	// 路由表
	router map[string]HandlerFunc
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: make(map[string]HandlerFunc)}
}

// 往路由表中添加路由
func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	key := method + "-" + pattern
	engine.router[key] = handler
}

// 添加 Get 请求
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

// 添加 POST 请求
func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

// 启动 HTTP 服务
func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.Method + "-" + req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
    w.WriteHeader(http.StatusNotFound)
    _, err := fmt.Fprintf(w, "404 NOT FOUND: %s\n", req.URL)
    panic(err)
	}
}
```

这里的`gee.go`是我们接下来的重头戏。我们重点讲一下这部分的实现。

- 首先定义类型`HandlerFunc`，这是提供给框架用户的，用来定义路由映射的处理方法。我们在`Engine`中，添加了一张路由映射表`router`，key 由请求方法和静态路由地址构成，例如`GET-/`、`GET-/hello`、`POST-/hello`，这样针对相同的路由，如果请求方法不同,可以映射不同的处理方法(Handler)，value 是用户映射的处理方法。
- 当用户调用`(*Engine).GET()`方法时，会将路由和处理方法注册到映射表 `router` 中，`(*Engine).Run()`方法，是 `ListenAndServe` 的包装。
- `Engine`实现的 `ServeHTTP` 方法的作用就是，解析请求的路径，查找路由映射表，如果查到，就执行注册的处理方法。如果查不到，就返回 `404 NOT FOUND` 。

执行`go run main.go`，再用 `curl` 工具访问，结果与最开始的一致。

```bash
$ curl http://localhost:8080/
URL.Path = "/"

$ curl http://localhost:8080/hello
Header["User-Agent"] = ["curl/7.64.1"]
Header["Accept"] = ["*/*"]

$ curl http://localhost:8080/world
404 NOT FOUND: /world
```

至此，整个`Gee`框架的原型已经出来了。尽管还没有实现比`net/http`标准库更强大的能力，但我们实现了路由映射表，提供给用户进行静态路由注册的方法，包装了启动服务的函数。不用担心，很快就可以将动态路由、中间件等功能添加上去了。


<div class="jump">
	<a href="#/./docs/go-web">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-2">Next</a>
</div>


----

> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day1.html

