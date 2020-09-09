# 4. Gee - 分组控制（Group）

- 实现路由分组控制(Route Group Control)，**代码约50行**

## 分组的意义

分组控制(Group Control)是 `Web` 框架应提供的基础功能之一。在真实的业务场景中，某些需要进行相同处理的路由可以归为一组，也就是按照路由分组。例如：

- 以`/admin`开头的路由需要鉴权；
- 以`/post`开头的路由匿名可访问；
- 以`/api`开头的路由是 `RESTful` 接口，可以对接第三方平台，需要三方平台鉴权。

大多数情况下，相同的前缀隶属于同一个组。因此，我们今天实现的分组控制也是以前缀来区分，并且支持分组的嵌套。例如`/post`是一个分组，`/post/a`和`/post/b`可以是该分组下的子分组。作用在`/post`分组上的中间件(middleware)，也都会作用在子分组，子分组还可以应用自己特有的中间件。

我们知道中间件可以给框架提供无限的扩展能力，而将中间件和分组结合，可以得到更加明显的收益。

- `/admin`的分组，可以应用鉴权中间件；
- `/`是默认的最顶层的分组，对其应用日志中间件，相当于所有的路由增加了记录日志的能力。

我们将在下一节介绍提供支持中间件的扩展能力。

## 分组嵌套

一个 Group 对象需要具备哪些属性呢？首先是前缀(prefix)，比如`/`，或者`/api`；要支持分组嵌套，那么还需要知道当前分组的父亲(parent)是谁；当然，按照我们一开始的分析，中间件是应用在分组上的，我们还需要存储应用在该分组上的中间件(middlewares)。

我们之前调用函数`(*Engine).addRoute()`来映射所有的路由规则和 Handler 。如果Group对象需要直接映射路由规则的话，比如我们想在使用框架时，这么调用：

```go
r := gee.New()
v1 := r.Group("/v1")
v1.GET("/", func(c *gee.Context) {
	c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
})
```

那么Group对象，还需要有访问`Router`的能力，为了方便，我们可以在Group中，保存一个指针，指向`Engine`，整个框架的所有资源都是由`Engine`统一协调的，那么就可以通过`Engine`间接地访问各种接口了。

总的来说，最后的 Group 的定义是这样的：

```go
// gee-web/gee/gee.go

type RouterGroup struct {
	prefix      string
	middleWares []HandlerFunc // 支持中间件
	parent      *RouterGroup  // 支持多级分组
	engine      *Engine       // 全局共用一个 engine 实例
}
```

我们还可以进一步地抽象，将`Engine`作为最顶层的分组，也就是说`Engine`拥有`RouterGroup`所有的能力。

```go
// gee-web/gee/gee.go

type Engine struct {
	*RouterGroup
	router *router
	groups []*RouterGroup // 存储所有分组
}
```

那我们就可以将和路由有关的函数，都交给`RouterGroup`实现了。

```go
// gee-web/gee/gee.go

// 初始化 engine
func New() *Engine {
	engine := &Engine{router: NewRouter()}
	engine.RouterGroup = &RouterGroup{engine: engine}
	engine.groups = []*RouterGroup{engine.RouterGroup}
	return engine
}

// 创建分组
func (group *RouterGroup) Group(prefix string) *RouterGroup {
	engine := group.engine
	newGroup := &RouterGroup{
		prefix: group.prefix + prefix,
		parent: group,
		engine: engine,
	}
	engine.groups = append(engine.groups, newGroup)
	return newGroup
}

func (group *RouterGroup) addRoute(method string, pattern string, handler HandlerFunc) {
	pattern = group.prefix + pattern
	log.Printf("Route %4s - %s", method, pattern)
	group.engine.router.addRoute(method, pattern, handler)
}

// 添加 Get 请求
func (group *RouterGroup) GET(pattern string, handler HandlerFunc) {
	group.addRoute("GET", pattern, handler)
}

// 添加 POST 请求
func (group *RouterGroup) POST(pattern string, handler HandlerFunc) {
	group.addRoute("POST", pattern, handler)
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

可以仔细观察下`addRoute`函数，调用了`group.engine.router.addRoute`来实现了路由的映射。由于`Engine`从某种意义上继承了`RouterGroup`的所有属性和方法，因为 `(*Engine).engine` 是指向自己的。这样实现，我们既可以像原来一样添加路由，也可以通过分组添加路由。

## 使用 Demo

测试框架的Demo就可以这样写了：

```go
// gee-web/main.go

func main() {
	r := gee.New()
	r.GET("/index", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Index Page</h1>")
	})
	v1 := r.Group("/v1")
	{
		v1.GET("/", func(c *gee.Context) {
			c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
		})

		v1.GET("/hello", func(c *gee.Context) {
			// expect /hello?name=geektutu
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Query("name"), c.Path)
		})
	}
	v2 := r.Group("/v2")
	{
		v2.GET("/hello/:name", func(c *gee.Context) {
			// expect /hello/geektutu
			c.String(http.StatusOK, "hello %s, you're at %s\n", c.Param("name"), c.Path)
		})
		v2.POST("/login", func(c *gee.Context) {
			c.JSON(http.StatusOK, gee.H{
				"username": c.PostForm("username"),
				"password": c.PostForm("password"),
			})
		})

	}

	r.Run(":9999")
}
```

通过 curl 简单测试：

```bash
$ curl "http://localhost:8080/api/speak?name=geek"
hello geek, you are at /api/speak

$ curl "http://localhost:8080/auth/assets/sixsixsix.jpg"
{"filepath":"sixsixsix.jpg"}
```



<div class="jump">
	<a href="#/./project/gee-3">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-5">Next</a>
</div>

---


> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day4.html