> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/5399191484)

背景
--

做海外产品的都知道，Stripe 是支持开通支付宝和微信支付的，但是有诸多限制。最近在解决 [ReadPo](https://link.zhihu.com/?target=https%3A//readpo.com/%3Futm_source%3Dzhihu%26utm_medium%3Dpost2) 国内用户想使用支付宝和微信支付订阅费用的需求时，发现中文互联网上很难找到准确信息，就随手记录下来整个过程。

从申请到开通，总计花费 7 天时间，还是非常高效的，Stripe 官方工作人员也非常耐心，体验很好。

Stripe 中能使用支付宝和微信吗？
-------------------

答案是部分支付方式中可以：

*   单次支付：可以。需要在 Stripe 后台中进行在线申请，一般会通过，支付宝更容易通过，微信更严格一些可能会开通不了，我有一个产品出现了微信不通过的情况。
*   订阅（也叫 Subscriptions、Billing、经常性支付）：默认不可以。但是官方留了一个通道，写的是邀请使用（Invite only）。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-339e87957e2163cf5e51b5d833d9eb68_r.jpg)

参考：

*   [Stripe：支付宝详细说明](https://link.zhihu.com/?target=https%3A//docs.stripe.com/payments/alipay)
*   [Stripe：各种支付方式的支持情况一览](https://link.zhihu.com/?target=https%3A//docs.stripe.com/payments/wallets%23product-support)

如何申请
----

### 1. 在线客服沟通

按照上面文档的提醒，到在线支持页面找客服沟通：

[Stripe: Help & Support](https://link.zhihu.com/?target=https%3A//support.stripe.com/%3Fcontact%3Dtrue)

点击右下角的联系支持：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-7cc72c6c3c850eabd1d0c5acc14f40a3_r.jpg)

选择沟通主题，会发现有三种沟通方式，我选择了在线聊天：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-5960bccdd97695339dc2b414b5de8b57_r.jpg)

不要走开，差不多一分钟以内就会有真人联系你。用英文，将自己的问题发给他，他会简单的问你是不是想开通支付宝和微信支付？确认后会告诉你：

> **支付宝可以开通，微信支付不支持连续订阅的支付方式。**

问你是否申请，当然要申请。

此时他会告诉你，**你需要填写一个电子表格**。确认后就可以结束会话了。接着在自己的邮箱里找到一封邮件，里面包含表格、和提交地址。

### 2. 填表并提交申请

表格长这样子：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-29ca85a61821213df5275da489543dba_r.jpg)

总共有 4 类问题：

*   基本信息
*   订阅服务的情况
*   客户支持
*   以及安全防护类的措施等

从问题上可以看到，这是支付宝对海外支付提出的审核要求，应该是比较担心中国用户在海外遇到欺诈问题。认真填写好每一项以后，签名（电子版也可以），将电子版按照要求上传，并回复邮件。

官方承诺会在一星期左右回复你结果（这是我问客服要的一个时间，也有其他网友反馈用了 1 个月）。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### 3. 开通

我在发送邮件大约 7 天后（包含非工作日），收到了他们邮件通知，大约长这样子：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-126ff284a4be8245db3805cd63b07c77_r.jpg)

查看 Stripe 的订阅付款界面，已经可以看到支付宝付款方式在中文地区会显示出来。如果没有显示，可能需要到 Stripe 后台中去开通一下 Alipay，并检查是否有特殊的设置，比如在自己的项目中指定了付款方式。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/v2-673ac8baea463253d8dba4fed27ad2af_r.jpg)

以上。