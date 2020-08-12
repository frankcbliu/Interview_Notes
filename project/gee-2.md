# 2. Gee - 上下文设计（Context）

- 将`路由(router)`独立出来，方便之后增强。
- 设计`上下文(Context)`，封装 Request 和 Response ，提供对 JSON、HTML 等返回类型的支持。
- 动手写 Gee 框架的第二天，**框架代码140行，新增代码约90行**

## 使用效果

为了展示第二天的成果，我们看一看在使用时的效果。

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

- `Handler`的参数变成成了`gee.Context`，提供了查询`Query/PostForm`参数的功能。
- `gee.Context`封装了`HTML/String/JSON`函数，能够快速构造HTTP响应。

## 设计Context

### 必要性

对 Web 服务来说，无非是根据请求`*http.Request`，构造响应`http.ResponseWriter`。但是这两个对象提供的接口粒度太细，比如我们要构造一个完整的响应，需要考虑消息头(Header)和消息体(Body)，而 Header 包含了状态码(StatusCode)，消息类型(ContentType) 等几乎每次请求都需要设置的信息。因此，如果不进行有效的封装，那么框架的用户将需要写大量重复，繁杂的代码，而且容易出错。针对常用场景，能够高效地构造出 HTTP 响应是一个好的框架必须考虑的点。

用返回 JSON 数据作比较，感受下封装前后的差距。

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
    http.Error(w, err.Error(), 500)
}
```

**封装后**：

```go
c.JSON(http.StatusOK, gee.H{
    "username": c.PostForm("username"),
    "password": c.PostForm("password"),
})
```

1. 针对使用场景，封装`*http.Request`和`http.ResponseWriter`的方法，简化相关接口的调用，只是设计 Context 的原因之一。对于框架来说，还需要支撑额外的功能。例如，将来解析动态路由`/hello/:name`，参数`:name`的值放在哪呢？再比如，框架需要支持中间件，那中间件产生的信息放在哪呢？Context 随着每一个请求的出现而产生，请求的结束而销毁，和当前请求强相关的信息都应由 Context 承载。因此，设计 Context 结构，扩展性和复杂性留在了内部，而对外简化了接口。路由的处理函数，以及将要实现的中间件，参数都统一使用 Context 实例， Context 就像一次会话的百宝箱，可以找到任何东西。

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
	c.Writer.Write([]byte(fmt.Sprintf(format, values...)))
}

// 返回 JSON
func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}

// 返回 Data
func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	c.Writer.Write(data)
}

// 返回 HTML
func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}
```

- 代码最开头，给`map[string]interface{}`起了一个别名`gee.H`，构建JSON数据时，显得更简洁。
- `Context`目前只包含了`http.ResponseWriter`和`*http.Request`，另外提供了对 `Method` 和 `Path` 这两个常用属性的直接访问。
- 提供了访问`Query`和`PostForm`参数的方法。
- 提供了快速构造`String/Data/JSON/HTML`响应的方法。

## 路由(Router)

我们将和路由相关的方法和结构提取了出来，放到了一个新的文件中`router.go`，方便我们下一次对 router 的功能进行增强，例如提供动态路由的支持。 router 的 handle 方法作了一个细微的调整，即 handler 的参数，变成了 Context。

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
}å
```

## 框架入口

```go
// gee-web/gee/gee.go

// 定义一个处理函数
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

将`router`相关的代码独立后，`gee.go`简单了不少。最重要的还是通过实现了 `ServeHTTP` 接口，接管了所有的 HTTP 请求。相比第一天的代码，这个方法也有细微的调整，在调用 `router.handle` 之前，构造了一个 Context 对象。这个对象目前还非常简单，仅仅是包装了原来的两个参数，之后我们会慢慢地给 Context 插上翅膀。

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

> 注意到代码中的 package/import 均被省略，读者需自行补充，使用 GoLand 的话 IDE 会自动补上。


<div class="jump">
	<a href="#/./project/gee-1">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-3">Next</a>
</div>

---


> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day2.html
