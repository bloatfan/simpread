> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [e-mailky.github.io](https://e-mailky.github.io/2018-07-14-dnsmasq)

> dnsmasq详解及配置, Linux, GNU,Linux,OpenWrt,TCP/IP, dnsmasq的简介    Dnsmasq 提供 DNS 缓存和 DHCP 服务功能。作为域名解析服务器(DNS)，dnsmasq可以通过缓存DNS 请求来提高对访问过的网址的连接速度。作为DHCP 服务器，dnsmasq 可以用于为局域网电脑分配内网ip地址和提供路由。DNS和DHCP两个功能可以同时或分别单独...

    Dnsmasq 提供 DNS 缓存和 DHCP 服务功能。作为域名解析服务器 (DNS)，dnsmasq 可以通过缓存 DNS 请求来提高对访问过的网址的连接速度。作为 DHCP 服务器，dnsmasq 可以用于为局域网电脑分配内网 ip 地址和提供路由。 DNS 和 DHCP 两个功能可以同时或分别单独实现。dnsmasq 轻量且易配置，适用于个人用户或少于 50 台主机的网络。此外它还自带了一个 PXE 服务器。。

1.  将 Dnsmasq 作为本地 DNS 服务器使用，直接修改电脑的本地 DNS 的 IP 地址即可
2.  应对 ISP 的 DNS 劫持（反 DNS 劫持），输入一个不存在的域名，正常的情况下浏览器是显示无法连接， DNS 劫持会跳转到一个广告页面。先随便 nslookup 一个不存在的域名，看看 ISP 商劫持的 IP 地址
3.  智能 DNS 加快解析速度，打开 / etc/dnsmasq.conf 文件，server = 后面可以添加指定的 DNS，例如国内外不同的网站使用不同的 DNS。

```
国内指定DNS
server=/cn/114.114.114.114
server=/taobao.com/114.114.114.114
server=/taobaocdn.com/114.114.114.114

国外指定DNS
server=/google.com/223.5.5.5
```

1.  屏蔽网页广告，将指广告的 URL 指定 127 这个 IP，就可以将网页上讨厌的广告给去掉了
    
    address=/ad.youku.com/127.0.0.1 address=/ad.iqiyi.com/127.0.0.1
    
2.  指定域名解析到特定的 IP 上。这个功能可以让你控制一些网站的访问，非法的 DNS 就经常把一些正规的网站解析到不正确 IP 上。
    
    address=/freehao123.com/123.123.123.123
    
3.  管理控制内网 DNS，首先将局域网中的所有的设备的本地 DNS 设置为已经安装 Dnsmasq 的服务器 IP 地址。 然后修改已经安装 Dnsmasq 的服务器 Hosts 文件：/etc/hosts，指定域名到特定的 IP 中。 例如想让局域网中的所有用户访问 www.freehao123.com 时跳转到 192.168.0.2， 添加：192.168.0.2 www.freehao123.com 在 Hosts 文件中既可，整个过程也可以说是 “DNS 劫持”。
    

dnsmasq 先去解析 hosts 文件， 再去解析 / etc/dnsmasq.d / 下的 *.conf 文件，并且这些文件的优先级要高于 dnsmasq.conf， 我们自定义的 resolv.dnsmasq.conf 中的 DNS 也被称为上游 DNS，这是最后去查询解析的；

如果不想用 hosts 文件做解析，我们可以在 / etc/dnsmasq.conf 中加入 no-hosts 这条语句，这样的话就直接查询上游 DNS 了， 如果我们不想做上游查询，就是不想做正常的解析，我们可以加入 no-reslov 这条语句。

编辑 dnsmasq 的配置文件 /etc/dnsmasq.conf 。这个文件包含大量的选项注释。

<table><thead><tr><th>具体参数</th><th>参数说明</th></tr></thead><tbody><tr><td>resolv-file</td><td>定义 dnsmasq 从哪里获取上游 DNS 服务器的地址， 默认从 / etc/resolv.conf 获取</td></tr><tr><td>strict-order</td><td>表示严格按照 resolv-file 文件中的顺序从上到下进行 DNS 解析，直到第一个解析成功为止。</td></tr><tr><td>listen-address</td><td>定义 dnsmasq 监听的地址，默认是监控本机的所有网卡上</td></tr><tr><td>address</td><td>启用泛域名解析，即自定义解析 a 记录，例如：address=/long.com/192.168.115.10 访问 long.com 时的所有域名都会被解析成 192.168.115.10</td></tr><tr><td>bogus-nxdomain</td><td>对于任何被解析到此 IP 的域名，将响应 NXDOMAIN 使其解析失效，可以多次指定通常用于对于访问不存在的域名，禁止其跳转到运营商的广告站点</td></tr><tr><td>server</td><td>指定使用哪个 DNS 服务器进行解析，对于不同的网站可以使用不同的域名对应解析。例如：server=/google.com/8.8.8.8 #表示对于 google 的服务，使用谷歌的 DNS 解析。</td></tr></tbody></table>

```
[root@localhost ~]# dnsmasq --test
dnsmasq: syntax check OK.
```

要在单台电脑上以守护进程方式启动 dnsmasq 做 DNS 缓存服务器，编辑 / etc/dnsmasq.conf，添加监听地址：

如果用此主机为局域网提供默认 DNS，请用为该主机绑定固定 IP 地址，设置：

```
listen-address=127.0.0.1
```

这种情况建议配置静态 IP

多个 ip 地址设置:

```
listen-address=192.168.x.x
```

Linux 处理 DNS 请求时有个限制，在 resolv.conf 中最多只能配置三个域名服务器（nameserver）。作为一种变通方法, 可以在 resolv.conf 文件中只保留 localhost 作为域名服务器，然后为外部域名服务器另外创建 resolv-file 文件。 首先，为 dnsmasq 新建一个域名解析文件：

```
listen-address=127.0.0.1,192.168.x.x
```

然后编辑 /etc/dnsmasq.conf 让 dnsmasq 使用新创建的域名解析文件：

```
[root@localhost ~]# vim /etc/resolv.dnsmasq.conf

# Google's nameservers, for example

nameserver 8.8.8.8

nameserver 8.8.4.4
```

dhcpcd 可以是通过创建（或编辑）/etc/resolv.conf.head 文件或 /etc/resolv.conf.tail 文件来指定 dns 服务器， 使 / etc/resolv.conf 不会被每次都被 dhcpcd 重写

```
[root@localhost ~]# vim  /etc/dnsmasq.conf

...

resolv-file=/etc/resolv.dnsmasq.conf
```

要使用 dhclient， 取消 /etc/dhclient.conf 文件中如下行的注释：

```
echo "nameserver 127.0.0.1" > /etc/resolv.conf.head   设置dns服务器为127.0.0.1
```

NetworkManager 可以靠自身配置文件的设置项启动 dnsmasq 。在 NetworkManager.conf 文件的 [main] 节段添加 dns=dnsmasq 配置语句，然后禁用由 systemd 启动的 dnsmasq.service:

```
prepend domain-name-servers 127.0.0.1;
```

可以在 /etc/NetworkManager/dnsmasq.d/ 目录下为 dnsmasq 创建自定义配置文件。例如，调整 DNS 缓存大小（保存在内存中）：

```
[root@localhost ~]# vim /etc/NetworkManager/NetworkManager.conf

[main]

plugins=keyfile

dns=dnsmasq
```

dnsmasq 被 NetworkManager 启动后，此目录下配置文件中的配置将取代默认配置。

**IPv6**

启用 dnsmasq 在 NetworkManager 可能会中断仅持 IPv6 的 DNS 查询 (例如 dig -6 [hostname]) 否则将工作。 为了解决这个问题，创建以下文件将配置 dnsmasq 总是监听 IPv6 的 loopback：

```
[root@localhost ~]# vim /etc/NetworkManager/dnsmasq.d/cache
cache-size=1000
```

此外， dnsmasq 不优先考虑上游 IPv6 的 DNS。不幸的是 NetworkManager 已不这样做 (Ubuntu Bug)。 一种解决方法是将禁用 IPv4 DNS 的 NetworkManager 的配置，假设存在。

其他方式

另一种选择是在 NetworkManagers“设置（通常通过右键单击小程序）和手动输入设置。 设置将取决于前端中使用的类型; 这个过程通常涉及右击小程序，编辑（或创建）一个配置文件， 然后选择 DHCP 类型为 “自动（指定地址）。”DNS 地址将需要输入，通常以这种形式：127.0.0.1, DNS-server-one, ….

dnsmasq 默认关闭 DHCP 功能，如果该主机需要为局域网中的其他设备提供 IP 和路由，应该对 dnsmasq 配置文件 (/etc/dnsmasq.conf) 必要的配置如下：

```
[root@localhost ~]# vim /etc/NetworkManager/dnsmasq.d/ipv6_listen.conf
listen-address=::1
```

查看租约

```
[root@localhost ~]# vim  /etc/dnsmasq.conf

# Only listen to routers' LAN NIC.  Doing so opens up tcp/udp port 53 to

# localhost and udp port 67 to world:

interface=<LAN-NIC>

 

# dnsmasq will open tcp/udp port 53 and udp port 67 to world to help with

# dynamic interfaces (assigning dynamic ips). Dnsmasq will discard world

# requests to them, but the paranoid might like to close them and let the

# kernel handle them:

bind-interfaces

 

# Dynamic range of IPs to make available to LAN pc

dhcp-range=192.168.111.50,192.168.111.100,12h

 

# If you’d like to have dnsmasq assign static IPs, bind the LAN computer's

# NIC MAC address:

dhcp-host=aa:bb:cc:dd:ee:ff,192.168.111.50
```

它可以将一个自定义域添加到主机中的（本地）网络：

```
[root@localhost ~]# cat /var/lib/misc/dnsmasq.leases
```

设置为开机启动：

```
local=/home.lan/
domain=home.lan
```

立即启动 dnsmashq：

```
[root@localhost ~]# systemctl enable dnsmasq
```

需要重启网络服务以使 DHCP 客户端重建一个新的 /etc/resolv.conf

查看 dnsmasq 是否启动正常，查看系统日志：

```
[root@localhost ~]# systemctl start dnsnsmasq
```

```
[root@localhost ~]# journalctl -u d
```

每天我们的工作和娱乐休闲都离不开电脑，经常看到电脑右下角弹出图片广告！大部分这个都是被劫持 DNS 商家推送过来的， 看起来很讨厌。虽然很多门户网站，比如 360、百度、阿里都有推出他们 DNS 服务，我们将本地的 DNS IP 地址更换成他们的， 在一定程度上，可以解决我们访问网速、广告拦截的问题。但是他们会推送自己的广告业务。所以我们自己可以架设本地 DNS 服务器， 这样用自己的 DNS 就不会有广告的问题。

Dnsmasq 也不是仅仅这个用途，我们也可以作为局域网机器批量 IP 维护使用，以及局域网解决特定网址域名禁止访问。

```
#no-hosts

# 添加读取额外的 hosts 文件路径，可以多次指定。如果指定为目录，则读取目录中的所有文件。

#addn-hosts=/etc/dnsmasq.hosts.d

# 读取目录中的所有文件，文件更新将自动读取

#hostsdir=/etc/dnsmasq.hosts.d

# 例如，/etc/hosts中的os01将扩展成os01.example.com

#expand-hosts

 

##############################################################################

# 缓存时间设置，一般不需要设置

# 本地 hosts 文件的缓存时间，通常不要求缓存本地，这样更改hosts文件后就即时生效。

#local-ttl=3600

# 同 local-ttl 仅影响 DHCP 租约

#dhcp-ttl=<time>

# 对于上游返回的值没有ttl时，dnsmasq给一个默认的ttl，一般不需要设置，

#neg-ttl=<time>

# 指定返回给客户端的ttl时间，一般不需要设置

#max-ttl=<time>

# 设置在缓存中的条目的最大 TTL。

#max-cache-ttl=<time>

# 不需要设置，除非你知道你在做什么。

#min-cache-ttl=<time>

# 一般不需要设置

#auth-ttl=<time>

 

##############################################################################

# 记录dns查询日志，如果指定 log-queries=extra 那么在每行开始处都有额外的日志信息。

#log-queries

# 设置日志记录器，'-' 为 stderr，也可以是文件路径。默认为：DAEMON，调试时使用 LOCAL0。

#log-facility=<facility>

#log-facility=/var/log/dnsmasq/dnsmasq.log

# 异步log，缓解阻塞，提高性能。默认为5，最大100。

#log-async[=<lines>]

#log-async=50

 

##############################################################################

# 指定用户和组

#user=nobody

#group=nobody

 

##############################################################################

# 指定DNS的端口，默认53，设置 port=0 将完全禁用 DNS 功能，仅使用 DHCP/TFTP

#port=53

# 指定 EDNS.0 UDP 包的最大尺寸，默认为 RFC5625 推荐的 edns-packet-max=4096

#edns-packet-max=<size>

# 指定向上游查询的 UDP 端口，默认是随机端口，指定后降低安全性、加快速度、减少资源消耗。

# 设置为 '0' 由操作系统分配。

#query-port=53535

# 指定向上游查询的 UDP 端口范围，方便防火墙设置。

#min-port=<port>

#max-port=<port>

# 指定接口，指定后同时附加 lo 接口，可以使用'*'通配符。

# 不能使用接口别名（例如："eth1:0"），请用 listen-address 选项替代。

#interface=wlp2s0

# 指定排除的接口，排除优先级高，可以使用'*'通配符

#except-interface=

# 仅接受同一子网的 DNS 请求。

# 仅在未指定 interface、except-interface、listen-address 或者 auth-server 时有效。

#local-service

# 指定不提供 DHCP 或 TFTP 服务的接口，仅提供 DNS 服务。

#no-dhcp-interface=enp3s0

# 指定IP地址，可以多次指定。

# interface 选项和 listen-address 选项可以同时使用。

# 下面两行与指定 interface 选项的作用类似。

listen-address=192.168.10.17

#listen-address=127.0.0.1

# 通常情况下即使设置了 interface 选项（例如：interface=wlp2s0 ）

# 将仍然绑定到通配符地址（例如：*:53 ）。

# 开启此项将仅监听指定的接口。

# 适用于在同一主机的不同接口或 IP 地址上运行多个 dns 服务器。

bind-interfaces

# 对于新添加的接口不进行绑定。仅 Linux 系统支持，其他系统等同于 bind-interfaces 选项。

#bind-dynamic

 

##############################################################################

# 如果 hosts 中的主机有多个 IP 地址，仅返回对应子网的 IP 地址。

localise-queries

# 如果反向查找的是私有地址例如192.168.X.X，仅从 hosts 文件查找，不再转发到上游服务器

#bogus-priv

# 对于任何被解析到此 IP 的域名，将响应 NXDOMAIN 使其解析失效，可以多次指定

# 通常用于对于访问不存在的域名，禁止其跳转到运营商的广告站点。

#bogus-nxdomain=64.94.110.11

# 忽略包含指定地址的 A 记录查询的回复。

# 例如上游有台 dns 服务器伪造 www.baidu.com 的 IP 为 1.1.1.1 并且响应速度非常快。

# 指定 ignore-address=1.1.1.1 可以忽略它的响应信息，

# 从而等待 www.baidu.com 正确的查询结果。

#ignore-address=<ipaddr>

filterwin2k

 

##############################################################################

# 指定 resolv-file 文件路径，默认/etc/resolv.conf

#resolv-file=/etc/resolv.conf

# 不读取 resolv-file 来确定上游服务器

#no-resolv

# 在编译时需要启用 DBus 支持。

#enable-dbus[=<service-name>]

# 严格按照resolv.conf中的顺序进行查找

#strict-order

# 向所有上游服务器发送查询，而不是一个。

all-servers

# 启用转发循环检测

#dns-loop-detect

 

##############################################################################

# 这项安全设置是拒绝解析包含私有 IP 地址的域名，

# 这些IP地址包括如下私有地址范围：10.0.0.0/8、172.16.0.0/12、192.168.0.0/16。

# 其初衷是要防止类似上游DNS服务器故意将某些域名解析成特定私有内网IP而劫持用户这样的安全***。

# 直接在配置文件中注销 stop-dns-rebind 配置项从而禁用该功能。

# 这个方法确实可以一劳永逸的解决解析内网 IP 地址的问题，

# 但是我们也失去了这项安全保护的特性，所以在这里我不推荐这个办法。

# 使用 rebind-domain-ok 进行特定配置，顾名思义该配置项可以有选择的忽略域名的 rebind 行为

stop-dns-rebind

rebind-localhost-ok

#rebind-domain-ok=[<domain>]|[[/<domain>/[<domain>/]

rebind-domain-ok=/.test.com/

 

##############################################################################

# 也不要检测 /etc/resolv.conf 的变化

#no-poll

# 重启后清空缓存

clear-on-reload

# 完整的域名才向上游服务器查找，如果仅仅是主机名仅查找hosts文件

domain-needed

 

##############################################################################

# IP地址转换

#alias=[<old-ip>]|[<start-ip>-<end-ip>],<new-ip>[,<mask>]

##############################################################################

#local=[/[<domain>]/[domain/]][<ipaddr>[#<port>][@<source-ip>|<interface>[#<port>]]

#server=[/[<domain>]/[domain/]][<ipaddr>[#<port>][@<source-ip>|<interface>[#<port>]]

server=/test.com/192.168.10.117

server=/10.168.192.in-addr.arpa/192.168.10.117

#rev-server=<ip-address>/<prefix-len>,<ipaddr>[#<port>][@<source-ip>|<interface>[#<port>]]

 

# 将任何属于 <domain> 域名解析成指定的 <ipaddr> 地址。

# 也就是将 <domain> 及其所有子域名解析成指定的 <ipaddr> IPv4 或者 IPv6 地址，通常用于屏蔽特定的域名。

# 一次只能指定一个 IPv4 或者 IPv6 地址，要同时返回 IPv4 和IPv6 地址，请多次指定 address= 选项。

# 注意： /etc/hosts 以及 DHCP 租约将覆盖此项设置。

#address=/<domain>/[domain/][<ipaddr>]

 

#ipset=/<domain>/[domain/]<ipset>[,<ipset>]

#mx-host=<mx name>[[,<hostname>],<preference>]

#mx-target=<hostname>

 

# SRV 记录

#srv-host=<_service>.<_prot>.[<domain>],[<target>[,<port>[,<priority>[,<weight>]]]]

 

# A, AAAA 和 PTR 记录 

#host-record=<name>[,<name>....],[<IPv4-address>],[<IPv6-address>][,<TTL>]

 

# TXT 记录

#txt-record=<name>[[,<text>],<text>]

 

# PTR 记录 

#ptr-record=<name>[,<target>]

 

#naptr-record=<name>,<order>,<preference>,<flags>,<service>,<regexp>[,<replacement>]

 

# CNAME 别名记录

#cname=<cname>,<target>[,<TTL>]

 

 

#dns-rr=<name>,<RR-number>,[<hex data>]

#interface-name=<name>,<interface>[/4|/6]

#synth-domain=<domain>,<address range>[,<prefix>]

#add-mac[=base64|text]

#add-cpe-id=<string>

#add-subnet[[=[<IPv4 address>/]<IPv4 prefix length>][,[<IPv6 address>/]<IPv6 prefix length>]]

##############################################################################

 

##############################################################################

# 缓存条数，默认为150条，cache-size=0 禁用缓存。

cache-size=1000

# 不缓存未知域名缓存，默认情况下dnsmasq缓存未知域名并直接返回为客户端。

no-negcache

# 指定DNS同属查询转发数量

dns-forward-max=1000

 

##############################################################################

#dnssec

#trust-anchor=[<class>],<domain>,<key-tag>,<algorithm>,<digest-type>,<digest>

#dnssec-check-unsigned

#dnssec-no-timecheck

#dnssec-timestamp=<path>

#proxy-dnssec

#dnssec-debug

 

##############################################################################

#auth-server=<domain>,<interface>|<ip-address>

#auth-zone=<domain>[,<subnet>[/<prefix length>][,<subnet>[/<prefix length>].....]]

#auth-zone=<domain>[,<interface name>[/6|/4][,<interface name>[/6|/4].....]]

#auth-soa=<serial>[,<hostmaster>[,<refresh>[,<retry>[,<expiry>]]]]

#auth-sec-servers=<domain>[,<domain>[,<domain>...]]

#auth-peer=<ip-address>[,<ip-address>[,<ip-address>...]]

 

# 启用连接跟踪，读取 Linux 入栈 DNS 查询请求的连接跟踪标记，

# 并且将上游返回的响应信息设置同样的标记。

# 用于带宽控制和防火墙部署。

# 此选项必须在编译时启用 conntrack 支持，并且内核正确配置并加载 conntrack。

# 此选项不能与 query-port 同时使用。

#conntrack

 

 

##############################################################################

#

#        DHCP 选项

#

##############################################################################

# 设置 DHCP 地址池，同时启用 DHCP 功能。

# IPv4 <mode> 可指定为 static|proxy ，当 <mode> 指定为 static 时，

# 需用 dhcp-host 手动分配地址池中的 IP 地址。

# 当 <mode> 指定为 proxy 时，为指定的地址池提供 DHCP 代理。

#dhcp-range=[tag:<tag>[,tag:<tag>],][set:<tag>,]<start-addr>[,<end-addr>][,<mode>][,<netmask>[,<broadcast>]][,<lease time>]

#dhcp-range=172.16.0.2,172.16.0.250,255.255.255.0,1h

#dhcp-range=192.168.10.150,192.168.10.180,static,255.255.255.0,1h

 

# 根据 MAC 地址或 id 固定分配客户端的 IP 地址、主机名、租期。

# IPv4 下指定 id:* 将忽略 DHCP 客户端的 ID ，仅根据 MAC 来进行 IP 地址分配。

# 在读取 /etc/hosts 的情况，也可以根据 /etc/hosts 中的主机名分配对应 IP 地址。

# 指定 ignore 将忽略指定客户端得 DHCP 请求。

#dhcp-host=[<hwaddr>][,id:<client_id>|*][,set:<tag>][,<ipaddr>][,<hostname>][,<lease_time>][,ignore]

#dhcp-hostsfile=<path>

#dhcp-hostsdir=<path>

# 读取 /etc/ethers 文件 与使用 dhcp-host 的作用相同。IPv6 无效。

#read-ethers

 

# 指定给 DHCP 客户端的选项信息，

# 默认情况下 dnsmasq 将发送：子网掩码、广播地址、DNS 服务器地址、网关地址、域等信息。

# 指定此选项也可覆盖这些默认值并且设置其他选项值。

# 重要：可以使用 option:<option-name>或者 option号 来指定。

# <option-name> 和 option号的对应关系可使用命令：

# dnsmasq --help dhcp 以及 dnsmasq --help dhcp6 查看，这点很重要。

# 例如设置网关参数，既可以使用 dhcp-option=3,192.168.4.4 也可以使用 dhcp-option = option:router,192.168.4.4。

# 0.0.0.0 意味着当前运行 dnsmasq 的主机地址。

# 如果指定了多个 tag:<tag> 必须同时匹配才行。

# [encap:<opt>,][vi-encap:<enterprise>,][vendor:[<vendor-class>],] 有待继续研究。

#dhcp-option=[tag:<tag>,[tag:<tag>,]][encap:<opt>,][vi-encap:<enterprise>,][vendor:[<vendor-class>],][<opt>|option:<opt-name>|option6:<opt>|option6:<opt-name>],[<value>[,<value>]]

#dhcp-option-force=[tag:<tag>,[tag:<tag>,]][encap:<opt>,][vi-encap:<enterprise>,][vendor:[<vendor-class>],]<opt>,[<value>[,<value>]]

#dhcp-optsfile=<path>

#dhcp-optsdir=<path>

#dhcp-option=3,1.2.3.4

#dhcp-option=option:router,1.2.3.4

#dhcp-option=option:router,192.168.10.254

#dhcp-option=option:dns-server,192.168.10.254,221.12.1.227,221.12.33.227

 

##############################################################################

# (IPv4 only) 禁用重用服务器名称和文件字段作为额外的 dhcp-option 选项。

# 一般情况下 dnsmasq 从 dhcp-boot 移出启动服务器和文件信息到 dhcp-option 选项中。

# 这使得在 dhcp-option 选项封包中有额外的选项空间可用，但是会使老的客户端混淆。

# 此选项将强制使用简单并安全的方式来避免此类情况。可以认为是一个兼容性选项。

#dhcp-no-override

 

##############################################################################

# 配置 DHCP 中继。

# <local address> 是运行 dnsmasq 的接口的 IP 地址。

# 所有在 <local address> 接口上接收到的 DHCP 请求将中继到 <server address> 指定的远程 DHCP 服务器。

# 可以多次配置此选项，使用同一个 <local address> 转发到多个不同的 <server address> 指定的远程 DHCP 服务器。

# <server address> 仅允许使用 IP 地址，不能使用域名等其他格式。

# 如果是 DHCPv6，<server address> 可以是 ALL_SERVERS 的多播地址 ff05::1:3 。

# 在这种情况下必须指定接口 <interface> ，不能使用通配符，用于直接多播到对应的 DHCP 服务器所在的接口。

# <interface> 指定了仅允许接收从 <interface> 接口的 DHCP 服务器相应信息。

#dhcp-relay=<local address>,<server address>[,<interface>]

 

##############################################################################

# 设置标签

#dhcp-vendorclass=set:<tag>,[enterprise:<IANA-enterprise number>,]<vendor-class>

#dhcp-userclass=set:<tag>,<user-class>

#dhcp-mac=set:<tag>,<MAC address>

#dhcp-circuitid=set:<tag>,<circuit-id>

#dhcp-remoteid=set:<tag>,<remote-id>

#dhcp-subscrid=set:<tag>,<subscriber-id>

#dhcp-match=set:<tag>,<option number>|option:<option name>|vi-encap:<enterprise>[,<value>]

#tag-if=set:<tag>[,set:<tag>[,tag:<tag>[,tag:<tag>]]]

 

#dhcp-proxy[=<ip addr>]......

 

##############################################################################

# 不分配匹配这些 tag:<tag> 的 DHCP 请求。

#dhcp-ignore=tag:<tag>[,tag:<tag>]

#dhcp-ignore-names[=tag:<tag>[,tag:<tag>]]

#dhcp-generate-names=tag:<tag>[,tag:<tag>]

# IPv4 only 使用广播与匹配 tag:<tag> 的客户端通信。一般用于兼容老的 BOOT 客户端。

#dhcp-broadcast[=tag:<tag>[,tag:<tag>]] 

 

##############################################################################

# IPv4 only 设置 DHCP 服务器返回的 BOOTP 选项，

# <servername> <server address> 可选，

# 如果未设置服务器名称将设为空，服务器地址设为 dnsmasq 的 IP 地址。

# 如果指定了多个 tag:<tag> 必须同时匹配才行。

# 如果指定 <tftp_servername> 将按照 /etc/hosts 中对应的 IP 地址进行轮询负载均衡。  

#dhcp-boot=[tag:<tag>,]<filename>,[<servername>[,<server address>|<tftp_servername>]]

# 根据不同的类型使用不同的选项。

# 使用示例：

#        dhcp-match=set:EFI_x86-64,option:client-arch,9

#        dhcp-boot=tag:EFI_x86-64,uefi/grubx64.efi

#        #dhcp-match=set:EFI_Xscale,option:client-arch,8

#        #dhcp-boot=tag:EFI_Xscale,uefi/grubx64.efi

#        #dhcp-match=set:EFI_BC,option:client-arch,7

#        #dhcp-boot=tag:EFI_BC,uefi/grubx64.efi

#        #dhcp-match=set:EFI_IA32,option:client-arch,6

#        #dhcp-boot=tag:EFI_IA32,uefi/grubx64.efi

#        #dhcp-match=set:Intel_Lean_Client,option:client-arch,5

#        #dhcp-boot=tag:Intel_Lean_Client,uefi/grubx64.efi

#        #dhcp-match=set:Arc_x86,option:client-arch,4

#        #dhcp-boot=tag:Arc_x86,uefi/grubx64.efi

#        #dhcp-match=set:DEC_Alpha,option:client-arch,3

#        #dhcp-boot=tag:DEC_Alpha,uefi/grubx64.efi

#        #dhcp-match=set:EFI_Itanium,option:client-arch,2

#        #dhcp-boot=tag:EFI_Itanium,uefi/grubx64.efi

#        #dhcp-match=set:NEC/PC98,option:client-arch,1

#        #dhcp-boot=tag:NEC/PC98,uefi/grubx64.efi

#        dhcp-match=set:Intel_x86PC,option:client-arch,0

#        dhcp-boot=tag:Intel_x86PC,pxelinux.0

 

##############################################################################

# DHCP 使用客户端的 MAC 地址的哈希值为客户端分配 IP 地址，

# 通常情况下即使客户端使自己的租约到期，客户端的 IP 地址仍将长期保持稳定。

# 在默认模式下，IP 地址是随机分配的。

# 启用 dhcp-sequential-ip 选项将按顺序分配 IP 地址。

# 在顺序分配模式下，客户端使租约到期更像是仅仅移动一下 IP 地址。

# 在通常情况下不建议使用这种方式。

#dhcp-sequential-ip

 

##############################################################################

# 多数情况下我们使用 PXE，只是简单的允许 PXE 客户端获取 IP 地址，

# 然后 PXE 客户端下载 dhcp-boot 选项指定的文件并执行，也就是 BOOTP 的方式。

# 然而在有适当配置的 DHCP 服务器支持的情况下，PXE 系统能够实现更复杂的功能。

# pxe-service 选项可指定 PXE 环境的启动菜单。

# 为不同的类型系统设定不同的启动菜单，并且覆盖 dhcp-boot 选项。

# <CSA> 为客户端系统类型：x86PC, PC98, IA64_EFI, Alpha, Arc_x86, Intel_Lean_Client, 

# IA32_EFI, X86-64_EFI, Xscale_EFI, BC_EFI, ARM32_EFI 和 ARM64_EFI，其他类型可能为一个整数。

# <basename> 引导 PXE 客户端使用 tftp 从 <server address> 或者 <server_name> 下载文件。

#     注意："layer" 后缀 (通常是 ".0") 由 PXE 提供，也就是 PXE 客户端默认在文件名附加 .0 后缀。

#     示例：pxe-service=x86PC, "Install Linux", pxelinux         （读取 pxelinux.0 文件并执行）

#           pxe-service=x86PC, "Install Linux", pxelinux, 1.2.3.4（不适用于老的PXE）

#     <bootservicetype> 整数，PXE 客户端将通过广播或者通过 <server address> 

#           或者 <server_name> 搜索对应类型的适合的启动服务。。

#     示例：pxe-service=x86PC, "Install windows from RIS server", 1

#           pxe-service=x86PC, "Install windows from RIS server", 1, 1.2.3.4

#     未指定 <basename>、<bootservicetype> 或者 <bootservicetype> 为 “0”，将从本地启动。

#     示例：pxe-service=x86PC, "Boot from local disk"

#           pxe-service=x86PC, "Boot from local disk", 0

# 如果指定 <server_name> 将按照 /etc/hosts 中对应的 IP 地址进行轮询负载均衡。  

#pxe-service=[tag:<tag>,]<CSA>,<menu text>[,<basename>|<bootservicetype>][,<server address>|<server_name>]

# 在 PXE 启动后弹出提示，<prompt> 为提示内容，<timeout> 为超时时间，为 0 则立即执行。

# 如果未指定此选项，在有多个启动选项的情况下等待用户选择，不会超时。

#pxe-prompt=[tag:<tag>,]<prompt>[,<timeout>]

# 根据不同的类型使用不同的菜单，使用示例：

#        #pxe-prompt="What system shall I netboot?", 120

#        # or with timeout before first available action is taken:

#        pxe-prompt="Press F8 or Enter key for menu.", 60

#        pxe-service=x86PC, "Now in x86PC (BIOS mode), boot from local", 0

#        pxe-service=x86PC, "Now in x86PC (BIOS mode)", pxelinux

#        pxe-service=PC98, "Now in PC98 mode", PC98

#        pxe-service=IA64_EFI, "Now in IA64_EFI mode", IA64_EFI

#        pxe-service=Alpha, "Now in Alpha mode", Alpha

#        pxe-service=Arc_x86, "Now in Arc_x86 mode", Arc_x86

#        pxe-service=Intel_Lean_Client, "Now in Intel_Lean_Client mode", Intel_Lean_Client

#        pxe-service=IA32_EFI, "Now in IA32_EFI mode", IA32_EFI

#        pxe-service=X86-64_EFI, "Now in X86-64_EFI (UEFI mode), boot from local", 0

#        pxe-service=X86-64_EFI, "Now in X86-64_EFI (UEFI mode)", grub/grub-x86_64.efi

#        pxe-service=Xscale_EFI, "Now in Xscale_EFI mode", Xscale_EFI

#        pxe-service=BC_EFI, "Now in BC_EFI mode", BC_EFI

#        # CentOS7 系统不支持下列两个选项

#        #pxe-service=ARM32_EFI,"Now in ARM32_EFI mode",ARM32_EFI

#        #pxe-service=ARM64_EFI,"Now in ARM64_EFI mode",ARM64_EFI

 

##############################################################################

# 默认为150，即最多分配150个ip地址出去，最大1000个ip

#dhcp-lease-max=150

# (IPv4 only) 指定DHCP端口，默认为67和68。如果不指定则为1067和1068，单指定一个，第二个加1

#dhcp-alternate-port[=<server port>[,<client port>]]

# 谨慎使用此选项，避免 IP 地址浪费。(IPv4 only) 允许动态分配 IP 地址给 BOOTP 客户端。

# 注意：BOOTP 客户端获取的 IP 地址是永久的，将无法再次分配给其他客户端。

#bootp-dynamic[=<network-id>[,<network-id>]]

# 谨慎使用此选项。

# 默认情况下 DHCP 服务器使用 ping 的方式进行确保 IP 未被使用的情况下将 IP 地址分配出去。

# 启用此选项将不使用 ping 进行确认。

#no-ping

 

##############################################################################

# 记录额外的 dhcp 日志，记录所有发送给 DHCP 客户端的选项（option）以及标签（tag）信息

#log-dhcp

# 禁止记录日常操作日志，错误日志仍然记录。启用 log-dhcp 将覆盖下列选项。

#quiet-dhcp

#quiet-dhcp6

#quiet-ra

 

# 修改 DHCP 默认租约文件路径，默认情况下无需修改

#dhcp-leasefile=/var/lib/dnsmasq/dnsmasq.leases

# (IPv6 only)

#dhcp-duid=<enterprise-id>,<uid>

 

##############################################################################

#dhcp-script=<path>

#dhcp-luascript=<path>

#dhcp-scriptuser=root

#script-arp

#leasefile-ro

 

#bridge-interface=<interface>,<alias>[,<alias>]

 

##############################################################################

# 给 DHCP 服务器指定 domain 域名信息，也可以给对应的 IP 地址池指定域名。

#     直接指定域名

#     示例：domain=thekelleys.org.uk

#     子网对应的域名

#     示例：domain=wireless.thekelleys.org.uk,192.168.2.0/24

#     ip范围对应的域名

#     示例：domain=reserved.thekelleys.org.uk,192.68.3.100,192.168.3.200

#domain=<domain>[,<address range>[,local]]

# 在默认情况下 dnsmasq 插入普通的客户端主机名到 DNS 中。

# 在这种情况下主机名必须唯一，即使两个客户端具有不同的域名后缀。

# 如果第二个客户端使用了相同的主机名，DNS 查询将自动更新为第二个客户端的 IP 地址。

# 如果设置了 dhcp-fqdn 选项，普通的主机名将不再插入到 DNS 中去，

# 仅允许合格的具有域名后缀的主机名插入到 DNS 服务器中。

# 指定此选项需同时指定不含 <address range> 地址范围的 domain 选项。

#dhcp-fqdn

# 通常情况下分配 DHCP 租约后，dnsmasq 设置 FQDN 选项告诉客户端不要尝试 DDNS 更新主机名与 IP 地址。

# 这是因为  name-IP 已自动添加到 dnsmasq 的 DNS 视图中的。

# 设置此选项将允许客户端 DDNS 更新，

# 在 windows 下允许客户端更新 windows AD 服务器是非常有用的。

# 参看  RFC 4702 。

#dhcp-client-update

 

#enable-ra

#ra-param=<interface>,[high|low],[[<ra-interval>],<router lifetime>]

 

 

##############################################################################

#

#        TFTP 选项

#

##############################################################################

# 对于绝大多数的配置，仅需指定 enable-tftp 和 tftp-root 选项即可。

# 是否启用内置的 tftp 服务器，可以指定多个逗号分隔的网络接口

#enable-tftp[=<interface>[,<interface>]]

#enable-tftp

#enable-tftp=enp3s0,lo

# 指定 tftp 的根目录，也就是寻找传输文件时使用的相对路径，可以附加接口，

#tftp-root=<directory>[,<interface>]

#tftp-root=/var/lib/tftpboot/

# 如果取消注释，那么即使指定的 tftp-root 无法访问，仍然启动 tftp 服务。

#tftp-no-fail

# 附加客户端的 IP 地址作为文件路径。此选项仅在正确设置了 tftp-root 的情况下可用，

# 示例：如果 tftp-root=/tftp，客户端为 192.168.1.15 请求 myfile.txt 文件时，

# 将优先请求 /tftp/192.168.1.15/myfile.txt 文件， 其次是 /tftp/myfile.txt 文件。

# 感觉没什么用。

#tftp-unique-root

# 启用安全模式，启用此选项，仅允许 tftp 进程访问属主为自己的文件。

# 不启用此选项，允许访问所有 tftp 进程属主可读取的文件。

# 如果 dnsmasq 是以 root 用户运行，tftp-secure 选项将允许访问全局可读的文件。

# 一般情况下不推荐以 root 用户运行 dnsmasq。

# 在指定了 tftp-root 的情况下并不是很重要。

#tftp-secure

# 将所有文件请求转换为小写。对于 Windows 客户端来说非常有用，建议开启此项。

# 注意：dnsmasq 的 TFTP 服务器总是将文件路径中的“\”转换为“/”。

#tftp-lowercase

# 允许最大的连接数，默认为 50 。

# 如果将连接数设置的很大，需注意每个进程的最大文件描述符限制，详见文档手册。

#tftp-max=<connections>

#tftp-max=50

# 设置传输时的 MTU 值，建议不设置或按需设置。

# 如果设定的值大于网络接口的 MTU 值，将按照网络接口的 MTU 值自动分片传输（不推荐）。

#tftp-mtu=<mtu size>

# 停止 tftp 服务器与客户端协商 "blocksize" 选项。启用后，防止一些古怪的客户端出问题。

#tftp-no-blocksize

# 指定 tftp 的连接端口的范围，方便防火墙部署。

# tftp 侦听在 69/udp ，连接端口默认是由系统自动分配的，

# 非 root 用户运行时指定的连接端口号需大于 1025 最大 65535。

#tftp-port-range=<start>,<end>

###############################################################################

#conf-dir=<directory>[,<file-extension>......]

#conf-file=/etc/dnsmasq.more.conf

conf-dir=/etc/dnsmasq.d

#servers-file=<file>
```

这里使用的是 CentOS 7.x 环境，如果需要编译安装可以直接到[官方网站](http://www.thekelleys.org.uk/dnsmasq/)选择版本编译。

安装完毕后，可以通过 dnsmasq -v 命令查看版本，有版本号出来就代表安装上了

修改配置文件前一定要先备份

```
[root@localhost ~]# yum install -y dnsmasq
```

```
[root@localhost ~]# echo 'resolv-file=/etc/dnsmasq.d/resolv.dnsmasq.conf'>> /etc/dnsmasq.conf

`表示dnsmasq 会从这个指定的文件中寻找上游dns服务器。`

[root@localhost ~]# echo 'addn-hosts=/etc/dnsmasq.d/dnsmasq.hosts' >> /etc/dnsmasq.conf

`添加读取额外的 hosts 文件路径，可以多次指定`
[root@localhost ~]# vim /etc/dnsmasq.conf

strict-order      取消这一行的注释，表示严格按照resolv.conf中的顺序进行查找

listen-address=127.0.0.1    添加监听地址这个 dnsmasq 本机自己使用有效。

listen-address=192.168.115.120   用此主机为局域网提供默认 DNS，写本机的局域网IP

listen-address=127.0.0.1,192.168.115.120   多个ip地址设置

如果想允许所有的用户使用你的DNS解析服务器，把listen-address去掉即可。
```

resolv.dnsmasq.conf 中设置的是真正的 Nameserver，可以填写各大商家提供的免费 DNS 地址。

```
[root@localhost ~]# echo 'nameserver 127.0.0.1' > /etc/resolv.conf

[root@localhost ~]# cp /etc/resolv.conf  /etc/dnsmasq.d/resolv.dnsmasq.conf

[root@localhost ~]# echo 'nameserver 8.8.8.8' >>/etc/dnsmasq.d/resolv.dnsmasq.conf

[root@localhost ~]# echo 'nameserver 192.168.115.120' >>/etc/dnsmasq.d/resolv.dnsmasq.conf

[root@localhost ~]# cp /etc/hosts  /etc/dnsmasq.d/dnsmasq.hosts
```

```
[root@localhost ~]# systemctl restart dnsmasq    重启dnsmasq服务

[root@localhost ~]# systemctl enable dnsmasq    设置成开机自启动

[root@localhost ~]# netstat -antp|grep 53        查看端口是否启动成功
```

将 Dnsmasq 作为本地 DNS 服务器使用，直接修改电脑的本地 DNS 的 IP 地址即可

[T1](https://e-mailky.github.io/images/others/1517120675792679.png)

输入一个不存在的域名，正常的情况下浏览器是显示无法连接，DNS 劫持会跳转到一个广告页面。先随便 nslookup 一个不存在的域名，看看 ISP 商劫持的 IP 地址。

接着编辑 / etc/dnsmasq.conf 文件，把 bogus-nxdomain=’劫持 IP’　加入进去，后面的 IP 是刚刚查询到的 DNS 劫持 IP 地址。

重启 dnsmasq，再尝试打开不存在的域名，这时浏览器就会显示正常的无法连接页面了。

打开 / etc/dnsmasq.conf 文件，server = 添加指定的 DNS，例如国内外不同的网站使用不同的 DNS。

```
[root@localhost ~]# dig www.taobao.com

.........................................................省略若干

;; Query time: 77 mse    第一次查询没有缓存，时间77

;; SERVER: 127.0.0.1#53(127.0.0.1)

;; WHEN: 二 1月 16 13:09:32 CST 2018

;; MSG SIZE  rcvd: 120

 

[root@localhost ~]# dig www.taobao.com

.........................................................省略若干

;; Query time: 0 msec    第二次再次查询，时间为0

;; SERVER: 127.0.0.1#53(127.0.0.1)

;; WHEN: 二 1月 16 13:11:39 CST 2018

;; MSG SIZE  rcvd: 123
```

将广告的 URL 指定 127.0.0.1 这个 IP，就可以将网页上讨厌的广告给去掉了。

```
[root@localhost ~]# vim /etc/dnsmasq.conf

国内指定DNS

server=/cn/114.114.114.114

server=/taobao.com/114.114.114.114

server=/taobaocdn.com/114.114.114.114

国外指定DNS

server=/google.com/223.5.5.5
```

这个功能可以让你控制一些网站的访问，非法的 DNS 就经常把一些正规的网站解析到不正确 IP 上。

```
[root@localhost ~]# vim /etc/dnsmasq.conf

address=/ad.youku.com/127.0.0.1

address=/ad.iqiyi.com/127.0.0.1
```

首先将局域网中的所有的设备的本地 DNS 设置为已经安装 Dnsmasq 的服务器 IP 地址。然后修改已经安装 Dnsmasq 的服务器 Hosts 文件：/etc/hosts，指定域名到特定的 IP 中。

例如：想让局域网中的所有用户访问 www.abc.com 时跳转到 192.168.115.100，添加’192.168.115.100 www.abc.com’ 到 Hosts 文件中既可，整个过程也可以说是 “DNS 劫持”。

```
[root@localhost ~]# vim /etc/dnsmasq.conf
address=/freehao123.com/123.123.123.123
```

[dnsmasq 详解及配置](http://blog.51cto.com/longlei/2065967)