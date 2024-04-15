> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.niekun.net](https://blog.niekun.net/archives/1869.html)

> dnsmasq 提供 DNS 缓存 / 查询服务和 DHCP(Dynamic Host Configuration Protocol) 服务等功能，用来管理本地局域网络系统。内置于常见的 Linux...

dnsmasq 提供 DNS 缓存 / 查询服务和 DHCP(Dynamic Host Configuration Protocol) 服务等功能，用来管理本地局域网络系统。内置于常见的 Linux 分发版，openWrt，macOS 系统中。

### 安装

直接使用包管理器安装：

```
apt install dnsmasq
```

查询版本：

```
dnsmasq -v
```

信息里 **Compile time options** 可以看到当前安装版本支持的选项功能 ，如：`ipset`

```
root@OpenWrt:/etc# dnsmasq -v
Dnsmasq version 2.80  Copyright (c) 2000-2018 Simon Kelley
Compile time options: IPv6 GNU-getopt no-DBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC no-ID loop-detect inotify dumpfile

This software comes with ABSOLUTELY NO WARRANTY.
Dnsmasq is free software, and you are welcome to redistribute it
under the terms of the GNU General Public License, version 2 or 3.
```

启动服务：

```
systemctl start dnsmasq
```

服务启动后，会监听本地或局域网内的 DNS 请求并根据配置规则进行处理。

### DNS 服务

#### Linux DNS 请求处理流程

`test.com` -> `/etc/hosts` -> `/etc/resolv.conf` -> `dnsmasq`

以上每个过程中只要得到了解析的 IP 地址则直接结束剩下的处理过程。

`/etc/hosts` 文件是 Linux 系统默认的 hosts 文件，一般发起的 DNS 请求会首先查询此 hosts 文件，如果没有匹配上则从 `/etc/resolv.conf` 文件找 DNS 服务器进行进一步查询。

`/etc/resolv.conf` 文件是 linux 系统的默认 dns 配置文件，一般情况下里面定义的域名服务器地址为本地：127.0.0.1 地址，由于 dnsmasq 默认监听本地及局域网 53 端口，则 DNS 请求就会传入 dnsmasq 进行进一步解析。

下面介绍 hosts 文件和 resolv 文件的意义。

#### hosts

hosts 文件主要用来指定某个域名的解析 IP，通常用来处理局域网设备的域名解析到对应设备 IP。一般情况下局域网设备设置的域名不能在公共 DNS 服务器进行解析。系统默认的 hosts 文件地址：`/etc/hosts`。

文件格式：

```
127.0.0.1 localhost
192.168.1.123 test1.home.lan
192.168.1.124 test2.home.lan
```

使用 hosts 文件也可以用来进行域名欺骗，实现广告屏蔽等功能，比如我想要屏蔽 360 的所有访问：

```
127.0.0.1 360.com
```

dnsmasq 可以选择使用自定义的 hosts 文件。

#### resolv.conf

resolv.conf 文件定义了 DNS 服务器地址，dns 请求会转发到设置的地址上。可以指定多个服务器进行顺序查询直到解析到 IP 地址，指定为本地地址 127.0.0.1 会转发到本地 dnsmasq 进行处理。`/etc/resolv.conf` 是系统默认解析服务器配置文件。

文件格式：

```
nameserver 127.0.0.1
nameserver 192.168.1.1
nameserver 114.114.114.114
```

系统默认的 `/etc/resolv.conf` 文件在每次系统启动会根据 DHCP 分配情况自动生成，一般指向本地地址或局域网网关。dnsmasq 可以配置自定义的 resolv 文件可以设置公网的 DNS 解析服务器，也就是上游 DNS 服务器。

#### dnsmasq 处理流程简介

`dnsmasq` -> `hosts.dnsmasq` -> `/etc/dnsmasq.conf` / `dnsmasq.conf` -> `resolv.dnsmasq.conf`

DNS 请求传入 dnsmasq 后通过其配置文件来进行 DNS 查询，首先查询 hosts 文件，如果设置了自定义 hosts 文件和系统默认 hosts 一起查询，没有匹配到的话就进入 conf 配置文件内部的 server 及 address 项进行匹配，如果依然没有结果则查询 relolv 自定义配置文件定义的上游 DNS 服务器。

`/etc/dnsmasq.d` 文件夹和 `/etc/dnsmasq.conf` 文件是 dnsmasq 的配置路径，可以设置监听地址，自定义 hosts.dnsmasq 文件地址，自定义 resolv.dnsmasq.conf 文件地址，也可以在文件内直接指定某域名使用的 DNS 解析服务器等。

`/etc/dnsmasq.d` 文件夹可以存放用户自定义的 dnsmasq 配置文件，效果等同于直接写入 dnsmasq.conf 文件内，方便整理自定义规则。

#### dnsmasq 配置文件

下面介绍 dnsmasq 配置常用的语句：

```
# 监听地址：
# 如果只写 127.0.0.1 则只处理本机的 DNS 解析，不写这句默认监听所有网口
listen-address=127.0.0.1,192.168.8.132

# 指定自定义 hosts 文件：
addn-hosts=/etc/hosts.dnsmasq

# 指定上游 DNS 服务列表的配置文件
resolv-file=/etc/resolv.dnsmasq.conf

# 按照 DNS 列表一个个查询，否则将请求发送到所有 DNS 服务器
strict-order

# 表示对下面设置的所有 server 发起查询请求，选择响应最快的服务器的结果
all-servers

# 指定默认查询的上游服务器
server=8.8.8.8
server=114.114.114.114

# 指定 .cn 的域名全部通过 114.114.114.114 这台国内DNS服务器来解析
server=/cn/114.114.114.114

# 给 *.apple.com 和 taobao.com 使用专用的 DNS
server=/taobao.com/223.5.5.5
server=/.apple.com/223.6.6.6

# 增加一个域名，强制解析到所指定的地址上，dns 欺骗
address=/360.com/127.0.0.1

# 加载外部配置文件，如：特定目录下的扩展名为 conf 的文件
conf-dir=/etc/config/dnsmasq, *.conf

# 设置DNS缓存大小(单位：DNS解析条数)
cache-size=500

# 存储域名解析的 IP 地址结果存储到 saveresult 的 ipset 结果中，可以交给iptables识别和转发
ipset=/test.com/saveresult
```

### DHCP 服务

待整理。。。

### 升级 dnsmasq-full

openWrt 默认安装的 dnsmasq 缺少一些选项功能，如：ipset，可以安装 dnsmasq-full 来实现更多功能，由于 dnsmasq 管理着域名解析工作，卸载 dnsmasq 后会导致无法正确解析域名从无法联网。有 2 种方法避免这种情况：

*   修改 `resolv.conf` 文件手动指定 DNS 服务器
*   提前下载好 dnsmasq-full 安装文件。

#### 手动指定 DNS 解析地址

修改 `/etc/resolv.conf` 文件，指定上游 DNS 服务器：

```
nameserver 114.114.114.114
```

卸载及安装新程序：

```
opkg remove dnsmasq && opkg install dnsmasq-full
opkg install ipset
```

#### 提前下载安装包

```
# 下载安装包
opkg download dnsmasq-full
# 查看下载的安装包名称：
ls
# 尝试安装，会提示失败，但可以安装好需要的依赖包
opkg install dnsmasq-full
# 删除原 dnsmasq
opkg remove dnsmasq
# 安装下载好的包
opkg install dnsmasq-full_2.80-15_x86_64.ipk
# 安装完成后可以删除安装包文件
rm dnsmasq-full_2.80-15_x86_64.ipk 

记得 ipset 也需要单独安装：
    opkg install ipset
```

### 参考链接

[一文玩转 V2ray 透明代理](https://www.solarck.com/openwrt-v2ray.html)  
[Dnsmasq 介绍与使用](http://www.enkichen.com/2017/05/23/dnsmasq-introduce/)  
[Dnsmasq+ipset+iptables 基于域名的流量管理](https://blog.csdn.net/lvshaorong/article/details/52981169)

标签：无