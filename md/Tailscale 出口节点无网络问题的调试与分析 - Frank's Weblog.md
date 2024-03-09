> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [nyan.im](https://nyan.im/p/troubleshoot-tailscale)

> 本文描述了作者使用 Tailscale 并将阿里云服务器作为出口节点时无法访问互联网。

English version: [Troubleshooting Tailscale Exit Node no Internet Issue – Frank’s Weblog](https://nyan.im/p/troubleshooting-tailscale-network-en)

[前文](https://nyan.im/p/tailscale)提到，我使用 Tailscale 将我的所有设备组成了一个 Mesh 网络，并且将位于阿里云北京的轻量应用服务器作为[出口节点](https://nyan.im/p/tailscale#%E5%87%BA%E5%8F%A3%E8%8A%82%E7%82%B9Exit_Node)用于访问一些限制地理位置的网络服务。

然而我却发现在使用该出口节点时完全无法访问互联网。我本以为是 Relay 的网络质量问题就没有在意。但是后来陆续发现该服务器上的其他一些服务都出现了问题，于是进行了一番检查，结果发现问题并没有这么简单。

首先我发现从服务器上完全无法访问互联网，但是直接 curl IP 地址是可以的，这样就基本上将问题定位到了 DNS 上。`resolvectl status`显示有两个 DNS 服务器，因为 IP 以 100.100 打头 [[1]](#100xyz-ip)，我以为这是 Tailscale 内网的 DNS 服务器（实际上不是，请看后文）。

```
Link 2 (eth0)
......
  Current DNS Server: 100.100.2.136
         DNS Servers: 100.100.2.136
                      100.100.2.138
```

我试图`dig @100.100.2.136 baidu.com`来检查 DNS 服务器的回应，得到`connection timed out: no servers could be reached.`。关掉 Tailscale 之后，上述命令的回应则正常。因此我认为问题在于 Tailscale 从某种形式上影响了系统的 DNS 解析。

### Workaround

要 workaround 这个问题，只需要修改服务器上的 DNS 配置即可。编辑`/etc/netplan/99-netcfg.yaml`，在`eth0`接口下加入`nameserver`，配置为国内的公共 DNS。

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: yes
      dhcp6: no
      nameservers:
        addresses: [114.114.114.114]
```

`sudo netplan apply`应用新配置，然后`dig baidu.com`得到正确回应。

然而，虽然修改 DNS 服务器让服务器能够正常访问互联网，但是阿里云内部的很多服务仍然需要阿里云内网的 DNS 解析，比如阿里云的内网 apt 源 (mirrors.cloud.aliyuncs.com) 以及云数据库等产品。将 apt 源配置为公共镜像可以 workaround apt 源问题。

### 定位问题

要定位问题，我们需要找到 IP 地址 100.100.2.136 不能访问的原因。我本以这两个 DNS 服务器是 Tailscale 内网中的 IP，但是通过各种方式都无法访问。经过搜索发现 100.100.2.136 和 100.100.2.138 其实是阿里云提供的内网 DNS 服务器。还有一些阿里云内部服务也使用类似的 IP，例如 apt 源的 IP 为 100.100.2.148，使用 curl 访问时也遇到了无法连接的问题。

因此我们可以得出一个初步结论：Tailscale 通过某种形式影响了 100.100.x.x IP 段的访问。

### 可能性

#### 路由

我的第一反应是 Tailscale 路由了整个`100.100.x.x` IP 段。然而，根据 Tailscale 文档，Tailscale 只路由了被分配的 IP 地址而不是整个 CIDR。`ip route list`也证实了这点。

```
ip route list table 52
100.69.x.x dev tailscale0
100.90.x.x dev tailscale0
100.96.x.x dev tailscale0
100.98.x.x dev tailscale0
100.100.100.100 dev tailscale0
100.104.x.x dev tailscale0
100.121.x.x dev tailscale0
100.127.x.x dev tailscale0
```

`ip route get 100.100.2.136`得到如下结果，表示数据包将被路由到 eth0 接口，这说明路由表是正确的，问题并不在路由上。

```
100.100.2.136 via 172.24.63.253 dev eth0 src 172.24.4.100 uid 0
    cache
```

#### iptables

另一个可能对数据包造成影响的是 iptables，`iptables -S`中和 Tailscale 相关的有如下几条：

```
-A ts-forward -i tailscale0 -j MARK --set-xmark 0x40000/0xffffffff
-A ts-forward -m mark --mark 0x40000 -j ACCEPT
-A ts-forward -s 100.64.0.0/10 -o tailscale0 -j DROP
-A ts-forward -o tailscale0 -j ACCEPT
-A ts-input -s 100.92.187.56/32 -i lo -j ACCEPT
-A ts-input -s 100.115.92.0/23 ! -i tailscale0 -j RETURN
-A ts-input -s 100.64.0.0/10 ! -i tailscale0 -j DROP
```

其中最后一条规则 drop 掉了整个 IP 段的数据包，使用`iptables -D` 删除规则后，问题解决。

经过搜索发现今年早些时候已经有人提出了 Issue：

[1] [tailscale drops 100.64.0.0/10 on firewall when ipv4 is disabled · Issue #3837 · tailscale/tailscale · GitHub](https://github.com/tailscale/tailscale/issues/3837)

[2] [FR: netfilter CGNAT mode when non-Tailscale CGNAT addresses should be allowed · Issue #3104 · tailscale/tailscale · GitHub](https://github.com/tailscale/tailscale/issues/3104)

### 结论

综上所述，引起这个问题的是 Tailscale 设置的一条屏蔽`100.64.0.0/10` IP 段的防火墙规则，而正好阿里云内网的一些服务位于这些 IP 段而被屏蔽。根据 [Tailscale CLI 文档](https://tailscale.com/kb/1080/cli/)，只需要在启动 tailscale 时加入`--netfilter-mod=off`参数，即可避免这条规则被设置。但是，这样会带来一些[安全隐患](https://seclists.org/oss-sec/2019/q4/122)。

Tailscale 设置这条规则，是因为其用于 Tailscale 网络的 IP 段 (`100.64.0.0/10`)[[1]](#100xyz-ip) 为[运营商级 NAT(Carrier Grade NAT, CGNAT)](https://en.wikipedia.org/wiki/Carrier-grade_NAT)，因而假设其不会被通常的私有网络使用，然而阿里云使用了这个 IP 段用于内网服务，因而引起了冲突。

### References

[iptables(8) – Linux man page](https://linux.die.net/man/8/iptables)

[1] [What are these 100.x.y.z addresses? · Tailscale](https://tailscale.com/kb/1015/100.x-addresses/)