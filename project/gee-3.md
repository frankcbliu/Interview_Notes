# 3. Gee - Trie 树路由（Router）

- 使用 `Trie` 树(前缀树)实现动态路由(dynamic route)解析。
- 支持两种模式`:name`和`*filepath`，**代码约150行**。

## 动态路由

在讲今天的内容之前，我们先来了解一下什么是动态路由。先看与之相对的静态路由：

- 我们在代码中定义了`/hello`所对应的`handleFunc`，则当我们的`URL`为`/hello`时，会由该函数去解析；

如果是动态路由呢？

- 我们定义了`/hello/:name`->`handleFunc`，当输入 URL 为`/hello/geek`时，会由该函数去解析，例如可以解析出`name: geek`；当 URL 为`/hello/frank`时，则可以解析出`name: frank`。
- 我们定义了`/hello/static/*filepath`规则，当输入`/hello/static/css/geek.css`时，会由该函数解析，得到`filepath: css/geek.css`。

总而言之，所谓动态路由，即**一条路由规则**可以匹配**某一类型而非某一条固定的路由**。例如`/hello/:name`，可以匹配`/hello/geek`、`hello/frank`等。

## Trie 树简介

在之前的版本中，我们使用非常简单的`map`结构来存储路由表，索引非常高效，但弊端也很明显，键值对的存储的方式，只能用来索引静态路由。如果我们想支持类似于`/hello/:name`这样的动态路由，怎么办呢？

动态路由有很多种实现方式，支持的规则、性能等有很大的差异。例如开源的路由实现`gorouter`支持在路由规则中嵌入正则表达式，例如`/p/[0-9A-Za-z]+`，即路径中的参数仅匹配数字和字母；另一个开源实现`httprouter`就不支持正则表达式。著名的`Web`开源框架`gin` 在早期的版本中，并没有实现自己的路由，而是直接使用了`httprouter`，后来不知为何，放弃了`httprouter`，自己实现了一个版本。


<p align="center">
<img  src="https://geektutu.com/post/gee-day3/trie_eg.jpg"></img>
</p>

实现动态路由最常用的数据结构，被称为`前缀树(Trie树)`。看到名字你大概也能知道前缀树长啥样了：每一个节点的所有的子节点都拥有相同的前缀。这种结构非常适用于路由匹配，比如我们定义了如下路由规则：

- `/:lang/intro`
- `/:lang/tutorial`
- `/:lang/doc`
- `/about`
- `/p/blog`
- `/p/related`

我们用前缀树来表示，是这样的：

<p align="center">
<img src="https://geektutu.com/post/gee-day3/trie_router.jpg"></img>
</p>

HTTP请求的路径恰好是由`/`分隔的多段构成的，因此，我们将每一段视作前缀树的一个节点。当我们需要进行路由匹配时，通过对  Trie 树查询，如果中间某一层的节点都不满足条件，那么就说明没有匹配成功，查询结束。

接下来我们实现的动态路由具备以下两个功能。

- 参数匹配`:`。例如 `/p/:lang/doc`，可以匹配 `/p/c/doc` 和 `/p/go/doc`。
- 通配`*`。例如 `/static/*filepath`，可以匹配`/static/fav.ico`，也可以匹配`/static/js/jQuery.js`，这种模式常用于静态服务器，能够递归地匹配子路径。

## Trie 树实现

首先我们需要设计树节点上应该存储哪些信息：

```go
// gee-web/gee/trie.go

type node struct {
	pattern  string  // 待匹配路由，例如 /p/:lang
	part     string  // 路由中的一部分，例如 :lang
	children []*node // 子节点，例如 [doc, tutorial, intro]
	isWild   bool    // 是否模糊匹配，part 开头为 : 或 * 时为true
}
```

与普通的树不同，为了实现动态路由匹配，加上了`isWild`这个参数。即当我们匹配 `/p/go/doc/`这个路由时，第一层节点，`p`精准匹配到了`p`，第二层节点，`go`模糊匹配到`:lang`，那么将会把`lang`这个参数赋值为`go`，继续下一层匹配。我们将匹配的逻辑，包装为一个辅助函数。

```go
// gee-web/gee/trie.go

// 子节点中第一个匹配成功的节点，用于插入
func (n *node) matchChild(part string) *node {
	for _, child := range n.children { // 遍历所有子节点
		// 满足子节点的 part 与当前 part 相等 or 当前子节点为模糊匹配
		if child.part == part || child.isWild {
			return child
		}
	}
	return nil
}

// 所有匹配成功的子节点，用于查找
func (n *node) matchChildren(part string) []*node {
	nodes := make([]*node, 0)
	for _, child := range n.children { // 遍历所有子节点
		// 满足子节点的 part 与当前 part 相等 or 当前子节点为模糊匹配
		if child.part == part || child.isWild {
			nodes = append(nodes, child)
		}
	}
	return nodes
}
```

对于路由来说，最重要的当然是注册与匹配了。开发服务时，注册路由规则，映射handler；访问时，匹配路由规则，查找到对应的handler。因此，Trie 树需要支持节点的插入与查询。插入功能很简单，递归查找每一层的节点，如果没有匹配到当前`part`的节点，则新建一个，有一点需要注意，`/p/:lang/doc`只有在第三层节点，即`doc`节点，`pattern`才会设置为`/p/:lang/doc`。`p`和`:lang`节点的`pattern`属性皆为空。因此，当匹配结束时，我们可以使用`n.pattern == ""`来判断路由规则是否匹配成功。例如，`/p/python`虽能成功匹配到`:lang`，但`:lang`的`pattern`值为空，因此匹配失败。查询功能，同样也是递归查询每一层的节点，退出规则是，匹配到了`*`，匹配失败，或者匹配到了第`len(parts)`层节点。

```go
// gee-web/gee/trie.go

// 注册路由时，插入结点
func (n *node) insert(pattern string, parts []string, depth int) {
	if len(parts) == depth { // 当深度到达目标层时
		n.pattern = pattern // 设置当前节点的 pattern 为注册的 pattern
		return
	}

	part := parts[depth]
	child := n.matchChild(part)
	if child == nil {
		child = &node{ //  非最底层的节点不设置 pattern
			part:   part,
			isWild: part[0] == ':' || part[0] == '*',
		}
		// 将匹配到的第一个子节点加入当前节点的 children 列表中
		n.children = append(n.children, child)
	}
	// 递归调用
	child.insert(pattern, parts, depth+1)
}

// 匹配路由时，需要查找满足条件的节点
func (n *node) search(parts []string, depth int) *node {
	if len(parts) == depth || strings.HasPrefix(n.part, "*") { // 到达最底层或者当前为 * 的模糊匹配
		if n.pattern == "" { // pattern 为空，说明未到最底层，查找失败
			return nil
		}
		return n
	}

	part := parts[depth]
	children := n.matchChildren(part)

	for _, child := range children { // 遍历所有满足条件的子节点
		result := child.search(parts, depth+1) // 递归调用
		if result != nil {
			return result
		}
	}
	return nil
}
```

## Router

Trie 树的插入与查找都成功实现了，接下来我们将 Trie 树应用到路由中去吧。我们使用 roots 来存储每种请求方式的Trie 树根节点。使用 handlers 存储每种请求方式的 HandlerFunc 。getRoute 函数中，还解析了`:`和`*`两种匹配符的参数，返回一个 map 。例如`/p/go/doc`匹配到`/p/:lang/doc`，解析结果为：`{lang: "go"}`，`/static/css/geek.css`匹配到`/static/*filepath`，解析结果为`{filepath: "css/geek.css"}`。

```go
// gee-web/gee/router.go

type router struct {
	roots    map[string]*node       // 存储根节点
	handlers map[string]HandlerFunc // 存储节点到 handleFunc 的映射
}

func NewRouter() *router {
	return &router{
		roots:    make(map[string]*node),
		handlers: make(map[string]HandlerFunc),
	}
}

// 解析 pattern
// 如果路由是 /static/*filepath，则 results = [static]
// 如果路由是 /static/:name/doc，则 results = [static, :name, doc]
func parsePattern(pattern string) []string {
	parts := strings.Split(pattern, "/")

	results := make([]string, 0)

	for _, part := range parts {
		if part != "" {
			results = append(results, part)
			if part[0] == '*' {
				break
			}
		}
	}
	return results
}

// 往路由表中添加路由
func (r *router) addRoute(method string, pattern string, handler HandlerFunc) {
	log.Printf("Route %4s - %s", method, pattern)
	parts := parsePattern(pattern)
	key := method + "-" + pattern

	// 获取根节点
	_, ok := r.roots[method]
	if !ok { // 不存在则创建
		r.roots[method] = &node{}
	}
	// 往根节点中插入
	r.roots[method].insert(pattern, parts, 0)
	r.handlers[key] = handler
}

// 从路由表中查询节点
func (r *router) getRoute(method string, path string) (*node, map[string]string) {
	searchParts := parsePattern(path) // 获取 parts 数组
	params := make(map[string]string)

	root, ok := r.roots[method] // 获取根节点
	if !ok {
		return nil, nil
	}

	n := root.search(searchParts, 0)

	if n != nil { // 匹配节点非空
		parts := parsePattern(n.pattern)
		for index, part := range parts {
			if part[0] == ':' {
				params[part[1:]] = searchParts[index]
			}
			if part[0] == '*' && len(part) > 1 { // *开头，且不只有*
				params[part[1:]] = strings.Join(searchParts[index:], "/")
				break
			}
		}
		return n, params
	}
	return nil, nil
}

// 处理路由函数
func (r *router) handle(c *Context) {
	// 获取节点和从路由中解析出来的参数
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil { // 节点是否存在，是判断路由是否存在的依据
		c.Params = params
		key := c.Method + "-" + c.Path
		if handler, ok := r.handlers[key]; ok {
			handler(c)
		} else {
			c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
		}
	}
}
```

## Context与handle的变化

在 HandlerFunc 中，希望能够访问到解析的参数，因此，需要对 Context 对象增加一个属性和方法，来提供对路由参数的访问。我们将解析后的参数存储到`Params`中，通过`c.Param("lang")`的方式获取到对应的值。

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
}

// 根据参数获取值
func (c *Context) Param(key string) string {
	value, ok := c.Params[key]
	if !ok {
		log.Printf("Find Param key: %v error!", key)
	}
	return value
}
```

接下来我们修改 `router` 中的 `handle`函数：

```go
// gee-web/gee/router.go

// 处理路由函数
func (r *router) handle(c *Context) {
	// 获取节点和从路由中解析出来的参数
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil { // 节点是否存在，是判断路由是否存在的依据
		c.Params = params
		key := c.Method + "-" + c.Path
		if handler, ok := r.handlers[key]; ok {
			handler(c)
		} else {
			c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
		}
	}
}
```

`router.go`的变化比较小，比较重要的一点是，在调用匹配到的`handler`前，将解析出来的路由参数赋值给了`c.Params`。这样就能够在`handler`中，通过`Context`对象访问到具体的值了。

## 单元测试

使用 GoLand 的同学可以打开`router.go`文件，右键选择：`Generate...->Tests for file`，会自动生成单测文件，只需再补充用例即可。

`NewRouter`、`addRoute`和`handle`不太好测，这里就省略了 。

```go
// gee-web/gee/router_test.go
package gee

import (
	"reflect"
	"testing"
)

func Test_parsePattern(t *testing.T) {
	tests := []struct {
		name    string
		pattern string
		want    []string
	}{ // 单测用例
		{name: "/p/*", pattern: "/p/*", want: []string{"p", "*"}},
		{name: "/p/:name", pattern: "/p/:name", want: []string{"p", ":name"}},
		{name: "/p/*name/*", pattern: "/p/*name/*", want: []string{"p", "*name"}},
		{name: "/p/:name/b/*", pattern: "/p/:name/b/*", want: []string{"p", ":name", "b", "*"}},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := parsePattern(tt.pattern); !reflect.DeepEqual(got, tt.want) {
				t.Errorf("parsePattern() got = %v, want %v", got, tt.want)
			}
		})
	}
}

func newTestRouter() *router {
	r := NewRouter()
	r.addRoute("GET", "/", nil)
	r.addRoute("GET", "/hello/:name", nil)
	r.addRoute("GET", "/hello/b/c", nil)
	r.addRoute("GET", "/hi/:name", nil)
	r.addRoute("GET", "/assets/*filepath", nil)
	return r
}
func Test_router_getRoute(t *testing.T) {
	type args struct {
		method string
		path   string
	}
	tests := []struct {
		name  string
		args  args
		want  string
		want1 map[string]string
	}{ // 单测用例
		{name: "geek", args: args{"GET", "/hello/geek"}, want: "/hello/:name", want1: map[string]string{"name": "geek"}},
		{name: "frank", args: args{"GET", "/hello/frank"}, want: "/hello/:name", want1: map[string]string{"name": "frank"}},
		{name: "hello_b_c", args: args{"GET", "/hello/b/c"}, want: "/hello/b/c", want1: map[string]string{}},
		{name: "assets", args: args{"GET", "/assets/233.jpg"}, want: "/assets/*filepath", want1: map[string]string{"filepath": "233.jpg"}},
	}
	r := newTestRouter()
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, got1 := r.getRoute(tt.args.method, tt.args.path)
			if !reflect.DeepEqual(got.pattern, tt.want) {
				t.Errorf("getRoute() got = %v, want %v", got.pattern, tt.want)
			}
			if !reflect.DeepEqual(got1, tt.want1) {
				t.Errorf("getRoute() got1 = %v, want %v", got1, tt.want1)
			}
		})
	}
}
```

## 使用Demo

看看框架使用的样例吧。

```go
// gee-web/main.go

func main() {
	r := gee.New()
	r.GET("/", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Hello Gee!</h1>\n")
	})
	r.GET("/hello", func(c *gee.Context) {
		// expect /hello?name=geek
		c.String(http.StatusOK, "hello %s, you are at %s\n", c.Query("name"), c.Path)
	})
	r.GET("/hello/:name", func(c *gee.Context) {
		// expect /hello/geek
		c.String(http.StatusOK, "hello %s, you are at %s\n", c.Param("name"), c.Path)
	})
	r.GET("/assets/*filepath", func(c *gee.Context) {
		c.JSON(http.StatusOK, gee.H{"filepath": c.Param("filepath")})
	})
	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

使用`curl`工具，测试结果。

```bash
$ curl "http://localhost:8080/hello/geek"
hello geekt, you are at /hello/geek

$ curl "http://localhost:8080/assets/css/geek.css"
{"filepath":"css/geek.css"}
```