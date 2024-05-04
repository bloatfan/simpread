> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.skyju.cc](https://blog.skyju.cc/post/v2ray-tor-proxy-ip-pool/)

> v2ray 轮询 + Tor 多链路，每次请求都可以拿到一个不同的 IP

![](https://raw.githubusercontent.com/lslz627/PicGo/master/46ff1701edbfd3ad.png)

当我们做以下业务的时候，可能会需要大量代理 IP：

*   HTTP CC/DDOS 网站
*   爬虫，突破反爬
*   基于 IP 判断用户的投票系统，刷票
*   玩 pixelcanvas 类型的像素画网站，用 IP 判断不同用户的
*   ……

互联网上免费代理 IP 的网站很多，自建代理 IP 池的方法也很多。比如下面这个项目：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/3eeef768797621a7.png)

[constverum/ProxyBroker](https://github.com/constverum/ProxyBroker)

包括国内也有一些类似的项目。

但这些方式都有缺点，比如代理 IP 稳定性不可靠，需要后期二次手动校验；国外代理国内访问有困难；匿名性没有很好的保证等等。

今天介绍一种我摸索出来的方法，利用 v2ray+Tor 自建代理 IP 池。你需要有一台自己的 VPS。

debian/ubuntu 可以运行命令：

安装好后，用 nano 或其他编辑器编辑`/etc/tor/torrc`，进行如下编辑（部分配置项）：

请参考 v2ray 的[官方文档](https://www.v2ray.com/)，这里 outbounds 部分配置如下：

```
apt install -y tor
```

outbounds 中配置了一个 tag 为 my-tor 的 socks 出口，其中加入刚才 tor 开的一堆端口。v2ray 会以轮询的方式使用每个端口，这样就可以做到每个连接都用不同的 IP。

routing 部分配置如下：

然后自行配置一个 inbound（这里不列出），我这里的配置是`websocket-over-https`。关于如何配置 inbound，可以参考 [v2ray 白话文教程](https://guide.v2fly.org/)。也可以使用 [v2-ui](https://github.com/sprov065/v2-ui)。

本地我使用 v2rayN 连接 VPS 的 v2ray，开启本地端口是 10808。

我使用以下脚本进行测试：

```
SOCKSPort 38801 #这里开启多个tor端口，对于tor来说，每个端口会使用不同的链路，也就是不同的代理IP
SOCKSPort 38802
SOCKSPort 38803
SOCKSPort 38804
SOCKSPort 38805
SOCKSPort 38806
SOCKSPort 38807
SOCKSPort 38808
SOCKSPort 38809
SOCKSPort 38810

SOCKSPolicy accept 127.0.0.1 #为了安全性，只允许localhost访问tor的端口
SOCKSPolicy reject *

NewCircuitPeriod 30 #对于每个端口来说，每30秒重新创建一个新链路，也就是换一个新IP
CircuitBuildTimeout 10 #对于新建每个链路的过程来说，建立程序超过10秒则直接放弃，保障了连接到线路的质量
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/99c79fdb4e55b908.png)

这样就完成了。你可以增加 tor 开放 socks 端口的数量，也可以缩小`NewCircuitPeriod`的数值，做到尽可能的每次访问都用一个全新的 IP。

### 4.1. VPS 连 v2ray 速度太慢了？

尝试给选用基于 HTTPS 的 v2ray inbound 协议，套一层 Cloudflare CDN。使用 [better-cloudflare-ip](https://github.com/badafans/better-cloudflare-ip) 这个项目自选最快的 cf 节点。

### 4.2. 如何进行 HTTP CC/DDOS 攻击？

由于 v2rayN 开放的本地端口是 socks 协议，而很多 CC/DDOS 软件只支持 HTTP 代理。那么可以安装一个 proxifier，把相应软件的流量强行导向 v2rayN 开放的本地 socks 端口。

### 4.3. 有什么优点？

*   IP 几乎是无限的、并且是高可用的
*   匿名性非常高

### 4.4. 有什么缺点？

*   有些网站禁止 tor 节点的 IP 访问
*   速度不如国内的代理 IP 快
*   请仔细查看你 VPS 提供商的 ToS，需要不禁止你在机子上跑 tor

本文仅提供技术思路，请勿用于非法业务，我对此不负责任。