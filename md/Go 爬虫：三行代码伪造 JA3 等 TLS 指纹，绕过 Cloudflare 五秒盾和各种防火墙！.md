> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.skyju.cc](https://blog.skyju.cc/post/tls-fingerprint-bypass-cloudflare/)

> 探索如何使用 Go 语言伪造 TLS 指纹，绕过 Cloudflare 五秒盾和其他防火墙。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1JF4WrnZvt.png)

[代码](https://blog.skyju.cc/categories/code/)

[Go 爬虫：三行代码伪造 JA3 等 TLS 指纹，绕过 Cloudflare 五秒盾和各种防火墙！](https://blog.skyju.cc/post/tls-fingerprint-bypass-cloudflare/)
-------------------------------------------------------------------------------------------------------------------

### 探索如何使用 Go 语言伪造 TLS 指纹，绕过 Cloudflare 五秒盾和其他防火墙。本文详细介绍了 JA3、JA4+ 等 TLS 指纹技术，分析了互联网上已有解决方案实现都太过繁琐、而且不支持用于第三方 HTTP 请求库的不足，并提供了一个简单有效的解决方案：实现一个自定义的 http.RoundTripper。相关代码已经开源在 GitHub 上。

先承认，写这个标题多少有点营销号那味，因为代码里多几个换行就超过三行代码了，而且也不一定能完美绕过所有防火墙及其以后的种种升级版本。但我还是想小小的骄傲一下，本文介绍的方法应该是目前市面上用起来最简单的，并且兼容性最好的（大概）。

Cloudflare 的五秒盾大家应该都很熟悉，对于网站主来说，使用它可以有效防止 CC 攻击；对于爬虫来说，解决它也算是一个重要的课题。而对于既当网站主又当爬虫开发者的我来说，就只能对其又爱又恨了……

要说到爬虫绕过防火墙的其中一个常见方法，应该得属修改 User-Agent。你用 Python、Go 等写的程序，发起 HTTP 请求时，如果不去额外指定，都有自己的 User-Agent。比如在 Go 里使用 [resty](https://github.com/go-resty/resty) 这个库的默认 UA 是`go-resty/2.10.0 (https://github.com/go-resty/resty)`，巴不得把你在用爬虫访问人家网站的事实昭告天下。因此我们一般会修改这个值，将其改成常见浏览器的 UA，例如 Chrome 的：`Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36`。这在以前就足以对付大部分防火墙了。

但是从今年开始 Cloudflare 的防火墙增加了一个识别项，也就是 TLS 的 JA3、JA4 等指纹。如果使用 HTTP/2 访问，还会再多检测一个 Akamai Fingerprint。这些是啥？拿一个典型的 HTTPS Client Hello 的包举例子：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/gvrfDYHZE4.png)

看到里面的 Cipher Suites 和 Extensions 这堆东西了吗？尤其是 Cipher Suites 这个 Array，不同浏览器的数量、种类、顺序都是不一样的。于是网络安全的研究者就根据这个计算了一个哈希，把它作为一个 HTTPS 客户端的 TLS 指纹。

你用 Go 写的爬虫，建立 TLS 握手的时候使用的是 Go 的库，自然也有 Go 自己的 TLS 指纹。那么 Cloudflare 可以直接把 Go 的 TLS 指纹给 ban 掉，只允许真实浏览器访问。甚至来说，实际上 Cloudflare 会把 TLS 指纹和你的 User-Agent 进行比对。人家看到你 User-Agent 宣称是谷歌浏览器的，而 TLS 指纹却是 Go 库的，这不赤裸裸的欺骗吗？绝对要把你 block 之门外了。

关于 JA3、JA4+ 和 Akamai Fingerprint 可以看下面几个文章：

介绍 JA3 的：[https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967/](https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967/)

介绍 JA4+ 的：[https://blog.foxio.io/ja4-network-fingerprinting-9376fe9ca637](https://blog.foxio.io/ja4-network-fingerprinting-9376fe9ca637)

JA4 相当于是 JA3 的升级版。介绍 JA4+ 的这篇文章尤其让我大开眼界。研究者不仅只是算出一个 TLS 指纹而已，还根据不同的因素创造出了多种指纹的变种（因此后面多了个加号）。比如 JA4SSH，就是针对 SSH 协议的。通过对指纹的分析，可以识别加密信道中是否存在恶意流量，本身还是很不错的。

而 Akamai Fingerprint 就是针对 HTTP/2 的，其简单的原理介绍可以看这里：[https://lwthiker.com/networks/2022/06/17/http2-fingerprinting.html](https://lwthiker.com/networks/2022/06/17/http2-fingerprinting.html)

可以访问这个网站看到你浏览器的 TLS 指纹：[https://tls.peet.ws/](https://tls.peet.ws/)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/PDkFykaiEq.png)

高版本 Chrome 加入了一些随机特性，每次访问 JA3 指纹都不一样。但不管你怎么随机，也就那么几种，都带有 Chrome 的特征。而且经过测试我这边 Chrome 浏览器的 JA4 指纹是不变的。

互联网上能找到的解决方案也不算少，都主要围绕 [uTLS](https://github.com/refraction-networking/utls) 这个库来使用。uTLS 库提供对 Go 原生 tls 库的替代，重写了 Client Hello 的过程，能够自定义上面说的 Cipher Suites 和 Extensions 字段等等。

比如我找到的[这篇博文](https://sxyz.blog/bypass-cloudflare-shield/)和[这篇博文](https://blog.csdn.net/chenzhuyu/article/details/132217262)，都是直接裸用 uTLS 库。前者甚至从底层 DialTCP 开始手搓 TLS 加密信道（这篇文章还是很值得一看的，可以帮助你把整个构造的底层原理弄清楚）。这也无可厚非，uTLS 库的缺点就是过于底层了，用起来比较麻烦。

于是我找到了 [tls-client](https://github.com/bogdanfinn/tls-client) 这个库，他在 uTLS 上面封装了一个 HTTP Client，可以直接使用其提供的方法发起请求，并且封装了常见主流浏览器的 profile：

但缺点也是有的，就是提供的接口过于原始了，对于我这样的懒人，早就习惯用 [resty](https://github.com/go-resty/resty)、[req](https://github.com/imroc/req) 这样封装完善的第三方库了。

好在 resty 提供一个 SetTransport 的方法，可以传入一个实现了 http.RoundTripper 的接口。相关的函数签名其实很简单：

```
options := []tls_client.HttpClientOption{
		tls_client.WithTimeoutSeconds(30),
		tls_client.WithClientProfile(profiles.Chrome_120),// Chrome 120 版本的指纹
	}
// ...
// 创建 Request，注意这里创建的 Request 不是 Go 标准库的 http.Request，而是 fhttp.Request，它自己的一个结构
req, err := fhttp.NewRequest(http.MethodGet, "https://tls.peet.ws/api/all", nil)
// ...
resp, err := client.Do(req)// 使用 tls-client 进行请求
// ...
```

无非就是接收到一个 http.Request 对象，然后进行网络请求，最后返回一个 http.Response。注意这里的 Request 和 Response 都是 Go 的 http 标准库中的，而 tls-client 使用了另一套 fhttp.Request 和 fhttp.Response，所以只要进行一下桥接，实现一个自定义 RoundTripper 就行了。

完整代码已经开源：[https://github.com/juzeon/spoofed-round-tripper](https://github.com/juzeon/spoofed-round-tripper)

这样和 resty 一起使用起来就变得很容易：

注意有一些用法是不行的，毕竟咱们的 RoundTripper 是自己的实现的，不是 Go 自己的 http.Transport。比如设置代理的时候：

```
type http.RoundTripper interface {
	RoundTrip(*http.Request) (*http.Response, error)
}
```

上面第一种尝试会报错的原因是 resty 的 SetProxy 方法会在内部尝试把传入的 http.RoundTripper 转成标准库的 http.Transport，显然咱们穿进去的是个假的，所以会报错。好在 tls-client 本身提供了设置代理的方法，可以直接用。

说实话，像 Go 这样能直接撅起底层，自己实现 TLS 握手过程的能力，在其他语言中时很少见的。比如 Python 基本上所有的网络请求库在 TLS 握手的时候都是直接调用底层 C 编写的 openssl 套件，因此完全就不支持自定义。搞了半天，Python 这个被鼓吹为最适合爬虫的语言在写爬虫上居然还不如 Go？

参考掘金上的这篇文章：[https://juejin.cn/post/7197740114252447781](https://juejin.cn/post/7197740114252447781)

嗯… 实际上上面介绍的 tls-client 这个库还提供一些其他语言的 binding，不过调用起来相对麻烦，感兴趣的话可以去看看人家的[文档](https://bogdanfinn.gitbook.io/open-source-oasis/shared-library)。

2023.12.29 更新
-------------

[req](https://github.com/imroc/req) 这个库提供一键伪装 HTTP、TLS 指纹的功能，具体可参见文档：[https://req.cool/zh/docs/tutorial/http-fingerprint/](https://req.cool/zh/docs/tutorial/http-fingerprint/)。