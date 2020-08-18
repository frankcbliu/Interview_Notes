# 7. Gee - 错误恢复（Panic Recover）

- 实现错误处理机制。

## panic

Go 语言中，比较常见的错误处理方法是返回 `error`，由调用者决定后续如何处理。但如果是无法恢复的错误，则可以手动触发 `panic`，同时，如果在程序运行过程中出现了类似于数组越界的错误，`panic` 也会被触发。所谓`panic` ，会中止当前执行的程序并退出。

下面是主动触发的例子：

```go
// testPanic.go
package main

import "fmt"

func main() {
	fmt.Println("before panic")
	panic("crash")
	fmt.Println("after panic")
}
```

执行后：

```bash
$ go run testPanic.go
before panic
panic: crash

goroutine 1 [running]:
main.main()
	/Users/Code/goProject/testPanic.go:8 +0x95
exit status 2
```

下面是数组越界触发的 `panic`

```go
// testPanic.go
package main

import "fmt"

func main() {
	arr := []int{1, 2, 3}
	fmt.Println(arr[4])
}
```

执行：

```bash
$ go run testPanic.go
panic: runtime error: index out of range [4] with length 3

goroutine 1 [running]:
main.main()
	/Users/Code/goProject/testPanic.go:8 +0x1d
exit status 2
```

## defer

`panic` 会导致程序被中止，但是在退出前，会先处理完当前协程上已经`defer` 的任务，执行完成后再退出。

```go
// testDefer.go
func main() {
	defer func() {
		fmt.Println("defer func")
	}()

	arr := []int{1, 2, 3}
	fmt.Println(arr[4])
}
```
在这里，`defer` 的任务执行完成之后，`panic` 还会继续被抛出，导致程序非正常结束。
```bash
$ go run testDefer.go 
defer func
panic: runtime error: index out of range [4] with length 3
```

可以 defer 多个任务，在同一个函数中 `defer` 多个任务，会逆序执行。即先执行最后 `defer` 的任务，原理很简单，函数中的`defer`函数会被压入`defer 栈`中，执行时依次弹栈执行。

## recover

Go 语言还提供了 `recover` 函数，可以避免因为 `panic` 发生而导致整个程序终止，`recover` 函数只能在 `defer` 中生效。

```go
// testRecover.go

func test_recover() {
	defer func() {
		fmt.Println("defer func")
		if err := recover(); err != nil {
			fmt.Println("recover success")
		}
	}()

	arr := []int{1, 2, 3}
	fmt.Println(arr[4])
	fmt.Println("after panic")
}

func main() {
	test_recover()
	fmt.Println("after recover")
}
```

执行：

```bash
$ go run testRecover.go 
defer func
recover success
after recover
```

我们可以看到，`recover` 捕获了 `panic`，程序正常结束。`test_recover()` 中的`after panic` 没有打印，这是正确的，当 `panic` 被触发时，控制权就被交给了 `defer` 。而在`main()` 中打印了`after recover`，说明程序已经恢复正常，继续往下执行直到结束。

## Gee 的错误处理机制

对一个 `Web` 框架而言，错误处理机制是非常必要的。可能是框架本身没有完备的测试，导致在某些情况下出现空指针异常，也有可能是用户不正确的参数，触发了某些异常，例如数组越界，空指针等。如果因为这些原因导致系统宕机，是断然不可接受的。

我们在第六弹中实现的框架并没有加入异常处理机制，如果代码中存在会触发 `panic` 的 `BUG`，很容易宕掉。

>  `net/http`内部实现了`recover()`，因此`panic`时不会整个程序挂掉，但是会导致这次路由没有返回；而我们实现的`Recovery`中间件，保证了每一次路由都有返回。

例如下面的代码：

```go
func main() {
	r := gee.New()
	r.GET("/panic", func(c *gee.Context) {
		names := []string{"geek"}
		c.String(http.StatusOK, names[100])
	})
	r.Run(":8080")
}
```

在上面的代码中，我们为 `gee` 注册了路由 `/panic`，而这个路由的处理函数内部存在数组越界 `names[100]`，如果访问 `localhost:8080/panic`，Web 服务就会宕掉。

今天，我们将在 `gee` 中添加一个非常简单的错误处理机制，即在此类错误发生时，向用户返回 `Internal Server Error`，并且在日志中打印必要的错误信息，方便进行错误定位。

我们之前实现了中间件机制，错误处理也可以作为一个中间件，增强 `gee` 框架的能力。

新增文件 `gee/recovery.go`，在这个文件中实现中间件 `Recovery`。

```go
// gee-web/gee/recovery.go
h
func Recovery() HandlerFunc {
	return func(c *Context) {
		defer func() {
			if err := recover(); err != nil {
				message := fmt.Sprintf("%s", err)
				log.Printf("%s\n\n", trace(message))
				c.Fail(http.StatusInternalServerError, "Internal Server Error")
			}
		}()

		c.Next()
	}
}
```

`Recovery` 的实现非常简单，使用 defer 挂载上错误恢复的函数，在这个函数中调用 `recover()`，捕获 `panic`，并且将堆栈信息打印在日志中，向用户返回 `Internal Server Error`。

你可能注意到，这里有一个 `trace()` 函数，这个函数是用来获取触发 `panic` 的堆栈信息，完整代码如下：

```go
// gee-web/gee-recovery.go

package gee

import (
	"fmt"
	"log"
	"net/http"
	"runtime"
	"strings"
)

func Recovery() HandlerFunc {
	return func(ctx *Context) {
		defer func() {
			if err := recover(); err != nil {
				message := fmt.Sprintf("%s", err)
				log.Printf("%s\n\n", trace(message))
				ctx.Fail(http.StatusInternalServerError, "Internal Server Error")
			}
		}()
		ctx.Next()
	}
}

// 获取出错文件和行号
func trace(message string) string {
	var pcs [32]uintptr
	n := runtime.Callers(3, pcs[:]) // 跳过开始的 3 个caller

	var str strings.Builder
	str.WriteString(message + "\nTraceBack: ")
	for _, pc := range pcs[:n] {
		fn := runtime.FuncForPC(pc)
		file, line := fn.FileLine(pc) // 获取文件号和行号
		str.WriteString(fmt.Sprintf("\n\t%s: %d", file, line))
	}
	return str.String()
}
```

在 `trace()` 中，调用了 `runtime.Callers(3, pcs[:])`，`Callers` 用来返回调用栈的程序计数器, 第 0 个 `Caller` 是 `Callers` 本身，第 1 个是上一层 `trace`，第 2 个是再上一层的 `defer func`。因此，为了日志简洁一点，我们跳过了前 3 个 `Caller`。

接下来，通过 `runtime.FuncForPC(pc)` 获取对应的函数，在通过 `fn.FileLine(pc)` 获取到调用该函数的文件名和行号，打印在日志中。

至此，`gee` 框架的错误处理机制就完成了。

## 使用 Demo

```go
// gee-web/gee/main.go

package main

import (
	"net/http"
	"gee"
)

func main() {
	r := gee.New()
	r.Use(gee.Logger(), gee.Recovery()) // 默认使用日志中间件和错误恢复

	// 数组越界错误：用于测试 Recovery()
	r.GET("/panic", func(c *gee.Context) {
		names := []string{"geek"}
		c.String(http.StatusOK, names[100])
	})

	err := r.Run(":8080")
	if err != nil { // 启动服务失败
		panic("start server error!")
	}
}
```

接下来进行测试，先访问主页，访问一个有`BUG`的 `/panic`，服务正常返回。接下来我们再一次成功访问了主页，说明服务完全运转正常。

```bash
$ curl "http://localhost:8080/api/speak?name=geek"
hello geek, you are at /api/speak

$ curl "http://localhost:8080/panic" 
{"message":"Internal Server Error"}

$ curl "http://localhost:8080/api/speak?name=geek"
hello geek, you are at /api/speak
```

我们可以在后台日志中看到如下内容，引发错误的原因和堆栈信息都被打印了出来，通过日志，我们可以很容易地知道，在`gee-web/main.go: 33` (下方第6行)的地方出现了 `index out of range [100] with length 1` 错误。

```bash
2020/08/18 00:37:39 [200] /api/speak?name=geek in 19.016µs
2020/08/18 00:37:42 runtime error: index out of range [100] with length 1
TraceBack: 
        /usr/local/Cellar/go/1.13.3/libexec/src/runtime/panic.go: 680
        /usr/local/Cellar/go/1.13.3/libexec/src/runtime/panic.go: 75
        /Users/Code/goProject/gee-web/main.go: 33
        /Users/Code/goProject/gee-web/gee/context.go: 45
        /Users/Code/goProject/gee-web/gee/recovery.go: 21
        /Users/Code/goProject/gee-web/gee/context.go: 45
        /Users/Code/goProject/gee-web/gee/middlewares.go: 16
        /Users/Code/goProject/gee-web/gee/context.go: 45
        /Users/Code/goProject/gee-web/gee/router.go: 98
        /Users/Code/goProject/gee-web/gee/gee.go: 100
        /usr/local/Cellar/go/1.13.3/libexec/src/net/http/server.go: 2803
        /usr/local/Cellar/go/1.13.3/libexec/src/net/http/server.go: 1891
        /usr/local/Cellar/go/1.13.3/libexec/src/runtime/asm_amd64.s: 1358

2020/08/18 00:37:42 [500] /panic in 248.468µs
2020/08/18 00:37:44 [200] /api/speak?name=geek in 8.612µs
```

## 参考

- [Package runtime - golang.org](https://golang.org/pkg/runtime/)
- [Is it possible get information about caller function in Golang? - StackOverflow](https://stackoverflow.com/questions/35212985/is-it-possible-get-information-about-caller-function-in-golang)


<div class="jump">
	<a href="#/./project/gee-6">Previous</a>
	<a href="#/./docs/go-web">目录</a>
	<a href="#/./project/gee-summary">Next</a>
</div>

---

> 本文改自【极客兔兔】博文：https://geektutu.com/post/gee-day7.html
