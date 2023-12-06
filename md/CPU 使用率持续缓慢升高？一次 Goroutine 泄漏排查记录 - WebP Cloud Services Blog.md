> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.webp.se](https://blog.webp.se/goroutine-leak-zh/)

> 在上一篇文章「我们是怎么在运行时修改 Go 程序的 LogLevel 的」中，我们讲述了我们的 WebPLB 日志被一个来自于我们自己的控制请求刷屏，并逐渐讲到了我们如何修改了 Loglevel

在上一篇文章「[我们是怎么在运行时修改 Go 程序的 LogLevel 的](https://blog.webp.se/change-loglevel-runtime-zh/)」中，我们讲述了我们的 WebPLB 日志被一个来自于我们自己的控制请求刷屏，并逐渐讲到了我们如何修改了 Loglevel 并可以用 Signal 的方式动态修改的故事。

由于使用了 [Fiber](https://github.com/gofiber/fiber) 框架，默认的 Logger 会给每个请求都输出一份 Log，虽然我们在应用程序上可以改变一些 Log 的 LogLevel，比如：

```
log.Infoln("converting image to sRGB color space")
```

可以改为

```
log.Debugln("converting image to sRGB color space")
```

但是对于请求的 Log 而言，Fiber 还是会记录下来，这里我们就有了一个新的需求：如何在某些请求中，不记录 Log 呢？

比如我们的控制请求是 `/some_kind_of_webp_cloud_command`，于是在某个修改中我们加入了这样的代码：

```
// fiber logger format
app.Use(func(c *fiber.Ctx) error {
    if c.Path() == "/some_kind_of_webp_cloud_command" {
        return c.Next()
    }

    return logger.New(logger.Config{
        Format:     "${ip} - [${time}] ${method} ${url} ${status} ${referer} ${ua}\n",
        TimeFormat: "2006-01-02 15:04:05",
    })(c)
})
```

这样对于特定的请求而言就不会记录 Log 了，通过 `c.Next()` 会跳过 Logger 中间件。

同时我们还 Bump 了一些依赖，比如：

*   `github.com/ClickHouse/clickhouse-go/v2`
*   `github.com/gofiber/fiber/v2`
*   `github.com/valyala/fasthttp`
*   `gorm.io/driver/mysql`

此外，加入了上文中对于 LogLevel 的修改，然后我们就发版部署了。

然后… 我们就看到了我们所有的机器 CPU 都在逐渐升高，例如这是我们德国区域某一台机器的 CPU 负载监控：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/cpu.png)

> 其中每次 CPU 使用率落下来的时候是我们重启了容器（部署了新的版本）。

现在问题来了，为什么会这样？

*   由于我们的某个定时任务（比如清理本地缓存）由于需要遍历文件系统卡住了？（但是 IO Wait 并不高）
*   难道因为 Fiber 或者 ClickHouse 的某个版本有问题？
*   或者监听系统信号的代码有问题导致了死循环？

* * *

有兴趣的读者可以在这里停一下，如果你遇到了类似的问题，应该如何排查？

* * *

首先我们可以公布一下，上面的三个猜想都不对，我们的定时任务会遍历文件系统是真，使用 `fstat` 确实会造成一些 CPU 使用率的提升，但是除非任务之间出现了 Overlap ，不然 CPU 使用率是不会稳步上升的。

为了调研这个问题，我们给 WebPLB 加入了 PProf ，当然，这一步还算比较简单，直接参考 Fiber 代码即可：

```
import (
  "github.com/gofiber/fiber/v2"
  "github.com/gofiber/fiber/v2/middleware/pprof"
)

app.Use(pprof.New())
```

等上线了 PProf 之后我们等了一天等 CPU 使用率上来，并拿到了一份 CPU 的 Profile，内容如下：

```
File: webplb
Build ID: 2d0e21c64238ef4dcfc1e549023659cb8b637004
Type: cpu
Duration: 50.14s, Total samples = 16.47s (32.85%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top20
Showing nodes accounting for 10650ms, 64.66% of 16470ms total
Dropped 227 nodes (cum <= 82.35ms)
Showing top 20 nodes out of 169
      flat  flat%   sum%        cum   cum%
    1670ms 10.14% 10.14%     1710ms 10.38%  runtime.siftdownTimer
    1410ms  8.56% 18.70%     1410ms  8.56%  runtime/internal/syscall.Syscall6
     830ms  5.04% 23.74%      830ms  5.04%  runtime.futex
     740ms  4.49% 28.23%      740ms  4.49%  runtime/internal/atomic.(*Uint32).Load (inline)
     610ms  3.70% 31.94%     5250ms 31.88%  github.com/gofiber/fiber/v2/middleware/logger.New.func1
     610ms  3.70% 35.64%      610ms  3.70%  runtime/internal/atomic.(*Uint32).CompareAndSwap (inline)
     580ms  3.52% 39.16%      590ms  3.58%  runtime.gopark
     510ms  3.10% 42.26%      510ms  3.10%  runtime.nanotime (inline)
     500ms  3.04% 45.29%     1600ms  9.71%  time.Time.appendFormat
     430ms  2.61% 47.91%      430ms  2.61%  sync/atomic.(*Value).Store
     400ms  2.43% 50.33%     3870ms 23.50%  runtime.runtimer
     350ms  2.13% 52.46%      530ms  3.22%  runtime.lock2
     330ms  2.00% 54.46%      330ms  2.00%  [libc.so.6]
```

诶，这就有意思了， `runtime.siftdownTimer` 是什么？为什么 `github.com/gofiber/fiber/v2/middleware/logger.New.func1` 的 cumulative time 这么高？我们来看看调用关系：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/profile.png)

> 这一眼看上去肯定是 Fiber 的 Logger 写出坑了！

于是我们立即对比了一下我们 Bump 版本的 Fiber 和 Clickhouse 对应版本之间代码，但是均没有发现和 Logger 相关的可能和这里改动的部分。

这里我们就有了一个新的想法， `runtime.siftdownTimer` 是什么，为什么它会有这么高的 flat time？

在网上搜了一圈之后可用的信息不多，比如这个帖子： [https://forum.golangbridge.org/t/runtime-siftdowntimer-consuming-60-of-the-cpu/3773](https://forum.golangbridge.org/t/runtime-siftdowntimer-consuming-60-of-the-cpu/3773)

帖子中人有如下神奇的使用 Ticker 的方式：

```
func() {
  for {
    select {
      case <-time.Tick(10*time.Milliseconds):
    }
  }
}
```

而我们的代码中所有的任务用到 Ticker 的地方都会在函数执行完成后销毁，不会出现循环中分配 Ticker 的事情，也不会出现分配了 Ticker 并不使用的问题。

于是我们继续分析 Profile，发现了一个情况，在查看 Go Routine 的时候，发现每次给 WebPLB 发一个新的请求，就会出现一个新的 Go routine，且还是 sleep 状态的，发了一些请求之后，已经出现了 80+ 个 sleep 的 Goroutine：

```
goroutine 85 [sleep]:
time.Sleep(0x1dcd6500)
	/usr/local/go/src/runtime/time.go:195 +0x125
github.com/gofiber/fiber/v2/middleware/logger.New.func1()
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:45 +0x73
created by github.com/gofiber/fiber/v2/middleware/logger.New in goroutine 35
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:43 +0x2c9

...

goroutine 87 [sleep]:
time.Sleep(0x1dcd6500)
	/usr/local/go/src/runtime/time.go:195 +0x125
github.com/gofiber/fiber/v2/middleware/logger.New.func1()
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:45 +0x73
created by github.com/gofiber/fiber/v2/middleware/logger.New in goroutine 35
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:43 +0x2c9
```

于是我们怀疑是不是上面我们针对 `/some_kind_of_webp_cloud_command` 地址的地方写出坑了，我们快速删掉了对应的代码，保留了原有的样式：

```
// fiber logger format
app.Use(logger.New(logger.Config{
    Format:     "${ip} - [${time}] ${method} ${url} ${status} ${referer} ${ua}\n",
    TimeFormat: "2006-01-02 15:04:05",
}))
```

并重新开始 Profile，这个时候我们发现无论我们怎么发请求给 WebPLB，Go routine 的数量都很稳定，与 Logger 相关的 Goroutine 也只在唯一的一个地方出现：

```
goroutine 20 [sleep]:
time.Sleep(0x1dcd6500)
	/usr/local/go/src/runtime/time.go:195 +0x125
github.com/gofiber/fiber/v2/middleware/logger.New.func1()
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:45 +0x73
created by github.com/gofiber/fiber/v2/middleware/logger.New in goroutine 1
	/go/pkg/mod/github.com/gofiber/fiber/v2@v2.51.0/middleware/logger/logger.go:43 +0x2c9
```

好啊，看来果然是这个问题，由于使用中间件的时候每次都会 `return` 一个新的 Logger，导致系统中大量的 Logger Go routine 堆积，久而久之就会导致 CPU 使用率**缓慢且稳定**上升。

让我们来看看这个是谁写的：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/blame.png)

> 或者我怀疑这个代码是 Copilot 补全出来的

但是这个问题我们应该如何解决呢？根据 Fiber 的文档 [https://docs.gofiber.io/api/middleware/logger](https://docs.gofiber.io/api/middleware/logger) ，中间件应该使用 Next 来完成类似的操作:

<table><thead><tr><th>Property</th><th>Type</th><th>Description</th><th>Default</th></tr></thead><tbody><tr><td>Next</td><td><code>func(*fiber.Ctx) bool</code></td><td>Next defines a function to skip this middleware when returned true.</td><td><code>nil</code></td></tr><tr><td>Done</td><td><code>func(*fiber.Ctx, []byte)</code></td><td>Done is a function that is called after the log string for a request is written to Output, and pass the log string as parameter.</td><td><code>nil</code></td></tr></tbody></table>

所以我们将代码改为了如下：

```
app.Use(logger.New(logger.Config{
		Format:     "${ip} - [${time}] ${method} ${url} ${status} ${referer} ${ua}\n",
		TimeFormat: "2006-01-02 15:04:05",
		Next: func(c *fiber.Ctx) bool {
			// Ignore some paths
			ignoredPaths := []string{"/some_kind_of_webp_cloud_healthz", "/some_kind_of_webp_cloud_command"}
			return slices.Contains(ignoredPaths, c.Path())
		},
	}))
```

目前新版本已经上线两天多，CPU 使用率再也没有出现之前的问题了。

WebP Cloud Services 团队是一个来自上海和赫尔辛堡的三人小团队，由于我们不融资，且没有盈利压力 ，所以我们会坚持做我们认为正确的事情，力求在我们的资源和能力允许范围内尽量把事情做到最好， 同时也会在不影响对外提供的服务的情况下整更多的活，并在我们产品上实践各种新奇的东西。