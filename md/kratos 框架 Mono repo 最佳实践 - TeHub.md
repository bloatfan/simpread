> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [tehub.com](https://tehub.com/a/ayPvBvv2p0)

> Mono repo 是什么 mono-repo 是一种越来越流行的项目管理方式，与之相对的叫 multi-repo。

Mono repo 是什么
-------------

mono-repo 是一种越来越流行的项目管理方式，与之相对的叫 multi-repo。multi-repo 就是多个仓库，git submodule 其实也是 multi-repo 的一种方式，主项目和子项目都是单独的 git 仓库，也就构成了多个仓库。而 mono-repo 就是一个大仓库，多个项目都放在一个 git 仓库里面。现在很多知名开源项目都是采用的 mono-repo 的组织方式，比如 Babel，React ,Jest, create-react-app, react-router 等等。mono-repo 特别适合联系紧密的多个项目，比如微服务，下面我们就进入本文的主题，认真看下 mono-repo。

### 调整 kratos 项目结构

创建 kratos 项目后，我们可以看到项目结构如下

```
application
|____api
| |____helloworld
| | |____v1
|____cmd
| |____helloworld
|____configs
|____internal
| |____conf
| |____data
| |____biz
| |____service
| |____server
|____test
|____third_party
|____pkg
|____go.mod
|____go.sum
|____LICENSE
|____README.md
```

当我们使用 Mono-repo 大仓库，项目结构调整为如下结构

> tips: kratos 官方没有提供生成 mono repo 项目的命令，所以需要我们手动调整。

```
application
|____api
| |____user
| | |____service
| | | |____v1
|____app
| |____user
| | |____admin
| | |____interface
| | |____job
| | |____service
| | | |____cmd
| | | |____configs
| | | |____internal
| | | | |____conf
| | | | |____data
| | | | |____biz
| | | | |____service
| | | | |____server
| | | | |____test
|____third_party
|____third_party
|____pkg
|____go.mod
|____go.sum
|____Makefile
|____LICENSE
|____README.md
```

重点：应用类型目录
---------

kratos 把微服务中的 app 服务类型主要分为 5 类：interface、service、job、admin、task，，应用 cmd 目录负责程序的：启动、关闭、配置初始化等。

`app/user/`下面的一级目录就是应用类型目录

*   interface: 对外的 BFF 服务，接受来自用户的请求，比如暴露了 HTTP/gRPC 接口。
*   service: 对内的微服务，仅接受来自内部其他服务或者网关的请求，比如暴露了 gRPC 接口只对内服务。
*   admin：区别于 service，更多是面向运营测的服务，通常数据权限更高，隔离带来更好的代码级别安全。
*   job: 流式任务处理的服务，上游一般依赖 message broker。
*   task: 定时任务，类似 cronjob，部署到 task 托管平台中。

Mono repo 开发流程
--------------

### 往 mono repo 添加一个服务

```
// 新建一个kratos服务
kratos new app/demo/service --nomod

// 我们也可以将生成的服务指定到合适的服务类型目录下面，比如
kratos new app/demo/interface --nomod
```

### 生成编译服务 api proto 文件

```
// 生成
kratos proto add api/demo/service/v1/demo.proto 
// 编译
kratos proto client api/demo/service/v1/demo.proto
```

### 生成 service 代码（也就是 mvc 中的 controller）

```
kratos proto server api/demo/service/v1/demo.proto -t app/demo/service/internal/service
```

执行完上面的几条命令，kratos cli 能帮我们做的事情基本就结束了

最后我们需要手动调整我们的项目代码了 比如：

```
// 删除默认生成的service
rm app/demo/service/internal/service/greeter.go

// 实现我们自己的biz逻辑
vim app/demo/service/internal/biz/demo.go

// more...
```

参考链接
----

kratos 官网：[https://go-kratos.dev/docs/](https://go-kratos.dev/docs/)

微服务电商示例项目：[https://github.com/go-kratos/beer-shop](https://github.com/go-kratos/beer-shop)

[https://github.com/wjs1152283574/kratos-mono-repo](https://github.com/wjs1152283574/kratos-mono-repo)

[https://github.com/go-kratos/kratos/issues/1708](https://github.com/go-kratos/kratos/issues/1708)