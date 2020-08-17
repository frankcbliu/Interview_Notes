# 5. Gee - 中间件（Middleware）

- 设计并实现 Web 框架的中间件(Middlewares)机制。
- 实现通用的`Logger`中间件，能够记录请求到响应的耗时，**代码约50行**。

## 中间件是什么

中间件(middlewares)，简单来说，就是业务无关的技术类组件。Web 框架本身不可能去理解所有的业务，因而不可能实现所有的功能。因此，框架需要有一个插口，允许用户自己定义功能，嵌入到框架中，仿佛这个功能是框架原生支持的一样。因此，对中间件而言，需要考虑2个比较关键的点：

- **插入点在哪？**使用框架的人并不关心底层逻辑的具体实现，如果插入点太底层，中间件逻辑就会非常复杂。如果插入点离用户太近，那和用户直接定义一组函数，每次在 Handler 中手工调用没有多大的优势了。
- **中间件的输入是什么？**中间件的输入，决定了扩展能力。暴露的参数太少，用户发挥空间有限。

那对于一个 Web 框架而言，中间件应该设计成什么样呢？我们接下来的实现，基本参考了 Gin 框架。

## 中间件设计

Gee 的中间件的定义与路由映射的 Handler 一致，处理的输入是`Context`对象。插入点是框架接收到请求初始化`Context`对象后，允许用户使用自己定义的中间件做一些额外的处理，例如记录日志等，以及对`Context`进行二次加工。另外通过调用`(*Context).Next()`函数，中间件可等待用户自己定义的 `Handler`处理结束后，再做一些额外的操作，例如计算本次处理所用时间等。即 Gee 的中间件支持用户在请求被处理的前后，做一些额外的操作。举个例子，我们希望最终能够支持如下定义的中间件，`c.Next()`表示等待执行其他的中间件或用户的`Handler`：

```go
// gee-web/gee/middlewares.go

// 统一日志打印中间件
func Logger() HandlerFunc {
	return func(ctx *Context) {
		// 开始计时
		t := time.Now()
		// 继续执行请求处理
		ctx.Next()
		// 计算耗时并打印日志
		log.Printf("[%d] %s in %v", ctx.StatusCode, ctx.Request.RequestURI, time.Since(t))
	}
}
```

另外，支持设置多个中间件，依次进行调用。

我们上一篇文章[分组控制 Group Control](http://www.szufrank.top/#/./project/gee-4)中讲到，中间件是应用在`RouterGroup`上的，应用在最顶层的 Group，相当于作用于全局，所有的请求都会被中间件处理。

事实上，我们使用中间件的目的，就是作为全局应用和某条路由级别的一个折衷，使得多条路由可以使用同一个中间件。为了更好地理解，我画了个图进行补充：

<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghsg86q2khj30zy0swmzj.jpg"></img>
</p>
我们之前的框架设计中，请求到达后，开始匹配路由，然后将请求的所有信息都保存在`Context`中。

中间件的处理也是类似，接收到请求后，先查找所有起作用的中间件，保存在`Context`中，然后再依次进行调用。为什么依次调用后，还需要在`Context`中保存呢？因为在设计中，中间件不仅作用在处理流程前，也可以作用在处理流程后，即在用户定义的 Handler 处理完毕后，还可以执行剩下的操作。

为此，我们给`Context`添加了2个参数，定义了`Next`方法：

```go
// gee-web/gee/context.go

type Context struct {
	// origin objects
	Writer  http.ResponseWriter
	Request *http.Request
	// 请求信息
	Method string
	Path   string
	Params map[string]string
	// 返回信息
	StatusCode int
	// 中间件信息
	handlers []HandlerFunc
	index    int
}

func NewContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer:  w,
		Request: req,
		Method:  req.Method,
		Path:    req.URL.Path,
		index:   -1, // 初始化为 -1
	}
}

// 依次调用当前 Context 中所有的中间件
func (c *Context) Next() {
	c.index++
	size := len(c.handlers)
	// 这里遍历所有 handler，是因为不是所有 handler 都会手动调用 c.Next()
	// 对于只作用于请求前的 handler，可以省略 c.Next()
	for ; c.index < size; c.index++ {
		c.handlers[c.index](c)
	}
}

// 返回失败信息
func (c *Context) Fail(code int, err string) {
	c.index = len(c.handlers)
	c.JSON(code, H{"message": err})
}
```

`index`是记录当前执行到第几个中间件，当在中间件中调用`Next`方法时，控制权交给了下一个中间件，直到调用到最后一个中间件，然后再从后往前，调用每个中间件在`Next`方法之后定义的部分。如果我们将用户在映射路由时定义的`Handler`添加到`c.handlers`列表中，结果会怎么样呢？想必你已经猜到了。

```go
func A(c *Context) {
    part1
    c.Next()
    part2
}
func B(c *Context) {
    part3
    c.Next()
    part4
}
```

假设我们应用了中间件 A 和 B，和路由映射的 Handler。`c.handlers`是这样的`[A, B, Handler]`，`c.index`初始化为`-1`。调用`c.Next()`，接下来的流程是这样的：

- `c.index++` ，变为 `0`
- 因为`0 < 3`，调用 `c.handlers[0]`，即 `A(c)`
- 执行 `part1`，调用 `c.Next()`
- `c.index++`，变为 `1`
- 因为`1 < 3`，调用 `c.handlers[1]`，即 `B(c)`
- 执行 `part3`，调用 `c.Next()`
- `c.index++`，变为 2
- `2 < 3`，调用 `c.handlers[2]`，即`Handler`
- `Handler` 调用完毕，返回到 `B` 中的 `part4`，执行 `part4`
- `part4` 执行完毕，返回到 `A` 中的 `part2`，执行 `part2`
- `part2` 执行完毕，结束。

一句话说清楚重点，最终的顺序是`part1 -> part3 -> Handler -> part 4 -> part2`。恰恰满足了我们对中间件的要求，接下来看调用部分的代码，就能全部串起来了。

## 代码实现

- 定义`Use`函数，将中间件应用到某个 Group 。

```go
// gee-web/gee/gee.go

// 将中间件应用到某个 group
func (group *RouterGroup) Use(middleWares ...HandlerFunc) {
	group.middleWares = append(group.middleWares, middleWares...)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	var middleWares []HandlerFunc
	// 将分组中的中间件加入到 middleWares 中
	for _, group := range engine.groups {
		if strings.HasPrefix(req.URL.Path, group.prefix) {
			middleWares = append(middleWares, group.middleWares...)
		}
	}

	c := NewContext(w, req)
	// 将 middleWares 赋值给 Context 中的 handlers
	c.handlers = middleWares
	engine.router.handle(c)
}
```

ServeHTTP 函数也有变化，当我们接收到一个具体请求时，要判断该请求适用于哪些中间件，在这里我们简单通过 URL 的前缀来判断。得到中间件列表后，赋值给 `c.handlers`。

- handle 函数中，将从路由匹配得到的 Handler 添加到 `c.handlers`列表中，执行`c.Next()`。

```go
// gee-web/gee/router.go

// 处理路由函数
func (r *router) handle(c *Context) {
	// 获取节点和从路由中解析出来的参数
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil { // 节点是否存在，是判断路由是否存在的依据
		c.Params = params
		key := c.Method + "-" + n.pattern
		if handler, ok := r.handlers[key]; ok {
			c.handlers = append(c.handlers, handler)
		} else {
			c.handlers = append(c.handlers, func(context *Context) {
				c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
			})
		}
	}
	c.Next()
}
```

## 使用 Demo

为了更好的展示中间件，这里我们再新增一个中间件：

```go
// gee-web/gee/middlewares.go

// 仅用于 auth 路由的测试中间件
func OnlyForAuth() HandlerFunc {
	return func(c *Context) {
		// 开始计时
		t := time.Now()
		// 假设当前服务返回出错
		c.Fail(500, "Internal Server Error")
		// 计算耗时并打印日志
		log.Printf("[%d] %s in %v for group Auth", c.StatusCode, c.Request.RequestURI, time.Since(t))
	}
}
```

接下来我们看主文件：

```go
// gee-web/gee/main.go

func main() {
	r := gee.New()
	r.Use(gee.Logger()) // 全局使用 Logger 中间件
	r.GET("/hello", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Hello Gee!</h1>\n")
	})

	api := r.Group("/api")
	{
		api.GET("/", func(c *gee.Context) {
			c.HTML(http.StatusOK, "<h1>Hello Api.</h1>\n")
		})
		api.GET("/speak", func(c *gee.Context) {
			// expect /speak?name=geek
			c.String(http.StatusOK, "hello %s, you are at %s\n", c.Query("name"), c.Path)
		})
	}

	auth := r.Group("/auth")
	auth.Use(gee.OnlyForAuth()) // 只有 /auth 使用 OnlyForAuth 中间件，返回500
	{
		auth.GET("/hello/:name", func(c *gee.Context) {
			// expect /hello/geek
			c.String(http.StatusOK, "hello %s, you are at %s\n", c.Param("name"), c.Path)
		})
		auth.GET("/assets/*filepath", func(c *gee.Context) {
			c.JSON(http.StatusOK, gee.H{"filepath": c.Param("filepath")})
		})
	}
	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

`gee.Logger()`即我们一开始就介绍的中间件，我们将这个中间件和框架代码放在了一起，作为框架默认提供的中间件。在这个例子中，我们将`gee.Logger()`应用在了全局，所有的路由都会应用该中间件。`OnlyForAuth()`是用来测试功能的，仅在`/auth`对应的 Group 中应用了。

接下来使用 `curl` 测试：

- 访问全局中间件路径：

```bash
$ curl localhost:8080/hello
<h1>Hello Gee!</h1>

>>> log
2020/08/16 23:42:09 [200] /hello in 14.122µs

$ curl localhost:8080/api/speak      
hello , you are at /api/speak

>>> log
2020/08/16 23:44:54 [200] /api/speak in 11.84µs
```

- 访问使用了全局中间件和`/auth`特有的中间件的路径：

```bash
$ curl localhost:8080/auth/hello/geek
{"message":"Internal Server Error"}

>>> log
2020/08/16 23:43:03 [500] /auth/hello/geek in 136.53µs for group Auth
2020/08/16 23:43:03 [500] /auth/hello/geek in 195.26µs
```


<div class="jump">
	<a href="#/./project/gee-4">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-6">Next</a>
</div>

---


> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day5.html
