> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.sainnhe.dev](https://www.sainnhe.dev/post/best-practice-for-dns-in-clash/)

> 基础 让我们先从官方 Wiki 开始看起。

[](#基础)基础
---------

让我们先从官方 [Wiki](https://github.com/Dreamacro/clash/wiki/configuration) 开始看起。

先来看这样一段配置：

我们来一个一个看。

*   `enable` 字段用于设定是否启用 DNS
*   `listen` 字段用于设定 DNS 服务监听的地址，这里使用了 53 端口，这是 DNS 服务的 [Well-known port](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)，即大多数系统的默认监听端口。
*   `ipv6` 字段用于设定是否启用 IPv6 ，当然启用，为什么不呢？

现在重点来说一下 `default-nameserver` 和 `nameserver`。

`default-nameserver` 是用来解析 `nameserver` 的 DNS 服务，它本身并不会解析你要访问的链接的域名。

什么意思？举个例子，你要访问 `www.google.com`，Clash 会向 `nameserver` 字段中的 DNS 服务地址发送查询请求来查询 `www.google.com` 的 IP 地址，这些请求是并发查询的，谁最先返回查询结果就用谁的结果。

在本例中，Clash 会向 `114.114.114.114`、`https://doh.pub/dns-query` 以及 `tls://dns.alidns.com:853` 这三个 DNS 服务地址发送请求，这些请求都不经过代理，直接发送。这些请求中第一个地址是 Do53 协议，第二个地址是 DoH 协议，第三个地址是 DoT 协议。这三种协议的具体区别会在后面细讲，现在你只需要知道 Do53 协议的 DNS 地址是通过 IP 来确定服务器地址的，而 DoH 和 DoT 通常都是通过域名来确定服务器地址的。

那么问题来了，为了将查询请求发送到 DNS 服务器，我们得知道 DNS 服务器的 IP 地址。Do53 协议的地址 `114.114.114.114` 直接通过 IP 地址来表示，可以通过这个 IP 地址来访问，可是 DoH 协议的地址使用的是域名 `doh.pub` 来表示，DoT 协议的地址使用的是 `dns.alidns.com` 来表示，如果想要访问它们，我们还需要解析 `doh.pub` 和 `dns.alidns.com` 这两个域名的 IP 地址，那谁负责解析这两个域名的 IP 地址呢？

这项工作就是由 `default-nameserver` 字段里的 DNS 服务来完成的，也就是 `114.114.114.114` 和 `8.8.8.8` 这两个。比如当我们要解析 `doh.pub` 这个域名的 IP 地址时，Clash 会向 `114.114.114.114` 和 `8.8.8.8` 同时发送查询请求，谁先返回查询结果就用谁的。

所以，`default-nameserver` 的作用并不是解析你要访问的 `www.google.com`，而是用来解析 `nameserver` 字段中的 DoH 和 DoT 链接的域名。

也正是因为如此，`default-nameserver` 中的 DNS 服务地址只能是 Do53 协议的，也就是只能是 `8.8.8.8` 这样的 IP 地址，而不能是 `https://` 或 `tls://` 这样的 DoH 或 DoT 地址，因为如果是 DoH 或 DoT 地址的话就没有谁可以解析它们的域名了。

现在来简单说一下 Do53、DoH、DoT 协议的具体区别。

Do53 协议地址是用 IP 地址来表示的，比如谷歌的 `8.8.8.8`，Cloudflare 的 `1.1.1.1` 都是 Do53 协议地址。DNS 查询请求直接发送到这个 IP 地址所对应的服务器，服务器递归查询并返回查询结果。

然而 Do53 有个致命问题，那就是**它是明文的**，也就是你的查询请求全部没有经过加密，系统管理员能够看到你的查询请求，供应商能够看到你的查询请求。

这非常危险！

为了解决这个问题，RFC 在 2016 年采纳了 DoT (DNS over TLS) 协议，在 2018 年采纳了 DoH (DNS over HTTPS) 协议，这两个协议都是加密的协议，能够防止中间人看到你的查询请求。

那么为啥已经有了 DoT 还要弄个 DoH 呢？它们有什么区别？

我们知道 HTTPS = HTTP + TLS ，DoH 协议在 TLS 的基础上又套了一层 HTTP 协议，这毫无疑问带来了额外的开销。

但它也有个优点，那就是它使用了 HTTPS 的 443 端口，这就使得流量看起来就像是一般的 HTTPS 流量，从而能够很好地伪装。

而 DoT 则不同，它有自己的专用端口 853 ，很容易被识别进而被封锁。

事实上这种直接封端口的事也确实出现过，有人说他们公司的 IT 管理员为了强制员工使用公司内部 DNS，直接封 53 (Do53 的端口) 和 853 端口，也有人说自己的网络环境下所有 DoT 流量均被阻断，怀疑是运营商搞的鬼。

总结一下就是，DoH 相比于 DoT 能够更好地伪装加密流量，而 DoT 相比于 DoH 开销更小，延迟更低。

那么延迟低多少呢？

我自己在我的服务器上进行了测试，其实 DoT 比 DoH 的延迟也就低个 1% ~ 3% ，但是不论 DoT 还是 DoH 延迟都是 Do53 的 7 ~ 10 倍。详情参考 [DoH vs DoT Benchmark: Choose Your Safe DNS Protocol](https://www.sainnhe.dev/post/doh-vs-dot-benchmark)

所以如果你选择了承受 DoT 和 DoH 协议带来的 7 ~ 10 倍的性能差距，那么就没必要太在乎这 1% ~ 3% 的性能差距了，要用建议直接上 DoH ，别考虑 DoT ，不然什么时候你的流量被阻断了都不知道怎么回事。

### [](#最佳实践)最佳实践

```
dns:
  enable: true
  listen: 0.0.0.0:53
  ipv6: true
  default-nameserver:
    - 114.114.114.114
    - 8.8.8.8
  nameserver:
    - 114.114.114.114
    - https://doh.pub/dns-query
    - tls://dns.alidns.com:853
```

相比于刚才的配置，我们做了两点修改：

第一，我们把 `default-nameserver` 中的 `114.114.114.114` 换成了 `1.1.1.1`。

为啥？因为 `114.114.114.114` 是三大运营商的 DNS ，虽然延迟低，但是有污染，解析结果可能指向错误的 IP 地址。

`1.1.1.1` 是 Cloudflare 的 DNS，`8.8.8.8` 是谷歌的 DNS，都能在国内访问，都能保证无污染，但是延迟较高。**（经评论区指正，这句话有误）**

不过虽然延迟较高，但这两个 DNS 都是放在 `default-nameserver` 字段的，也就是只会影响 `nameserver` 字段中的域名解析结果。当你连上 Clash 后这两个 DNS 只会调用少量几次，所以并不会造成多大影响。

第二点修改是我们把 `nameserver` 字段中的 DNS 服务地址换成了 `https://doh.pub/dns-query` 和 `https://dns.alidns.com/dns-query`，第一个是腾讯的 DNS ，第二个是阿里的 DNS，都是 DoH 协议。

这两个 DNS 在国内访问速度相对较快，但是也有个很大的缺点，就是会有 DNS 污染。我们接下来看看怎么解决这个问题。

[](#分流)分流
---------

解决 DNS 污染的基本思路就是分流，解析国内的域名用阿里和腾讯的 DNS，解析国外的域名用国外的 DNS。

来看这样一段配置：

`fallback-filter` 字段就是我们的分流规则。当匹配分流规则时，就用 `fallback` 字段中的 DNS，如果不匹配分流规则就用 `nameserver` 字段中的 DNS。

这个例子中的分流规则有两个：

*   当解析的域名在 GeoIP 数据库内的国家代码不是 `CN` 时，也就是要解析的域名位于国外时。
*   当 `nameserver` 中的 DNS 解析结果位于 `240.0.0.0/4` 这一 IP 段内时。这种情况通常是解析结果被污染了，因为被污染的解析结果通常就位于这个 IP 段内。

`fallback` 字段内我们加了两个 DoH 地址，`https://cloudflare-dns.com/dns-query` 是 Cloudflare DNS，`https://dns.google/dns-query` 是谷歌 DNS 。

然而当你把这两个加上之后你会发现，并不能用。

事实上几乎所有国外比较有名的公共 DoH 地址都不能用，我测试过以下地址，全部无法连接：

*   Cloudflare: `https://cloudflare-dns.com/dns-query`
*   Google: `https://dns.google/dns-query`
*   Cisco OpenDNS: `https://dns.opendns.com/dns-query`
*   IBM Quad9: `https://dns.quad9.net/dns-query`
*   AdGuard: `https://dns.adguard-dns.com/dns-query`

Clash 在连接 DNS 服务器的时候是不会走代理的，也就是这些 DNS 地址都是直连的，没一个能用。

幸运的是，如果用 HTTPS + IP 的方式来访问的话，有一些服务是可以直连的，以下是我测试过能够直连的 DoH 服务：

*   `https://1.1.1.1/dns-query`: Cloudflare 主 DNS
*   `https://1.0.0.1/dns-query`: Cloudflare 次 DNS
*   `https://1.1.1.2/dns-query`: Cloudflare 主 DNS（过滤恶意网站）
*   `https://1.0.0.2/dns-query`: Cloudflare 次 DNS（过滤恶意网站）
*   `https://1.1.1.3/dns-query`: Cloudflare 主 DNS（过滤成人内容）
*   `https://1.0.0.3/dns-query`: Cloudflare 次 DNS（过滤成人内容）
*   `https://208.67.222.222/dns-query`: Cisco OpenDNS 主 DNS
*   `https://208.67.220.220/dns-query`: Cisco OpenDNS 次 DNS
*   `https://9.9.9.9/dns-query`: IBM Quad9（过滤恶意网站）

### [](#最佳实践-1)最佳实践

```
dns:
  enable: true
  listen: 0.0.0.0:53
  ipv6: true
  default-nameserver:
    - 1.1.1.1
    - 8.8.8.8
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

这套配置使用了三个 DNS，这三个 DNS 都有过滤恶意网站的功能。

当客户端发起一个 DNS 查询请求时，Clash 会并发向这三个解析服务器发送查询请求，谁的响应先到就用谁的。

这套配置看起来已经相当不错了，既用上了无污染的 DoH，又能够过滤恶意网站。

但它还是有一些缺陷：

1.  延迟太高，基本都在 1000 ms 以上
2.  不稳定，说不定什么时候就给你掐断了，毕竟谷歌的 `https://8.8.8.8/dns-query` 已经无法使用了
3.  无法自定义过滤内容

我们接下来看看有没有什么更好的方案。

[](#免费自建-dns)免费自建 DNS
---------------------

接下来我们来自建一个 DNS，这个 DNS 的延迟大约在 300 ~ 500 ms，日常用完全够了，并且这个 DNS 还能自定义过滤内容，包括广告 / 追踪器 / 恶意网站等。

要自建这个 DNS，你需要一个注册了一周以上的 GitHub 账号，因为如果是刚注册的账号，会被判定为机器人而无法新建服务。

*   Fork [serverless-dns/serverless-dns](https://github.com/serverless-dns/serverless-dns)
*   访问 [Deno Deploy](https://deno.com/deploy) 并用 GitHub 账号注册登陆
*   在 [Dashboard](https://dash.deno.com/) 界面新建一个项目，名字自己取，比如我们命名为 `serverless-dns`
    *   选你刚刚 Fork 的仓库
    *   分支选 main -> GitHub Action
    *   Add Env Variable
        *   `CLOUD_PLATFORM`: `deno-deploy`
        *   `RUNTIME`: `deno`
        *   `DENO_ENV`: `production`
    *   点击 Link
*   回到你 Fork 的 GitHub 仓库的主页，点进 Action 页面，确认启用 GitHub Action
*   点进 Settings -> Security -> Secrets and variables -> Actions -> New repository secret -> 变量名为 `DENO_PROJECT_NAME`，值为 `serverless-dns` 就是第三步中的项目名
*   回到 Action 页面，左侧 Deno -> 右边 Run workflow
    *   Deploy via action or auto: `action`
    *   Git branch / tag / commit (used for auto deploy-mode): `main`
    *   Deploy branch (used for auto deploy-mode): `live`
    *   Run workflow
*   回到 Deno Deploy 界面，访问项目的页面，一般是 `https://<name>.deno.dev`，然后配置过滤器，复制 DoH 链接即可。

Deno Deploy 免费版的限制是：

*   每天 100,000 次请求
*   每月 100 GiB 的出站流量

我自己正常使用 serverless-dns 平均每天 1000 次请求，0.4 MiB 的流量，所以完全够用。

**注意**：如果你要用自定义域名，那么域名里一定不要包含 `dns`，不然流量很容易被阻断。

### [](#最佳实践-2)最佳实践

现在我们把部署好的 DNS 放到 Clash 配置里，大概长这样：

其中 `max.rethinkdns.com` 和 `sky.rethinkdns.com` 是官方部署的公共 DNS，而 `<name>.deno.dev` 是你自己部署的 DNS。

[](#q--a)Q & A
--------------

**不是有了官方部署的公共 DNS 吗？为啥还要自己部署一个？**

因为官方只提供了部署在 Cloudflare Worker 和 Fly.io 上的实例，我自己实测它们的延迟都在 1000 ms 以上。而我们自己在 Deno 上部署的 DNS 延迟可以在 500 ms 以下。这里把官方提供的两个 DNS 放进去是为了做兜底，防止 deno.dev 抽风。如果 deno.dev 抽风了，那么由于 Clash 会使用这三个 DNS 中最先返回的那个结果，此时 Clash 就会使用官方的两个延迟较高的 DNS。

**当你过了一遍 serverless-dns 的官方文档后，你会发现其实还可以部署到 Cloudflare Workers 上，而且好像比 Deno Deploy 更方便，那要不要选 Cloudflare Workers 呢？**

我的建议是不要用 Cloudflare Workers，因为：

*   Cloudflare Workers 自带的域名 `workers.dev` 被墙了，你得用自己的域名，但 `deno.dev` 没被墙，可以直接用
*   我自己实测 Cloudflare Workers 的延迟是 Deno Deploy 的两倍

**既然 serverless-dns 能过滤广告，那我能不能不用 fallback 了，全部换成我自己部署的 serverless-dns？**

我的建议是，如果你在 Clash 配置里加了分流规则，比如 [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules)，那就最好用 `fallback`，否则可能会出现这样的问题：

假设进行了分流，国内的网站直连，国外的网站走代理，但是不管是国内还是国外网站你都用 serverless-dns 来解析域名。

当你访问某个网站时，用 serverless-dns 来解析它的域名，由于 serverless-dns 其实一定程度上是反代 + 过滤，所以你解析的时候上游 DNS 服务器（比如 `8.8.8.8`）不会知道你的真实 IP 地址，而是会根据 Deno Deploy 服务器的 IP 地址选择相应的负载均衡策略，然后解析出来的 IP 地址就是距离 Deno Deploy 服务器最近的 IP 地址（比如美国）。

可是你人在中国，去访问解析出来的美国的 IP 地址，就会出现丢包严重，甚至完全无法访问的情况。

实际上也确实如此，你如果试一试就知道，要是分流 + 全部使用 serverless-dns 会造成很多网页无法正常访问，而原因就是我刚才说的那个，反代导致上游 DNS 服务器选择了错误的负载均衡策略。

所以建议不要这样用。

**腾讯 DNS 貌似也有过滤的功能，并且免费额度看起来够用，要不试试？**

别。

DoT 和 DoH 请求按 8 倍计价，完全不够用。看这个帖子：[DNSPod Public DNS 专业版的用量计算是不是有问题？](https://v2ex.com/t/910840#reply9)