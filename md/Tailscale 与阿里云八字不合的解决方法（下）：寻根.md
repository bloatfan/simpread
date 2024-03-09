> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/653295049)

本文首发于我的博客：[Tailscale 与阿里云八字不合的解决方法（2）](https://blog.baldcoder.top/articles/resolving-the-incompatibility-between-tailscale-and-alibaba-cloud-episode-2/)

> 下面的白名单方法虽然似乎相对安全，但是更加麻烦，要加入的 IP 太多了，最后我还是直接禁掉 Tailscale 的 Drop 规则了。

疑案追踪
----

[书接上文](https://zhuanlan.zhihu.com/p/653276369)。

上文说到，不安全的做法是直接禁掉 Tailscale 的 Drop 规则。那么有没有安全的做法呢？当然就是需要的时候才禁掉规则啦：比如，需要 `sudo apt-get update` 等安装软件的时候才临时禁用掉，之后再恢复。

但是这样有个问题：Tailscale 的 Drop 规则把 WordPress 也影响了。WordPress 时时刻刻都在运行，那临时禁止规则就变成一句空谈了。

那么问题就变成了：有没有办法能保留规则的情况下，WordPress 仍然正常运行呢？这就需要探究这场疑案的根源了。

谁是幕后真凶？
-------

怎么查看是什么拖慢了 WordPress 速度呢？这个时候又要请 [Query Monitor](https://wordpress.org/plugins/query-monitor/) 出山了。刷新了几次后，抓到了直接凶手：一些 HTTP 请求等待超时了。

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-457f03fa7176391128de5a5fa4adb862_r.jpg)

现在已知：

*   Tailscale 的 Drop 规则在 Azure 上工作良好。
*   阿里云使用 `100.64.0.0/10` 保留网段提供了内网服务，而 Tailscale 的 Drop 规则丢弃了来自这个网段的所有流量，导致阿里云的服务工作不正常。
*   删掉 Tailscale 的 Drop 规则，阿里云和 WordPress 都工作正常了。

很容易推理出一个结论：WordPress 在什么地方被阿里云的内网服务影响了。但是我想破头都想不到会有什么联系？？？

走弯路
---

当时想到，会不会 nginx 同时监听了公网 IP 和 Tailscale IP 导致的问题？如果不监听 Tailscale IP 会不会恢复？

于是 `sudo ifconfig` 得到网址信息，然后让 nginx 只监听私有 IP `172.x.x.x`。但是失败了，不是这个原因。即使卸载了 tailscale，只要自己加上那个 Drop 规则就不行。

> 阿里云、Azure 等很多云服务商给 VPS 分配 IP 时会分配一个公网 IP 和一个内网 IP，但是 `sudo ifconfig` 只能看到内网 IP 没有公网 IP。这是因为云服务厂商会做一个公网 IP 到内网 IP 的 “转换”，所以 VPS 看不见公网 IP。因此，要显式指定监听公网 IP，只需要指定内网 IP 即可。  
> 参考：  
> [1] [No public IP when running ifconfig on publicly accessible Azure instance created via Terraform - Stack Overflow](https://stackoverflow.com/questions/62996199/no-public-ip-when-running-ifconfig-on-publicly-accessible-azure-instance-created)  
> [2] [阿里云的公网 IP 地址，在服务器端不能绑定，是怎么回事 - CSDN 社区](https://bbs.csdn.net/topics/392271886)

交警查车！
-----

没办法，只能祭出查 log 大法：看看 Drop 规则到底拦掉了什么流量。为此，首先彻底卸掉 tailscale，然后添加两条规则：

```
sudo iptables -A INPUT -s 100.64.0.0/10 -j LOG --log-level debug  --log-prefix='[Tailscale Netfilter Blocked] '
sudo iptables -A INPUT -s 100.64.0.0/10 -j DROP
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-ba1504e5e2b60adce5ad7214d2a60c6a_r.jpg)

`sudo iptables -A INPUT -s 100.64.0.0/10 -j LOG --log-level debug --log-prefix='[Tailscale Netfilter Blocked] '`

*   `iptables`: Linux 上的防火墙规则管理工具。
*   `-A INPUT`: 表示将规则添加到 INPUT 链中，INPUT 链用于处理接收到的数据包。
*   `-s 100.64.0.0/10`: 源 IP 地址匹配条件，这里是匹配源 IP 地址在 100.64.0.0/10 子网范围内的数据包。
*   `-j LOG`: 表示如果数据包匹配了前面的条件，将其记录到系统日志，但不中断数据包流程。
*   `--log-level debug`: 设置日志记录级别为 debug，这将记录详细的调试信息。
*   `--log-prefix='[Tailscale Netfilter Blocked] '`: 设置日志前缀为 "[Tailscale Netfilter Blocked]"，这有助于识别日志的来源。

`sudo iptables -A INPUT -s 100.64.0.0/10 -j DROP`

*   `-j DROP`: 如果数据包匹配了前述条件，将其丢弃，即阻止其进一步处理。

即在日志中记录匹配规则 `100.64.0.0/10` 的流量，然后丢弃掉。

然后查看日志：

```
sudo tail -f /var/log/kern.log | grep --line-buffered "\[Tailscale Netfilter Blocked\]"
```

*   `tail -f /var/log/kern.log`: `tail` 命令用于显示文件的末尾内容。选项 `-f` 指示 `tail` 在文件更新时继续显示新增的内容，实现了实时监视。`/var/log/kern.log` 是一个系统内核日志文件，它记录了内核级别的事件和信息。
*   `|`: 管道操作符，将一个命令的输出作为另一个命令的输入。
*   `grep --line-buffered "\[Tailscale Netfilter Blocked\]"`: `grep` 命令用于在文本中搜索匹配指定模式的行。`--line-buffered` 选项使 `grep` 在每一行被输出时立即刷新输出，以便实时显示日志。`\[Tailscale Netfilter Blocked\]` 是一个正则表达式，用于匹配包含 "[Tailscale Netfilter Blocked]" 的日志行。

即以实时方式监视系统内核日志文件，过滤显示那些包含 "[Tailscale Netfilter Blocked]" 前缀的日志行。

然后执行 `sudo apt-get update`，刷新博客网页等操作。日志文件输出：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-06cb0c7ccb5348df161ae61512c9ccbb_r.jpg)

这 `100.100.2.136` 和 `100.100.2.138` 是何方神圣？怎么来来回回都是这两个？

一必应，原来这两个是阿里云内网 DNS 服务！水落石出，原来是 DNS 解析失败了！

解决方法
----

这个解决方法很简单：将内网 DNS 设置为外网 DNS 就好了，之后 WordPress 成功恢复正常。阿里云官方文档：[如何在 Linux 实例中自定义配置 DNS_云服务器 ECS - 阿里云帮助中心](https://help.aliyun.com/zh/ecs/how-do-i-customize-the-dns-settings-of-a-linux-instance#2efaa5616ba9f)

然后删掉为了查 log 而设立的临时规则：

```
sudo iptables -D INPUT 1
sudo iptables -D INPUT 1
sudo iptables -L --line-numbers
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/v2-3c5f94d39194a9177022152b2811f160_r.jpg)

重新安装 tailscale。然后输入

```
sudo iptables -N ts-input-whitelist
sudo iptables -I ts-input-whitelist -s 100.64.0.0/10 -j LOG --log-level debug --log-prefix='[Tailscale Netfilter Blocked] '
# 手动添加白名单
sudo iptables -I ts-input-whitelist -s 100.100.45.106 -j ACCEPT
sudo iptables -I ts-input-whitelist -s 100.118.28.50 -j ACCEPT
# 添加更多的阿里云内网 IP
sudo iptables -I INPUT -j ts-input-whitelist
sudo iptables -L --line-numbers
```

大致意思是：手动添加放行白名单，最后记录不在白名单里被拦截的 IP。

`100.100.45.106` 是阿里云杭州的 Agent。各个区域的 Agent IP [如何为云助手客户端所在实例配置安全组规则_云服务器 ECS - 阿里云帮助中心](https://help.aliyun.com/zh/ecs/user-guide/configure-network-permissions-for-the-cloud-assistant-agent)

之后只要使用

```
sudo tail -f /var/log/kern.log | grep --line-buffered "\[Tailscale Netfilter Blocked\]"
```

就能看到不在白名单里被拦截的阿里云内网 IP，然后加到 `ts-input-whitelist` 里就行了。

**以上 iptables 规则重启服务器后会丢失，需要重新输入，或者自行搜索持久化的方法。**