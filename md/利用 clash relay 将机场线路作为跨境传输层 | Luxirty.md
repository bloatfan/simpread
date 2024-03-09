> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [luxirty.com](https://luxirty.com/article/01815b7d-073d-48ef-9528-7b4ad033f8d0)

> 这是一个由 NotionNext 生成的站点

[](#b9d44f5b56e34bee8b55ece272329be1 "序言")**序言**
------------------------------------------------

关于代理，一直以来分成两派：自建 & 机场，但是各有优缺点：

<table><tbody><tr><td><p>ㅤ</p></td><td><p><b>自建</b></p></td><td><p><b>机场</b></p></td></tr><tr><td><p>优点</p></td><td><p>保护隐私、自定义程度高</p></td><td><p>无需维护、速度较快</p></td></tr><tr><td><p>缺点</p></td><td><p>不稳定、慢、会被阻断</p></td><td><p>有日志和审计规则、威胁隐私</p></td></tr></tbody></table>

各举两个例子：

自建：ip 被阻断，在紧急需要的时刻却在焦头烂额地换新配置、换 ip 、换端口。

机场：浏览历史被完整记录、某些网站被设置为黑名单、某些端口被禁止使用。

那么有没有一种兼顾了速度、稳定、隐私的办法呢？

有的，答案就是 **Clash Relay Group + Prxoy Provider + LoadBlance/Fallback**。

* * *

[](#df0d3725174444f4bf9b6f43e3a6df66 "原理")**原理**
------------------------------------------------

Clash 的 relay 分组会把组内节点串成一个代理链，只需要将机场节点作为前置节点，⾃建节点作为后置节点，相当于只把机场作为跨境隧道，最终落地解密出真实流量的还是我们的⾃建节点。

`你的电脑<->机场<->⾃建节点<->交友⽹站`

这样⼀来，机场不知道我们实际访问的⽹站，而⾃建节点只能看到机场的 ip，链路上没有任何一环同时得知访问来源和最终目的。

本质上，Apple 公司推出的 iCloud private relay 也是这样的思路。

这样就实现了既享受机场的便捷，又保护了⾃⼰的隐私，当然其实还有其他非常多的好处，例如无比的稳定性。

* * *

[](#fc080062384c43aea71be4b14b2d40f1 "好！怎么实现呢？")**好！怎么实现呢？**
------------------------------------------------------------

⾸先了解⼀下 Clash 的配置⽂件，我们要关注的主要是下面四个字段：

```
proxies:
proxy-providers:
proxy-groups:
rules:
```

如果你不知道怎么写其余部分，完整的配置可以去 github 的 wiki ⾥边看，或者直接找⼀份来⾃⼰改。

### [](#437305ca7f6c4000a3bcba06d8500fcd "第⼀步，利用 proxy provider 将机场节点加入到配置中")**第⼀步，利用 proxy provider 将机场节点加入到配置中**

随便找⼀个订阅转换的⽹站，⽐如说 sublink.dev , 把订阅链接放进去，客户端选 Clash ，然后点**进阶选项，在左下⾓找到 “转换为 NodeList” 勾选上**，最后点⽣成订阅链接。

然后把链接复制出来这样填：

```
proxy-providers:
	jichang:
		type: http
		path: ./jichang.yaml
		url: # ⽣成的订阅链接填在这⾥！
		interval: 3600
		health-check:
			enable: true
			url: https://www.gstatic.com/generate_204
			# 使⽤https 避免⽆良⽼板伪造延迟！
			interval: 300
```

有些机场有好多地⽅的节点，为了我们⽅便写负载均衡和故障转移，可以把我们⾃建节点所在地区的节点单独过滤出来。

⽐如我⾃建的节点在美国，就像这样写（在实践中我们发现使用 file 可能会导致在路由器上出现问题，因此你可以一样使用 http 类型从网络获取节点）：

```
美国的节点:
		type: file # 这⾥是 file了！因为不需要再从⽹络获取
		path: ./jichang.yaml #这⾥路径就填和上⾯⼀样的
		filter: "US|美国" #正则表达式
		health-check:
			enable: true
			url: https://www.gstatic.com/generate_204
			# 使⽤https 防⽌⽆良⽼板伪造延迟！
			interval: 300
```

最后还需要⽤⼀个分组来包含这些节点，在 groups 下⾯这样写：

```
proxy-groups:
  - name: 机场的分组
    type: select
    use: # provider 的节点要放在 use 下⾯
      - jichang
    proxies:
      - DIRECT # use 是可以和普通的节点混⽤的！
```

### [](#aa282273500c4d0cb8599b561fb2b393 "第⼆步，添加自建节点")**第⼆步，添加自建节点**

接下来把⾃建节点的配置写到 proxies ⾥⾯，这个就不需要多说了吧。

```
proxies:
  - name: "⾃建的节点"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
```

提⽰：由于跨境段是机场线路负责，因此⾃建节点可以不考虑混淆，使用 vmess+websocket+tls 之类复杂的协议是不必要的，shadowsocks AEAD 即可，它的 0 rtt 可以带来最佳体验。

提示 2: 如果你有较多的自建节点，为了更加方便，其实你可以把自建节点同样编写为一个 provider，使用与机场节点一致的方式导入。

### [](#8a58090ea453470db251ae1b81561613 "最后⼀步，编写 Relay Group")**最后⼀步，编写 Relay Group**

只需要把 group 的 type 写成 relay ，然后把机场的节点和自建节点**按顺序**写到 proxies 中即可。

```
- name: Relay
    type: relay
    proxies:
      - 机场的分组 #写机场分组的名字
      - ⾃建的节点
```

（如果是第⼀次接触 Clash ，有同学可能想问：那我怎么样让流量⾛这⾥呢？这是由 rule 控制的，例如在 rule ⾥写 DOMAIN,[google.com](https://google.com/),Relay ，google 就会⾛这个代理分组了。不过这个不是本篇的重点，看⽂档就好！ [https://lancellc.gitbook.io/clash/clash-config-file/rules](https://lancellc.gitbook.io/clash/clash-config-file/rules) ）

当然可能还有同学想问，那我偶尔想直连⾃建节点，或者只⽤机场怎么办？

这个问题有两个解决办法：

1 、Clash 的 group 是可以嵌套的（除了 relay group ⾥⾯不能套 relay group ）。

2 、relay group 中 DIRECT 节点就相当于跳过这个节点。

### [](#c6f6a277ea124642ad79cdb7b9020bd2 "引入更高的稳定性")引入更高的稳定性

Clash 还有三种分组：负载均衡、故障转移、⾃动选择

负载均衡：随机使⽤组内的节点。

故障转移：定时测试节点的连通性，当当前节点不通的时候就会⾃动切到可⽤的节点。

⾃动选择：定时切换到延迟最低的节点。

众所周知，无论是什么样的⾃动切换功能，都会带来⼀个问题：在⽹站看来你的 ip 在不断变化，这回导致你的风控等级不断提高，需要不断重新登陆账户或者可能直接被封号。

但如果使⽤了链式代理，由于落地不变，前置节点可以随意更换，哪怕每次连接都更换节点，你的落地 ip 依然是稳定的：

```
proxy-groups:
  - name: "最低延迟"
    type: url-test
    use:
      - 美国的节点 #我们前⾯过滤出来的，忘记了的话参考第二章
    url: 'https://www.gstatic.com/generate_204'
    interval: 300
    tolerance: 10 #延迟相差 10ms 以内就不⽤切换
  - name: Relay
    type: relay
    proxies:
      - 最低延迟
      - ⾃建的节
```

或者负载均衡：

```
proxy-groups:
  - name: "随机选！"
    type: load-balance
    use:
      - 美国的节点
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
   #lazy: true
   #disable-udp: true
    strategy: round-robin #作为前置节点⽤这个⽐较好
  - name: Relay
    type: relay
    proxies:
      - 随机选！
      - ⾃建的节点
```

还可以买好多个机场来做故障转移 balabala……，唔，再说下去就没意思了，能看到这里的都是很厉害的⼈，肯定能想到的。

### [](#266ed399efe94edf9568909876403605 "结尾")**结尾**

其实已经看到很多人这样用了，但一直没有一个完整的体系化的教程，也没有把负载均衡和故障转移这些分组和 relay 结合起来，所以就写了这个帖子。