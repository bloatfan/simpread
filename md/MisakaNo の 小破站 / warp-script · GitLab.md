> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [gitlab.com](https://gitlab.com/Misaka-blog/warp-script)

> CloudFlare WARP 一键部署脚本

[**README.md**](/Misaka-blog/warp-script/-/blob/main/README.md?ref_type=heads)

[](#warp-script)warp-script
===========================

CloudFlare WARP 一键管理脚本

[](#%E4%B8%80%E9%94%AE%E8%84%9A%E6%9C%AC)一键脚本
---------------------------------------------

```
wget -N https://gitlab.com/Misaka-blog/warp-script/-/raw/main/warp.sh && bash warp.sh
```

[](#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)常见问题
---------------------------------------------

### [](#1-%E5%A6%82%E6%9E%9C%E6%88%91%E6%83%B3%E4%BD%BF%E7%94%A8%E5%85%A8%E5%B1%80-ipv4--ipv6%E6%88%96%E6%88%91%E6%83%B3%E5%B0%86-vps-%E7%9A%84%E5%87%BA%E7%AB%99%E4%BA%A4%E7%BB%99-warp-%E8%BF%9B%E8%A1%8C%E4%BB%A3%E7%90%86%E6%88%91%E8%AF%A5%E6%98%AF%E5%AE%89%E8%A3%85-wgcf-%E6%88%96-warp-go)1. 如果我想使用全局 IPv4 / IPv6、或我想将 VPS 的出站交给 WARP 进行代理，我该是安装 WGCF 或 WARP-GO？

WGCF 和 WARP-GO 都是第三方的 CloudFlare WARP 的 Linux 应用程序。由于 WGCF 在香港、美西区域遭到 CloudFlare 的官方限制，故只能使用 WARP-GO

对于没遭到 CloudFlare 封禁的大部分区域的建议：WGCF > WARP-GO

### [](#2-%E5%A6%82%E6%9E%9C%E6%88%91%E4%BB%85%E4%BD%BF%E7%94%A8-socks5-%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E6%88%91%E8%AF%A5%E6%98%AF%E5%AE%89%E8%A3%85-warp-cli-%E6%88%96-wireproxy)2. 如果我仅使用 socks5 代理模式，我该是安装 WARP-Cli 或 WireProxy？

WARP-Cli 是由 CloudFlare 官方提供的 Linux 客户端，但是目前仅支持 AMD64 的 CPU 架构；WireProxy 是支持 WireGuard 协议的 socks5 代理程序（类似 xray、sing-box 等）。由于脚本使用 WGCF 进行申请 WARP 账号并且生成配置文件、但是由于 WGCF 在香港、美西区域遭到 CloudFlare 的官方限制，如为 CPU 架构为 AMD64 的 VPS、只能使用 WARP-Cli、如为非 AMD64 的 CPU 架构只能等候 CloudFlare 对 WARP-Cli 的重视并开发

对于没遭到 CloudFlare 封禁的大部分区域、且 CPU 架构为 AMD64 的建议：WARP-Cli > WireProxy

### [](#3-%E5%AF%B9%E4%BA%8E%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8-wireguard--sing-box-warp-%E8%8A%82%E7%82%B9%E7%9A%84)3. 对于直接使用 WireGuard / Sing-box WARP 节点的

可使用本脚本的 11 选项进行提取。如果你不想在 VPS 安装 WARP 或者是没有 VPS 的用户，可从下面两个 repl 的其中之一提取

WGCF：[https://replit.com/@misaka-blog/wgcf-profile-generator](https://replit.com/@misaka-blog/wgcf-profile-generator)

WARP-GO：[https://replit.com/@misaka-blog/warpgo-profile-generator](https://replit.com/@misaka-blog/warpgo-profile-generator)

Sing-box：[https://replit.com/@misaka-blog/warpgo-sbfile-generator](https://replit.com/@misaka-blog/warpgo-sbfile-generator)

> 由于配置文件是由服务器生成的，并且每位用户的网络环境不一样，故不会帮助用户设置优选 WARP Endpoint IP。可参考此方法：[https://blog.misaka.rest/2023/03/12/cf-warp-yxip/](https://blog.misaka.rest/2023/03/12/cf-warp-yxip/) 优选可用的 Endpoint IP 并替换 engage.cloudflareclient.com:2408 为自己本地网络环境可用的 WARP Endpoint IP

### [](#4-%E5%9C%A8%E9%83%A8%E5%88%86-ipv6-only-%E7%9A%84%E6%9C%BA%E5%99%A8%E5%AE%89%E8%A3%85-wgcf-warp)4. 在部分 IPv6 Only 的机器安装 WGCF-WARP

运行本脚本代码安装 WARP 之后，由于 EndPoint 不清楚是上游原因还是啥情况被屏蔽了，需要修改 EndPoint IP 以使用

下面是一键修改命令：

```
wg-quick down wgcf
echo "Endpoint = [2001:67c:2b0:db32:0:1:a29f:c001]:2408" >> /etc/wireguard/wgcf.conf
wg-quick up wgcf
curl -4 ip.p3terx.com
```

待回显出现 104 或 8 开头的 IP 即为成功

> 注：由于此地址是 DNS64 对 IPv4 的 WARP Endpoint IP 转换的 IPv6 地址，受制于 DNS64 的服务器速度限制，实际跑起来可能只有 20M 的速度

### [](#5-%E6%88%91%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8-warp-%E7%9A%84-ip-%E5%9C%B0%E5%9D%80%E4%BD%9C%E4%B8%BA%E8%8A%82%E7%82%B9%E7%9A%84%E5%85%A5%E7%AB%99%E5%9C%B0%E5%9D%80%E5%90%97)5. 我可以使用 WARP 的 IP 地址作为节点的入站地址吗？

不行，因为 CloudFlare WARP 的定位仅是 VPN，没有义务为你提供一个专属的 IP 进行服务。

### [](#6-%E4%B8%BA%E5%95%A5-cloudflare-%E6%9C%89%E4%BA%86-warp-cli%E4%BD%BF%E7%94%A8%E8%BF%99%E4%BA%9B%E7%AC%AC%E4%B8%89%E6%96%B9%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84-warp-%E8%84%9A%E6%9C%AC%E8%BF%98%E6%9C%89%E4%BB%80%E4%B9%88%E7%94%A8)6. 为啥 CloudFlare 有了 WARP-Cli，使用这些第三方客户端的 WARP 脚本还有什么用？

由于 WARP-Cli 的开发进度缓慢，如仅支持 AMD64 的 CPU 架构、不支持 IPv6 Only 的 VPS，所以说脚本使用了多种第三方客户端，尽力满足大多数用户的相关需求

[](#warp-endpoint-ip-%E4%BC%98%E9%80%89%E8%84%9A%E6%9C%AC)WARP Endpoint IP 优选脚本
-------------------------------------------------------------------------------

### [](#for-windows)For Windows

下载地址：[https://gitlab.com/Misaka-blog/warp-script/-/blob/main/files/warp-yxip/warp-yxip-win.7z](https://gitlab.com/Misaka-blog/warp-script/-/blob/main/files/warp-yxip/warp-yxip-win.7z)

### [](#for-macos)For MacOS

```
wget -N https://gitlab.com/Misaka-blog/warp-script/-/raw/main/files/warp-yxip/warp-yxip-mac.sh && bash warp-yxip-mac.sh
```

如无 wget 请使用以下命令安装：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update
brew install wget
```

### [](#for-linux-%E5%8C%85%E6%8B%AC%E5%AE%89%E5%8D%93-termux-%E5%92%8C%E8%8B%B9%E6%9E%9C-ish)For Linux （包括安卓 Termux 和苹果 iSH）

```
wget -N https://gitlab.com/Misaka-blog/warp-script/-/raw/main/files/warp-yxip/warp-yxip.sh && bash warp-yxip.sh
```

安卓 Termux 如无 wget 请使用以下命令安装：`pkg update && pkg install wget`

苹果 iSH 初始命令：`apk add -f openssh bash wget`，如遇更新包卡着不动输入以下命令：`sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories`

[](#%E9%B8%A3%E8%B0%A2%E9%A1%B9%E7%9B%AE)鸣谢项目
---------------------------------------------

*   Fscarmen：[https://gitlab.com/fscarmen/warp](https://gitlab.com/fscarmen/warp)
*   CloudFlare WARP：[https://one.one.one.one/](https://one.one.one.one/)
*   Wgcf：[https://github.com/ViRb3/wgcf](https://github.com/ViRb3/wgcf)
*   WARP-GO：[https://gitlab.com/ProjectWARP/warp-go](https://gitlab.com/ProjectWARP/warp-go)
*   某匿名大佬的 CloudFlare WARP EndPoint IP 优选工具及 WARP API
*   CloudflareWarpSpeedTest：[https://github.com/peanut996/CloudflareWarpSpeedTest](https://github.com/peanut996/CloudflareWarpSpeedTest)

[](#%E8%B5%9E%E5%8A%A9)赞助
-------------------------

爱发电：[https://afdian.net/a/Misaka-blog](https://afdian.net/a/Misaka-blog)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f3132323139313336362f3231313533333436392d33353130303966622d396165382d343630312d393932612d6162626635343636356236382e6a7067)