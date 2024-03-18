> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.typeboom.com](https://www.typeboom.com/archives/112/)

> 前言上文讲到基于 Tunnelbroker 搭建的随机 ipv6 代理，但是它的运行效率实在是低下，并发达到 200 左右就已经是极限。此外，Tunnelbroker 政策不断在收紧，部分 IDC 还不允许 6IN...

前言
--

上文讲到基于 Tunnelbroker 搭建的随机 ipv6 代理，但是它的运行效率实在是低下，并发达到 200 左右就已经是极限。此外，Tunnelbroker 政策不断在收紧，部分 IDC 还不允许 6IN4 协议，大部分 waf 也已经开始直接封锁 / 64 的 IPv6 地址

正文
--

俗话说车到山前必有路，部分 IDC 也在更新技术，比如 Buyvm 提供 / 48 的 IPv6，相当于 65536 个 / 64，并且可以自己决定路由的最后一跳。这就为我们提供了很多便利

和上文一样，首先启用`ip_nonlocal_bind`特性

copy

```
echo "net.ipv6.ip_nonlocal_bind = 1" >> /etc/sysctl.conf
sysctl -p
```

然后使用命令`ip a`查看自己的分配到的子网

copy

```
root@localhost:~# ip a
...<此处省略一部分输出>...
inet 209.55.56.95/24 metric 100 brd 209.55.56.255 scope global dynamic eth0
   valid_lft 2532037sec preferred_lft 2532037sec
inet6 2605:6466:4481::/48 scope global
   valid_lft forever preferred_lft forever
inet6 2605:6434:20:554::/48 scope global
   valid_lft forever preferred_lft forever
inet6 fe80::216:67ff:fefc:98a8/64 scope link
   valid_lft forever preferred_lft forever
```

其中，`2605:6466:4481::/48`就是分配到的子网。在拿到子网之后，我们为其添加路由，告诉系统这整个段都归自己了：

copy

```
ip route add local <被分配的IPv6子网> dev lo
```

添加好路由之后，就可以用 curl 来验证一下了：

copy

```
curl --int <子网中的随便一个IPv6地址> ip.sb
```

如果返回的 IPv6 地址和 int 参数指定的 IPv6 地址的一样就没有问题，可以开始添加 ip6tables 规则了。

同时使用 ip6tables 中的 statistic 和 SNAT 模块，就可以按一定规则指定出口 ip，这里使用 random 模式，再递增地指定概率 probability。举个例子，假如有三个 IP，就可以写成：

copy

```
ip6tables -t nat -A POSTROUTING -m statistic --mode random --probability 0.33 -j SNAT --to-source <IP1>
ip6tables -t nat -A POSTROUTING -m statistic --mode random --probability 0.5 -j SNAT --to-source <IP2>
ip6tables -t nat -A POSTROUTING -m statistic --mode random --probability 1.0 -j SNAT --to-source <IP3>
```

为什么概率要这样写，是因为 ip6tables 的规则是有先后顺序的，从上到下第一条有 33% 的概率匹配到，第二条是剩下的 66% 中的 50% 概率，而最后一个则是另外 50%。这样就可以变相达到 “随机” 的目的。

为了方便配置，我写了一个 Python 脚本，有一些瑕疵，但已经可用了：

copy

```
https://raw.githubusercontent.com/Rainscall/RO6PG.sh/main/ip6tables_rule_gen.py
```

下载后执行时需要加上参数：

copy

```
python3 ip6tables_rule_gen.py <子网,如2605:6466:4481::/48> <IP个数，建议4000> >> temp.sh
```

执行后当前目录将生成一个 shell 脚本`temp.sh`，执行它就可以生效了

可以用以下命令来验证一下，如果每次返回都是不同的 IPv6 地址就成功了 (Ctrl+C 退出)：

copy

```
while true; do curl ip.sb;done
```

如果你想要将其搭建成代理，可以考虑使用 singbox 或者 squid 等软件，这里就不再赘述了