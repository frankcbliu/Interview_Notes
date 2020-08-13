# 2. Gee - 上下文设计（Context）

- 将`gee`中的`路由(router)`独立出来，方便后续设计。
- 增加`上下文(Context)`，封装 `Request` 和 `Response` ，提供对 `JSON`、`HTML` 等返回类型的支持。

## 用法展示

这里我们采用倒叙的方式，先看看第二天的代码写完后，在`main`函数中如何使用：

```go
// gee-web/main.go

func main() {
	r := gee.New()
	r.GET("/", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Hello Gee!</h1>")
	})
	r.GET("/hello", func(c *gee.Context) {
		// expect /hello?name=geek
		c.String(http.StatusOK, "hello %s, you are at %s\n", c.Query("name"), c.Path)
	})
	r.POST("/login", func(c *gee.Context) {
		c.JSON(http.StatusOK, gee.H{
			"username": c.PostForm("username"),
			"password": c.PostForm("password"),
		})
	})
	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

- 如下所示，我们看到`Get`方法的参数`HandlerFunc`的参数变成成了`*gee.Context`，提供了查询`Query/PostForm`参数的功能。

```go
// 添加 Get 请求
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}
```

- `gee.Context`封装了`HTML/String/JSON`函数，能够快速构造HTTP响应。

## Context 设计

### 必要性

对 Web 服务来说，无非是根据请求`*http.Request`，构造响应`http.ResponseWriter`。但是这两个对象提供的接口粒度太细，如果我们要构造一个完整的`请求-响应`，需要考虑消息头(Header)和消息体(Body)，而 Header 包含了状态码(StatusCode)，消息类型(ContentType) 等几乎**每次请求都需要设置的信息**。因此，如果不能进行有效的封装，那么我们在使用框架时就需要写大量重复、繁杂的代码，而且容易出错。作为一个好的框架，我们需要针对常用场景能够快速地构造出 HTTP 响应。

这里我们用返回 JSON 数据作比较，感受一下封装前后的差别。

**封装前**

```go
obj = map[string]interface{}{
    "name": "geek",
    "password": "123456",
}
w.Header().Set("Content-Type", "application/json")
w.WriteHeader(http.StatusOK)
encoder := json.NewEncoder(w)
if err := encoder.Encode(obj); err != nil {
  panic(err)
}
```

**封装后**：

```go
c.JSON(http.StatusOK, gee.H{
    "username": c.PostForm("username"),
    "password": c.PostForm("password"),
})
```

显然，封装后的代码简洁、清晰了许多。

除了封装`*http.Request`和`http.ResponseWriter`的方法，简化相关接口的调用，我们设计 Context 还有其他原因：

- 例如，将来解析动态路由`/hello/:name`，参数`:name`的值放在哪呢？
- 再比如，框架需要支持中间件，那中间件产生的信息放在哪呢？

事实上，`Context` 是贯穿请求的生命周期的，当一个请求从出现到结束，`Context` 也相应地从产生到销毁。和当前请求强相关的信息都应该存储在 `Context` 上。因此，我们设计 `Context` 时，将扩展性和复杂性留在了内部，对外则简化接口。路由的处理函数、将要实现的中间件，参数都统一使用 `Context` 实例， `Context` 就像一次会话的百宝箱，可以找到任何东西。

### 具体实现

```go
// gee-web/gee/context.go

type H map[string]interface{}

type Context struct {
	// origin objects
	Writer  http.ResponseWriter
	Request *http.Request
	// 请求信息
	Method string
	Path   string
	// 返回信息
	StatusCode int
}

func NewContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer:  w,
		Request: req,
		Method:  req.Method,
		Path:    req.URL.Path,
	}
}

// 获取 Form 参数
func (c *Context) PostForm(key string) string {
	return c.Request.FormValue(key)
}

// 获取 Query 参数
func (c *Context) Query(key string) string {
	return c.Request.URL.Query().Get(key)
}

// 设置状态码
func (c *Context) Status(code int) {
	c.StatusCode = code
	c.Writer.WriteHeader(code)
}

// 设置请求头
func (c *Context) SetHeader(key string, value string) {
	c.Writer.Header().Set(key, value)
}

// 返回 format 字符串
func (c *Context) String(code int, format string, values ...interface{}) {
	c.SetHeader("Content-Type", "text/plain")
	c.Status(code)
	_, err := c.Writer.Write([]byte(fmt.Sprintf(format, values...)))
	if err != nil {
		panic(err)
	}
}

// 返回 JSON
func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		panic(err)
	}
}

// 返回 Data
func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	_, err := c.Writer.Write(data)
	if err != nil {
		panic(err)
	}
}

// 返回 HTML
func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	_, err := c.Writer.Write([]byte(html))
	if err != nil {
		panic(err)
	}
}
```

- 代码最开头，给`map[string]interface{}`起了一个别名`gee.H`，便于简洁地构建JSON数据。
- `Context`目前只包含了`http.ResponseWriter`和`*http.Request`，以及提供对 `Method` 和 `Path` 这两个常用属性的直接访问。
- 提供了访问`Query`和`PostForm`参数的方法。
- 提供了快速构造`String/Data/JSON/HTML`响应的方法。

## 路由(Router)

我们将和路由相关的方法和结构提取出来，放到了一个`router.go`中，方便我们下一次对 router 的功能进行升级，例如提供动态路由的支持。 router 的 handle 方法作了一个细微的调整，即 handler 的参数，变成了 Context。

```go
// gee-web/gee/router.go

type router struct {
	// 路由表
	handlers map[string]HandlerFunc
}

func NewRouter() *router {
	return &router{handlers: make(map[string]HandlerFunc)}
}

// 往路由表中添加路由
func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	log.Printf("Route %4s - %s", method, pattern)
	key := method + "-" + pattern
	r.handlers[key] = handler
}

// 处理路由函数
func (r *router) handle(c *Context) {
	key := c.Method + "-" + c.Path
	if handler, ok := r.handlers[key]; ok {
		handler(c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
	}
}
```

## 框架入口

```go
// gee-web/gee/gee.go

// 此处参数类型改为了 *Context
type HandlerFunc func(c *Context)

type Engine struct {
	router *router
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: NewRouter()}
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	engine.router.addRoute(method, pattern, handler)
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
	c := NewContext(w, req)
	engine.router.handle(c)
}
```

将`router`相关的代码独立后，`gee.go`简洁了不少。当然，最重要的还是通过实现 `ServeHTTP` 接口，接管所有的 HTTP 请求。相比第一天的代码，这个方法也有细微的调整，在调用 `router.handle` 之前，先构造一个 `Context` 对象。这个对象目前还非常简单，仅仅包装了两个参数，之后我们会慢慢地给 Context 插上翅膀。

如何使用，`main.go`一开始就已经亮相了。运行`go run main.go`，借助 curl ，一起看一看今天的成果吧。

```bash
$ curl -i 127.0.0.1:8080
HTTP/1.1 200 OK
Content-Type: text/html
Date: Wed, 12 Aug 2020 09:47:36 GMT
Content-Length: 20

<h1>Hello Gee!</h1>

$ curl "http://localhost:8080/hello?name=geek"
hello geek, you are at /hello

$ curl "http://localhost:8080/login" -X POST -d 'username=geek&password=123456'
{"password":"123456","username":"geek"}

$ curl "http://localhost:8080/xxx"
404 NOT FOUND: /xxx
```

> 注意到代码中的 `package/import` 均被省略，读者需自行补充，使用 GoLand 的话 IDE 会自动补上。


<div class="jump">
	<a href="#/./project/gee-1">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-3">Next</a>
</div>

---


> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day2.html
