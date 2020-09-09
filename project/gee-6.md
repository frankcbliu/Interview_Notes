# 6. Gee - HTML 模板（Template）

- 支持静态资源服务(Static Resource)。
- 支持`HTML`模板渲染。


## 服务端渲染

现在越来越流行**前后端分离**的开发模式，即后端提供 `RESTful` 接口，返回结构化的数据(通常为 `JSON` 或者 `XML`)；前端则使用 `AJAX` 技术请求对应的后端接口，获取数据后，利用 `JavaScript` 进行渲染。

`Vue/React` 等前端框架持续火热，这种开发模式下前后端解耦，优势非常突出。

- 后端同学专心解决资源利用，并发，数据库等问题，只需要考虑数据如何生成；
- 前端同学专注于界面设计，只需要考虑拿到数据后如何渲染即可。

使用过 `JSP` 的同学，应该能感受到前后端耦合的痛苦。`JSP` 的表现力是远不如 `Vue/React` 等专业前端渲染框架的。而且前后端分离在当前还有另外一个不可忽视的优势。因为后端只关注于数据，接口返回值是结构化的，与前端解耦。同一套后端服务能够同时支撑小程序、移动APP、PC端 Web 页面，以及对外提供的接口。随着前端工程化的不断地发展，`Webpack、gulp` 等工具层出不穷，前端技术越来越自成体系了。

当然，前后端分离也有缺点，因为页面是在客户端进行渲染的，这对爬虫来说并不友好。`Google` 爬虫已经能够爬取渲染后的网页，但是短期内爬取服务端直接渲染的 `HTML` 页面仍是主流。

今天的内容便是介绍 Web 框架如何支持`服务端渲染`的场景。

## 静态文件(Serve Static Files)

我们都知道前端三剑客，`JavaScript`、`CSS` 和 `HTML`。要实现服务端渲染，首先便是要支持 JS、CSS 等静态文件。在之前我们设计动态路由时，支持了通配符`*`匹配多级子路径。比如路由规则`/assets/*filepath`，可以匹配`/assets/`开头的所有的地址。例如`/assets/js/geektutu.js`，匹配后，参数`filepath`就赋值为`js/geektutu.js`。

那如果我么将所有的静态文件放在`/static/`目录下，那么`filepath`的值即是该目录下文件的相对地址。映射到真实的文件后，将文件返回，静态服务器就实现了。

找到文件后，如何返回这一步，`net/http`库已经实现了。因此，gee 框架要做的，仅仅是解析请求的地址，映射到服务器上文件的真实地址，交给`http.FileServer`处理就好了。

```go
// gee-web/gee/gee.go

// 创建静态文件处理 handler
func (group *RouterGroup) CreateStaticHandler(relativePath string, fs http.FileSystem) HandlerFunc {
	absolutePath := path.Join(group.prefix, relativePath)
	fileServer := http.StripPrefix(absolutePath, http.FileServer(fs))
	return func(ctx *Context) {
		file := ctx.Param("filepath")
		// 判断文件是否存在 or 是否有权限处理文件
		if _, err := fs.Open(file); err != nil {
			ctx.Status(http.StatusNotFound)
			return
		}
		fileServer.ServeHTTP(ctx.Writer, ctx.Request)
	}
}

// 将硬盘上的 root 路径映射到路由 relativePath 上
func (group *RouterGroup) Static(relativePath string, root string) {
	handler := group.CreateStaticHandler(relativePath, http.Dir(root))
	urlPattern := path.Join(relativePath, "/*filepath")
	// 注册 handler
	group.GET(urlPattern, handler)
}
```

我们给`RouterGroup`添加了2个方法，`Static(relativePath, root)`这个方法是暴露给用户的。用户可以将磁盘上的某个文件夹`root`映射到路由`relativePath`。例如：

```go
r := gee.New()
r.Static("/assets", "/gee-web/static")
// 或相对路径 r.Static("/assets", "./static")
r.Run(":8080")
```

用户访问`localhost:8080/assets/js/geek.js`，最终返回`/gee-web/static/js/geek.js`。

## HTML 模板渲染

Go语言内置了`text/template`和`html/template`2个模板标准库，其中[html/template](https://golang.org/pkg/html/template/)为 HTML 提供了较为完整的支持。包括普通变量渲染、列表渲染、对象渲染等。`gee` 框架的模板渲染直接使用了`html/template`提供的能力。

```go
// gee-web/gee/gee.go

type Engine struct {
	*RouterGroup
	router        *router
	groups        []*RouterGroup     // 存储所有分组
	htmlTemplates *template.Template // 用于 html 渲染
	funcMap       template.FuncMap
}

// 设置 funcMap
func (engine *Engine) SetFuncMap(funcMap template.FuncMap) {
	engine.funcMap = funcMap
}

// 渲染函数
func (engine *Engine) LoadHTMLGlob(pattern string) {
	engine.htmlTemplates = template.Must(template.New("").Funcs(engine.funcMap).ParseGlob(pattern))
}
```

首先为 Engine 示例添加了 `*template.Template` 和 `template.FuncMap`对象，前者将所有的模板加载进内存，后者则是模板渲染函数。

另外，给用户分别提供了设置自定义渲染函数`funcMap`和加载模板的方法。

接下来，对原来的 `(*Context).HTML()`方法做了些小修改，使之支持根据模板文件名选择模板进行渲染。

```go
// gee-web/gee/context.go

type Context struct {
	// ...
	// engine 指针
	engine *Engine
}

// 返回 HTML
func (c *Context) HTML(code int, name string, data interface{}) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	if err := c.engine.htmlTemplates.ExecuteTemplate(c.Writer, name, data); err != nil {
		c.Fail(http.StatusInternalServerError, err.Error())
	}
}
```

我们在 `Context` 中添加了成员变量 `engine *Engine`，这样就能够通过 `Context` 访问 `Engine` 中的 `HTML` 模板。实例化 `Context` 时，还需要给 `c.engine` 赋值。

```go
// gee-web/gee/gee.go

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	// ...
	c := NewContext(w, req)
	// 给 ctx 的 engine 赋值
	c.engine = engine
	// 将 middleWares 赋值给 Context 中的 handlers
	c.handlers = middleWares
	engine.router.handle(c)
}
```

## Demo

最终的目录结构

```go
.
├── README.md
├── gee
│   ├── context.go
│   ├── gee.go
│   ├── go.mod
│   ├── middlewares.go
│   ├── router.go
│   ├── router_test.go
│   └── trie.go
├── go.mod
├── main.go
├── static
│   └── css
│       └── geek.css
└── templates
    ├── arr.tmpl
    ├── css.tmpl
    └── custom_func.tmpl
```

新增模板文件：

```html
<!-- templates/css.tmpl -->
<html>
    <link rel="stylesheet" href="/assets/css/geek.css">
    <p>geek.css is loaded.</p>
</html>
```



```go
// gee-web/gee/main.go

type student struct {
	Name string
	Age  int8
}

func formatAsDate(t time.Time) string {
	year, month, day := t.Date()
	return fmt.Sprintf("%d-%02d-%02d", year, month, day)
}

func main() {
	r := gee.New()
	r.Use(gee.Logger())
	r.SetFuncMap(template.FuncMap{ // 自定义渲染函数
		"formatAsDate": formatAsDate,
	})
	r.LoadHTMLGlob("templates/*") // 加载渲染模板
	r.Static("/assets", "./static")

	api := r.Group("/api")
	{
		api.GET("/", func(c *gee.Context) {
			c.HTML(http.StatusOK, "css.tmpl", nil)
		})
		api.GET("/speak", func(c *gee.Context) {
			// expect /speak?name=geek
			c.String(http.StatusOK, "hello %s, you are at %s\n", c.Query("name"), c.Path)
		})
		stu1 := &student{Name: "geek", Age: 20}
		stu2 := &student{Name: "frank", Age: 22}
		api.GET("/students", func(c *gee.Context) {
			c.HTML(http.StatusOK, "arr.tmpl", gee.H{
				"title":  "gee",
				"stuArr": [2]*student{stu1, stu2},
			})
		})
		api.GET("/date", func(c *gee.Context) {
			c.HTML(http.StatusOK, "custom_func.tmpl", gee.H{
				"title": "gee",
				"now":   time.Now(),
			})
		})
	}
	// auth...

	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

访问下`localhost:8080/api`，模板正常渲染，CSS 静态文件加载成功：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghtr4fdy5nj30k4052q36.jpg"></img>
</p>

访问`localhost:8080/api/students`，成功将数据加载到模板中：

<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghtr4mkjy6j30q009amxt.jpg"></img>
</p>


<div class="jump">
	<a href="#/./project/gee-5">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-7">Next</a>
</div>

---

> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day6.html
