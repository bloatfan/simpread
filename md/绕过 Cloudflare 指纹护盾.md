> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sxyz.blog](https://sxyz.blog/bypass-cloudflare-shield/)

> 最近才知道，除了 TLS 指纹，竟然还有 HTTP/2 指纹，这两种 Cloudflare 都有采用，这篇博客介绍如何绕过它们。

[Home](https://sxyz.blog/)[Archives](https://sxyz.blog/archives/)[Tags](https://sxyz.blog/tags/)[Links](https://sxyz.blog/links/)[About](https://sxyz.blog/about/)

04-22 bypass-cloudflare-shield

最近才知道，除了 TLS 指纹，竟然还有 HTTP/2 指纹，这两种 Cloudflare 都有采用，这篇博客介绍如何绕过它们。

起因
--

最近发现之前写的搜图 Bot 坏掉了，这个 Bot 接入了 3 个搜索后端，出问题的是 [ascii2d.net](https://ascii2d.net/)。由于它最近套上了 Cloudflare，所有请求都要经过护盾验证。

```
func TestCF(t *testing.T) {
	req, _ := http.NewRequest("GET", "https://ascii2d.net", nil)
	resp, _ := http.DefaultClient.Do(req)

	body, _ := io.ReadAll(resp.Body)
	log.Println(string(body))
}
```

为了方便叙述，这里使用 Go 内置的 `http`，常规项目可以使用 [resty](https://github.com/go-resty/resty) 这种进一步封装的 HTTP Client。

以上代码发出一个 `GET` 请求，并得到如下响应：

```
<!DOCTYPE html>
<html lang="en-US">
  <head>
    <title>Just a moment...</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
</html>
```

这里只截取了部分响应，当出现 `Just a moment...` 就说明已经被 Cloudflare 拦住了。

调查
--

被拦截时，按照一个攻城狮的思维惯性，就是要想办法破掉它。正当我准备实施，打开浏览器一探究竟时，出现了诡异的一幕：浏览器没有盾！

我立刻意识到这可能和 HTTP Client 的 fingerprint 有关。于是我决定先从简单的 request headers 入手：

```
func TestCF(t *testing.T) {
	req, _ := http.NewRequest("GET", "https://ascii2d.net", nil)
	req.Header.Set("accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7")
	req.Header.Set("accept-language", "en,zh-CN;q=0.9,zh;q=0.8")
	req.Header.Set("user-agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36")
	// ...

	resp, _ := http.DefaultClient.Do(req)
	body, _ := io.ReadAll(resp.Body)
	log.Println(string(body))
}
```

没错，就是让这些 headers 尽量接近于真实浏览器。但最终发现这并没有起效，那么可以合理推测并不是 application layer 的问题，那就只剩 session layer 了，也就是 TLS。

分析
--

接下来，通过 Wireshark 分别抓包 Go 程序和浏览器各自的 TLS Client Hello，并比对其差异。这里由于只需要查看 Client Hello 握手时的信息，无须进一步解密其更上层 HTTP 协议栈的数据，因此抓起来很容易：

*   Wireshark 选定适当的网卡，并应用 `tls.handshake.extensions_server_name contains "ascii2d.net"` 过滤器
*   运行 Go 测试，发出请求
*   启动浏览器，导航到 [ascii2d.net](https://ascii2d.net/)
*   分析两个包的区别

经过反复比对、测试，发现与 Cipher Suites 有关 —— Cloudflare 通过包含的密码套件及其顺序识别 HTTP Client 的类型：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/go-cipher-suites.png)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/chrome-cipher-suites.png)

于是解决问题的办法就显而易见了：将 Go HTTP Client 的 Cipher Suites 伪造成浏览器。

但坑的是，Go 官方为了所谓的 “安全性”，是[不允许用户修改密码套件](https://github.com/golang/go/issues/29349)的。他们的理由是，如果你不是密码专家，可能会选择已不安全的密码套件，或是对其错误地配置。

因此 Go 为开发者选择当下最佳的密码套件组合，并强迫开发者为此买单。不过，可能也正因如此，出现了像 [uTLS](https://github.com/refraction-networking/utls) 这样优秀的库，它提供最大的可配置性，并将 Go 的 TLS 完全替换，以帮助人们对抗流量审查，这在其它语言社区是很少见的。

uTLS
----

这里就使用 uTLS 伪造浏览器指纹。安装它：

```
go get -u github.com/refraction-networking/utls
```

再而需要修改一下前面的测试文件：

```
func TestCF(t *testing.T) {
	req, _ := http.NewRequest("GET", "https://ascii2d.net", nil)
	resp, _ := newClient().Do(req)

	body, _ := io.ReadAll(resp.Body)
	log.Println(string(body))
}
```

这里使用 `newClient()` 创建一个新的 HTTP Client，而不再使用 Go 的默认 Client：

```
func newClient(t http.RoundTripper) *http.Client {
	return &http.Client{
		Transport: &uTransport{
			tr1: &http.Transport{},
			tr2: &http2.Transport{},
		},
		// ...
	}
}
```

最后使用我们稍后实现的 `uTransport` 完全替换掉 HTTP Client 的默认 `Transport`。

其实，除手动实现 `Transport` 外，也可以使用 [ja3transport](https://github.com/CUCyber/ja3transport)，它的主要工作是将 JA3 配置字符转换为 uTLS 的配置结构。

JA3 字符则可以在 Wireshark 的 Client Hello 包中复制得到，类似于下面这样：

```
771,4865-4866-4867-49196-49195-49188-49187-49162-49161-52393-49200-49199-49192-49191-49172-49171-52392-157-156-61-60-53-47-49160-49170-10,65281-0-23-13-5-18-16-11-51-45-43-10-21,29-23-24-25,0
```

其中包含 Client Hello 握手时所携带的全部信息。但这里不使用 ja3transport，因为它已经很久没维护了，部分新的 uTLS 扩展字段它并不支持。因此下面我们手动实现基于 uTLS 的 Transport。

伪造 TLS 指纹
---------

先定义 `uTransport` 结构，`http.Client.Transport` 需要满足 `http.RoundTripper` 接口，即必须实现其唯一的方法 `RoundTrip`，签名为 `RoundTrip(*Request) (*Response, error)`：

```
import tls "github.com/refraction-networking/utls"

type uTransport struct {
	tr1 *http.Transport
	tr2 *http2.Transport
}

func (u *uTransport) RoundTrip(req *http.Request) (*http.Response, error) {
	if req.URL.Scheme == "http" {
		return u.tr1.RoundTrip(req)
	} else if req.URL.Scheme != "https" {
		return nil, fmt.Errorf("unsupported scheme: %s", req.URL.Scheme)
	}

	port := req.URL.Port()
	if port == "" {
		port = "443"
	}

	// TCP connection
	conn, err := net.DialTimeout("tcp", fmt.Sprintf("%s:%s", req.URL.Hostname(), port), 10*time.Second)
	if err != nil {
		return nil, fmt.Errorf("net.DialTimeout error: %+v", err)
	}

	// TLS connection
	uConn := tls.UClient(conn, &tls.Config{ServerName: req.URL.Hostname()}, tls.HelloCustom)
	if err = uConn.ApplyPreset(u.newSpec()); err != nil {
		return nil, fmt.Errorf("uConn.ApplyPreset() error: %+v", err)
	}
	if err = uConn.Handshake(); err != nil {
		return nil, fmt.Errorf("uConn.Handshake() error: %+v", err)
	}

	alpn := uConn.ConnectionState().NegotiatedProtocol
	switch alpn {
	case "h2":
		req.Proto = "HTTP/2.0"
		req.ProtoMajor = 2
		req.ProtoMinor = 0

		if c, err := u.tr2.NewClientConn(uConn); err == nil {
			return c.RoundTrip(req)
		} else {
			return nil, fmt.Errorf("http2.Transport.NewClientConn() error: %+v", err)
		}

	case "http/1.1", "":
		req.Proto = "HTTP/1.1"
		req.ProtoMajor = 1
		req.ProtoMinor = 1

		if err := req.Write(uConn); err == nil {
			return http.ReadResponse(bufio.NewReader(uConn), req)
		} else {
			return nil, fmt.Errorf("http.Request.Write() error: %+v", err)
		}

	default:
		return nil, fmt.Errorf("unsupported ALPN: %v", alpn)
	}
}
```

其中 `RoundTrip` 做了如下工作：

*   如果是 HTTP，则直接 fallback 到 http.Transport 的`RoundTrip`，因为它与 TLS 不相干
*   如果是 HTTPS，则建立一个新的 TCP 连接，并基于此创建一个新的 TLS 会话，最后按照 ALPN 协商结果，处理不同的 HTTP 版本

最后，实现 `newSpec()` 方法，它创建一个新的 `ClientHelloSpec` uTLS 配置对象：

```
func (*uTransport) newSpec() *tls.ClientHelloSpec {
	return &tls.ClientHelloSpec{
		TLSVersMax:         tls.VersionTLS13,
		TLSVersMin:         tls.VersionTLS12,
		CipherSuites:       []uint16{tls.GREASE_PLACEHOLDER, 0x1301, 0x1302, 0x1303, 0xc02b, 0xc02f, 0xc02c, 0xc030, 0xcca9, 0xcca8, 0xc013, 0xc014, 0x009c, 0x009d, 0x002f, 0x0035},
		CompressionMethods: []uint8{0x0}, // no compression
		Extensions: []tls.TLSExtension{
			&tls.UtlsGREASEExtension{},
			&tls.SNIExtension{},
			&tls.UtlsExtendedMasterSecretExtension{},
			&tls.RenegotiationInfoExtension{},
			&tls.SupportedCurvesExtension{Curves: []tls.CurveID{tls.GREASE_PLACEHOLDER, tls.X25519, tls.CurveP256, tls.CurveP384}},
			&tls.SupportedPointsExtension{SupportedPoints: []byte{0x0}}, // uncompressed
			&tls.SessionTicketExtension{},
			&tls.ALPNExtension{AlpnProtocols: []string{"http/1.1"}},
			&tls.StatusRequestExtension{},
			&tls.SignatureAlgorithmsExtension{SupportedSignatureAlgorithms: []tls.SignatureScheme{0x0403, 0x0804, 0x0401, 0x0503, 0x0805, 0x0501, 0x0806, 0x0601}},
			&tls.SCTExtension{},
			&tls.KeyShareExtension{KeyShares: []tls.KeyShare{
				{Group: tls.CurveID(tls.GREASE_PLACEHOLDER), Data: []byte{0}},
				{Group: tls.X25519},
			}},
			&tls.PSKKeyExchangeModesExtension{Modes: []uint8{tls.PskModeDHE}}, // pskModeDHE
			&tls.SupportedVersionsExtension{Versions: []uint16{tls.GREASE_PLACEHOLDER, tls.VersionTLS13, tls.VersionTLS12}},
			&tls.UtlsCompressCertExtension{Algorithms: []tls.CertCompressionAlgo{tls.CertCompressionBrotli}},
			&tls.ApplicationSettingsExtension{SupportedProtocols: []string{"h2"}},
			&tls.UtlsGREASEExtension{},
			&tls.UtlsPaddingExtension{GetPaddingLen: tls.BoringPaddingStyle},
		},
		GetSessionID: nil,
	}
}
```

这堆配置照着 Wireshark 抓到的 Client Hello 配就好了，这里是 Google Chrome 112.0.5615.137。另外需要注意，每个 `ClientHelloSpec` 只能被使用一次，因此需要每次方法调用时创建一个新的。

现在运行 `TestCF` 测试，发现一切都正常了：

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta
      content="width=device-width,initial-scale=1.0,minimum-scale=1.0"
      
    />
    <title>二次元画像詳細検索</title>
  </head>
</html>
```

Akamai fingerprint
------------------

你可能会发现，`AlpnProtocols` 怎么只配置了 `[]string{"http/1.1"}`，试着将其补全：

```
&tls.ALPNExtension{AlpnProtocols: []string{"h2", "http/1.1"}},
```

然后再次运行测试，你会发现，我们再次回到了原点：

```
<!DOCTYPE html>
<html lang="en-US">
  <head>
    <title>Just a moment...</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
</html>
```

这意味着不支持 HTTP/2？事实是，前面我们仅仅伪造了 TLS 指纹，但当使用 HTTP/2 over TLS 时，Cloudflare 还会再多检查一个 HTTP/2 的指纹 —— 通常被称为 akamai fingerprint，它长得像这样：

```
// go 1.20.3
fingerprint: 2:0,4:4194304,6:10485760|1073741824|0|a,m,p,s
hash (MD5): 55541b174e8a8adc32544ca36c6fd053
```

你可以在 [https://tls.peet.ws/api/all](https://tls.peet.ws/api/all) 查看你的浏览器 HTTP/2 指纹。这里简单解释一下它的生成方式：

*   对 HTTP/2 中的 4 种 frame 采样，分别是：SETTINGS、WINDOW_UPDATE、PRIORITY、HEADERS，它们间使用 “|” 隔开，即：
    
    ```
    SETTINGS|WINDOW_UPDATE|PRIORITY|HEADERS
    ```
    
*   SETTINGS 部分：对发送的 settings 编码，格式 `KEY:VALUE`，多个逗号隔开。其中 `HEADER_TABLE_SIZE` 的 `KEY=1`、`ENABLE_PUSH` 的 `KEY=2`、`MAX_CONCURRENT_STREAMS`、`INITIAL_WINDOW_SIZE`、`MAX_FRAME_SIZE`、`MAX_HEADER_LIST_SIZE` 以此类推。
    
    如同时发送 `ENABLE_PUSH=0`、`INITIAL_WINDOW_SIZE=4194304`，则编码为 `2:0,4:4194304`
    
*   WINDOW_UPDATE 部分：若存在该 frame，则编码为 `increment` 值，否则 `00`
    
*   PRIORITY 部分：若存在该 frame，则编码为 `stream:exclusive:dependsOn:weight`，否则 `0`
    
*   HEADERS 部分：以 `:` 开头的 header 的第一个字符参与编码，多个逗号隔开。如 `:method`、`:scheme` 编码为 `m,s`
    

更多细节可参见 [Passive Fingerprinting of HTTP/2 Clients](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shuster-Passive-Fingerprinting-Of-HTTP2-Clients-wp.pdf)。

伪造 HTTP/2 指纹
------------

上面介绍了 HTTP/2 指纹的生成方式，于是我们可以发现，伪造其实非常简单，只需要修改其中某个参数，让生成的 Hash 不在 Cloudflare 数据库中即可，如：

```
func newClient(t http.RoundTripper) *http.Client {
	return &http.Client{
		Transport: &uTransport{
			tr1: &http.Transport{},
			tr2: &http2.Transport{
				MaxDecoderHeaderTableSize: 1 << 16,  // this line added
			},
		},
		// ...
	}
}
```

当然，为了进一步地浑水摸鱼你可以伪造更多参数，乃至于最终和浏览器拥有相同的 Hash。然后再跑一次测试：

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta
      content="width=device-width,initial-scale=1.0,minimum-scale=1.0"
      
    />
    <title>二次元画像詳細検索</title>
  </head>
</html>
```

完美！现在成功支持了 HTTP/2。

结尾
--

最后顺带提下，上面的 `uTransport` 为最简实现，还缺乏许多功能，如连接池管理，若拿来使用需酌情完善。

作者由可爱[智子](https://sxzz.moe/)强力驱动，提供精神鼓励与保障 😸

*   [Golang](https://sxyz.blog/tags/golang)