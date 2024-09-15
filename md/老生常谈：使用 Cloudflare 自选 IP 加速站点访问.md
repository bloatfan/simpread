> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [i.hsfzxjy.site](https://i.hsfzxjy.site/cloudflare-blog-acceleration/)

> 本站此前是依托 Github Pages 搭建的，即静态文件托管于 Github 上，再将自定义域名 i.hsfzxjy.site CNAME 到 hsfzxjy.github.io。

本站此前是依托 Github Pages 搭建的，即静态文件托管于 Github 上，再将自定义域名 i.hsfzxjy.site CNAME 到 hsfzxjy.github.io。这种方案免费是免费，但国内的访问速度不甚良好，有几个省甚至出现了超时的现象。为了优化访问，本站现将静态文件迁移至 Cloudflare Pages 上，并使用 Cloudflare 的自选 IP 加速访问。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/image.png) ![](https://raw.githubusercontent.com/bloatfan/PicGo/master/image-1.png)

[图 1] 优化前的效果；[图 2] 优化后的效果。

Github 没有对国内访问作网络优化，因此从国内出发的所有请求，无论使用 Pages 域名或是自定义域名，都会经由较差的线路到达境外的 Github 服务器。以下是网络拓扑的示意图。

国内 IP 境外 Github 服务器 i.hsfzxjy.sitehsfzxjy.github.io

**Cloudflare 与 Github Pages 相比，拥有众多的边缘节点。** 这些边缘节点相当于 Cloudflare 的大门，只要请求到达这里，余下的流程便被 Cloudflare 接管，无论是去 Cloudflare 内部服务或是到其他的境外服务器都是很快的。而选择正确的边缘节点，可以加速特定区域到达 Cloudflare 大门的速度，进而优化整个访问的耗时。

国内 IPCloudflare 其他境外服务器边缘节点 CF Worker / Pagesi.hsfzxjy.site（快！）i.hsfzxjy.siteCloudflarehsfzxjy.github.io104.19.22.121 边缘节点 SSL/TLS 自定义主机名 gh.monad.run 回退源 CNAMEA 记录

以下从右到左解释整个流程：

1.  首先，我们需要有一个辅助域名在 Cloudflare 上解析。本例中的辅助域名是 monad.run。我们新增一条 CNAME 记录，指向 hsfzxjy.github.io。如此一来，便可用 gh.monad.run 访问 Github Pages 上的内容。
2.  其次，也是最关键的一步，我们需要让 i.hsfzxjy.site 主域名收到的请求都转发至 gh.monad.run。为完成这一步，需要在 Cloudflare 的 “SSL/TLS -> 自定义主机名” 设置中，配置回退源为 gh.monad.run 及主机名为 i.hsfzxjy.site。
3.  最后，我们添加一条从主域名 i.hsfzxjy.site 到边缘节点 104.19.22.121 的 A 记录，从而将主域名的请求转发至 Cloudflare，再由 Cloudflare 转发至后续的处理服务。此处的 104.19.22.121 即所谓的 “Cloudflare 优选 IP”，经由其可优化从国内的访问速度。此类 IP 的列表随时可能变化，最新的一批可在 [ip.164746.xyz](https://ip.164746.xyz/) 中查询。注意这一步不要求主域名 i.hsfzxjy.site 在 Cloudflare 上解析，只需要在原有的 DNS 服务商（如 DNSPod）上操作即可。

步骤 1 和 步骤 3 都是标准的 DNS 操作，不再赘述。以下重点说说步骤 2 的操作。

在开始前，我们需要再明确一下此步中涉及的两个域名：主域名 i.hsfzxjy.site 和辅助域名 gh.monad.run。前者面向客户端，后者连接我们真正的服务器。读者需清楚了解这两个域名的关系，并替换为自己的情形，以免操作失误。

首先，我们找到回退源的设置。该设置位于 “SSL/TLS -> Custom HostNames” 下。如果是首次使用，Cloudflare 会要求你开通 Cloudflare for SaaS 服务，绑定信用卡开通即可，这个服务的免费额度足够我们使用。在右边的 “Fallback Origin” 中输入我们的**辅助域名** gh.monad.run，然后点击 “Add Fallback Origin”。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step1.png)

完成后点击 “Add Custom Hostname”，跳转至另一个界面添加**主域名** i.hsfzxjy.site。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step3.png)

随后回到上一步。此时我们的设置已经生效，但主域名的状态为 Pending。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step4.png)

Pending 状态意味着 Cloudflare 需要我们进一步验证方能确认 i.hsfzxjy.site 确实为我们所持有。验证的具体步骤需要我们在主域名的服务商处添加两条 TXT 记录，内容为上图所示。以下是在 DNSPod 中的添加截图。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step5.png)

添加完成后回到 Cloudflare 界面，点击 “Refresh” 按钮。稍等片刻，状态提示变成 “The hostname is using Cloudflare and cannot be activated with an TXT or HTTP validation token. To activate the custom hostname, the DNS target needs to point to the SaaS zone.” 这说明验证已生效，但还需一步将主域名的 DNS 解析指向辅助域名，即添加一条 CNAME 记录，将 i.hsfzxjy.site 指向 gh.monad.run。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step6.png)

再稍等片刻，状态提示变成 “Active” 即表示设置成功。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/step7.png)

至此，正常的回退源配置已经完成，我们可以删除之前的两条 TXT 记录，因为它们只用于临时验证。随后我们**修改 i.hsfzxjy.site 到 gh.monad.run 的 CNAME 记录为 到优选 IP 104.19.22.121 的 A 记录**，从而让主域名的请求经由 Cloudflare 的优选 IP 加速访问，即可完成整个流程。

> 作者：hsfzxjy  
> 链接：[https://i.hsfzxjy.site/cloudflare-blog-acceleration/](https://i.hsfzxjy.site/cloudflare-blog-acceleration/)  
> 许可：[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).  
> 著作权归作者所有。本文**不允许**被用作商业用途，非商业转载请注明出处。