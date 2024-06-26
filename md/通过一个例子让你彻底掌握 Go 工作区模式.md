> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [polarisxu.studygolang.com](https://polarisxu.studygolang.com/posts/go/workspace/)

> 大家好，我是 polarisxu。

大家好，我是 polarisxu。

早在 Go1.18 快要发布时，我就试用了工作区（workspace）模式，并写了一篇介绍文章：[Go1.18 快讯：Module 工作区模式](https://polarisxu.studygolang.com/posts/go/dynamic/go1.18-workspace/) 。

然而，Go1.18 正式发布后，工作区模式有点变化，导致那篇文章的部分内容不试用了，于是今天重新补一篇，因为工作区模式真的很有用。

工作区模式（Workspace mode），可不是之前 GOPATH 时代的 Workspace，而是希望在本地开发时支持多 Module。

01 缘起[](#01-缘起)
---------------

为了大家全面理解工作区模式，通过一个具体例子讲解。

本地有两个项目，分别是两个 module：mypkg 和 example。（Windows 系统请按自己方式创建目录）

```
$ cd ~/
$ mkdir polarisxu
$ cd polarisxu
$ mkdir mypkg example
$ cd mypkg
$ go mod init github.com/polaris1119/mypkg
$ touch bar.go
```

在 bar.go 中增加如下示例代码：

```
package mypkg

func Bar() {
    println("This is package mypkg")
}
```

接着，在 example 模块中处理：

```
$ cd ~/polarisxu/example
$ go mod init github.com/polaris1119/example
$ touch main.go
```

在 main.go 中增加如下内容：

```
package main

import (
    "github.com/polaris1119/mypkg"
)

func main() {
    mypkg.Bar()
}
```

这时候，如果我们运行 go mod tidy，肯定会报错，因为我们的 mypkg 包根本没有提交到 github 上，肯定找不到。

```
$ go mod tidy
....
fatal: repository 'https://github.com/polaris1119/mypkg/' not found
```

`go run main.go` 也就不成功。

我们当然可以提交 mypkg 到 github，但我们每修改一次 mypkg，就需要提交（而且每次提交之后需要在 example 中 go get 最新版本），否则 example 中就没法使用上最新的。

针对这种情况，目前是建议通过 replace 来解决，即在 example 中的 go.mod 增加如下 replace：（v1.0.0 根据具体情况修改，还未提交，可以使用 v1.0.0）

```
module github.com/polaris1119/example

go 1.19

require github.com/polaris1119/mypkg v1.0.0

replace github.com/polaris1119/mypkg => ../mypkg
```

再次运行 go run main.go，输出如下：

```
$ go run main.go
This is package mypkg
```

当都开发完成时，我们需要手动删除 replace，并执行 go mod tidy 后提交，否则别人使用就报错了。

这还是挺不方便的，如果本地有多个 module，每一个都得这么处理。

02 工作区模式[](#02-工作区模式)
---------------------

针对上面的这个问题，Michael Matloob 提出了 Workspace Mode（工作区模式）。相关 issue 讨论：[cmd/go: add a workspace mode](https://github.com/golang/go/issues/45713) ，[这里是 Proposal](https://go.googlesource.com/proposal/+/master/design/45713-workspace.md) 。并在 Go1.18 中发布了。

因此，要使用工作区，请确保 Go 版本在 1.18+。

我本地当前版本：

```
$ go version
go version go1.19.2 darwin/arm64
```

通过 go help work 可以看到 work 相关命令：

```
$ go help work
Work provides access to operations on workspaces.

Note that support for workspaces is built into many other commands, not
just 'go work'.

See 'go help modules' for information about Go's module system of which
workspaces are a part.

See https://go.dev/ref/mod#workspaces for an in-depth reference on
workspaces.

See https://go.dev/doc/tutorial/workspaces for an introductory
tutorial on workspaces.

A workspace is specified by a go.work file that specifies a set of
module directories with the "use" directive. These modules are used as
root modules by the go command for builds and related operations.  A
workspace that does not specify modules to be used cannot be used to do
builds from local modules.

go.work files are line-oriented. Each line holds a single directive,
made up of a keyword followed by arguments. For example:

    go 1.18

    use ../foo/bar
    use ./baz

    replace example.com/foo v1.2.3 => example.com/bar v1.4.5

The leading keyword can be factored out of adjacent lines to create a block,
like in Go imports.

    use (
      ../foo/bar
      ./baz
    )

The use directive specifies a module to be included in the workspace's
set of main modules. The argument to the use directive is the directory
containing the module's go.mod file.

The go directive specifies the version of Go the file was written at. It
is possible there may be future changes in the semantics of workspaces
that could be controlled by this version, but for now the version
specified has no effect.

The replace directive has the same syntax as the replace directive in a
go.mod file and takes precedence over replaces in go.mod files.  It is
primarily intended to override conflicting replaces in different workspace
modules.

To determine whether the go command is operating in workspace mode, use
the "go env GOWORK" command. This will specify the workspace file being
used.

Usage:

    go work <command> [arguments]

The commands are:

    edit        edit go.work from tools or scripts
    init        initialize workspace file
    sync        sync workspace build list to modules
    use         add modules to workspace file

Use "go help work <command>" for more information about a command.
```

_注意_：上文中提到，工作区不只是 go work 相关命令，Go 其他命令也会涉及工作区内容，比如 go run、go build 等。

根据这个提示，我们初始化 workspace：

```
$ cd ~/polarisxu
$ go work init mypkg example
$ tree
.
├── example
│   ├── go.mod
│   └── main.go
├── go.work
└── mypkg
    ├── bar.go
    └── go.mod
```

注意几点：

*   多个子模块应该在一个目录下（或其子目录）。比如这里的 polarisxu 目录；（这不是必须的，但更好管理，否则 go work init 需要提供正确的子模块路径）
*   go work init 需要在 polarisxu 目录执行；
*   go work init 之后跟上需要本地开发的子模块目录名；

打开 go.work 看看长什么样：

```
go 1.19

use (
    ./example
    ./mypkg
)
```

go.work 文件的语法和 go.mod 类似（go.work 优先级高于 go.mod），因此也支持 replace。

注意：实际项目中，多个模块之间可能还依赖其他模块，建议在 go.work 所在目录执行 `go work sync`。

现在，我们将 example/go.mod 中的 replace 语句删除，再次执行 go run main.go（在 example 目录下），得到了正常的输出。也可以在 polarisxu 目录下，这么运行：go run example/main.go，也能正常。

注意，go.work 不需要提交到 Git 中，因为它只是你本地开发使用的。

当你开发完成，应该先提交 mypkg 包到 GitHub，然后在 example 下面执行 go get：

```
$ go get -u github.com/polaris1119/mypkg@latest
```

然后禁用 workspace（通过 GOWORK=off 禁用），再次运行 example 模块，是否正确：

```
$ cd ~/polarisxu/example
$ GOWORK=off go run main.go
```

目前 VSCode 的 go 插件已经支持 workspace，不需要做什么配置就可以愉快的玩耍。

03 总结[](#03-总结)
---------------

在 GOPATH 年代，多 GOPATH 是一个头疼的问题。当时没有很好的解决，Module 就出现了，多 GOPATH 问题因此消失。但多 Module 问题随之出现。Workspace 方案较好的解决了这个问题。