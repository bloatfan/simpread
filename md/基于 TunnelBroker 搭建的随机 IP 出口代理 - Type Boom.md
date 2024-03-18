> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.typeboom.com](https://www.typeboom.com/archives/111/)

> 最近需要做一个对各种 “xxScan” 网站进行爬取数据的爬虫，用于监听用户是否完成付款。但是网站采用了基于 IP 的速率限制规则，次数多了，一限就是好一阵，很容易出现 “掉单” 的情况，人工处理这样的订单...

最近需要做一个对各种 “xxScan” 网站进行爬取数据的爬虫，用于监听用户是否完成付款。但是网站采用了基于 IP 的速率限制规则，次数多了，一限就是好一阵，很容易出现 “掉单” 的情况，人工处理这样的订单也很麻烦。至于 IP 代理池，这种服务较为昂贵，且不确定是否客户端本身会不会带有后门，毕竟这些平台大多数 IP 都是来源于被感染的设备所组成的 BotNet。

现在大多数网站都已经支持了 IPv6，而 IPv6 的资源十分丰富，国内的三大毒瘤都会直接分配一个 /64 甚至 /56 的子网，也就是 2^64 个独立的 IPv6 地址。

但爬虫这种东西一般会放置在服务器上运行，至少不能使用自己的真实 IP 吧。而部分 IDC 是不分配 IPv6 的，但是不用担心，在很多年前，HE 和 Cogent 在争夺 T1 地位的时候，HE 推出了一个叫 TunnelBroker 的东西，可以通过 6in4 协议为你的机器添加公网 IPv6，并且是直接 Route 到设备上，更加便于配置。

教程
--

首先你需要准备一台没有 IPv6，但是拥有公网 IPv4，且防火墙允许 ICMP、TCP、6in4 协议通过的 VPS，大多数系统均可，下文以 CentOS 7.9 为例。

准备完服务器后，可以前往 [TunnelBroker 官网](https://tunnelbroker.net/)注册一个账户，在创建新隧道时，隧道服务器 (Tunnel server) 尽量选择离服务器近的地方(国内服务器建议直接美西)。

然后就可以打开隧道详情 (Tunnel Details)，选择配置样板 (Example Configurations)，在 Select Your OS 下拉栏中选择：“Linux-route2”(Redhat 系)，然后复制输入框中的内容，例子如下：

copy

```
modprobe ipv6
ip tunnel add he-ipv6 mode sit remote <远程服务器地址，一般不动> local <你的服务器IPv4地址，如果是NAT需要写内网地址> ttl 255
ip link set he-ipv6 up
ip addr add <分配给你的IPv6主地址>/64 dev he-ipv6
ip route add ::/0 dev he-ipv6
ip -f inet6 addr
```

将这些直接复制到 SSH 中执行，不出意外的话使用命令`ifconfig`就可以新加的网卡`he-ipv6`了。要验证是否可用，可以使用命令：

copy

```
curl ipv6.ip.sb
```

如果成功访问并返回了一个 ipv6 地址，则配置成功，可以配置随机出口代理了。

### 配置随机出口代理

首先启用内核的 `ip_nonlocal_bind` 特性来允许绑定任意 IP：

copy

```
sysctl net.ipv6.ip_nonlocal_bind=1
```

然后为隧道添加路由

copy

```
ip route add local <被分配的IPv6子网> dev lo
```

完成后就可以再利用 CURL 验证一下是否可以使用该子网下的任何 IPv6 地址出口了：

copy

```
curl --int <子网中的随便一个IPv6地址> ip.sb
```

如果成功访问并返回了指定的 ipv6 地址，则配置成功。接下来就可以开始编译并安装随机出口服务端 (由 [zu1k](https://zu1k.com/) 编写) 了，执行下列命令即可：

copy

```
yum install git -y
git clone https://github.com/zu1k/http-proxy-ipv6-pool
cd http-proxy-ipv6-pool
yum install rustc -y
curl -sSf https://static.rust-lang.org/rustup.sh | sh
cargo build
cd target/debug
mv http-proxy-ipv6-pool /usr/local/bin
chmod a+x /usr/local/bin/http-proxy-ipv6-pool
```

完成后，执行 http-proxy-ipv6-pool，如果没有任何输出，则安装成功，按`Ctrl+C`退出软件。这个软件似乎没有后台执行，为了方便管理，可以使用 screen 使其长期运行。使用软件可以用以下语法：

copy

```
http-proxy-ipv6-pool -b <监听IP>:<监听端口号> -i <被分配的IPv6子网>
```

举个例子，假如我想在`127.0.0.1`的端口`45678`上搭建 HTTP 代理，IPv6 子网 2001:173:12:44c::/64，则可以使用命令：

copy

```
http-proxy-ipv6-pool -b 127.0.0.1:45678 -i 2001:173:12:44c::/64
```

软件开始工作后，可以使用 curl 来验证是否有效，假设和刚才的例子一样，软件监听`127.0.0.1`的端口`45678`，则可以使用命令：

copy

```
curl ip.sb -x 127.0.0.1:45678
```

多运行几次，不出意外的话每次返回的 IP 都是不同的，如果要将其转为 Socks5 等协议，还可以使用 Singbox 等软件，这里就不再赘述了。