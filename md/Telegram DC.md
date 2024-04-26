> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [lab.best33.com](https://lab.best33.com/telegram-datacenter/)

我的 Telegram 账号在哪个 DC ？
======================

众所周知，Telegram 宣称自己有 5 个数据中心，但事实上只有三个物理地址。

我们通过 Telegram login 获取用户的头像地址，然后从头像重定向的域名来判断用户的数据中心。

由于 DC2/3 是由 DC4/1 提供 web 服务的，根据已有信息，我们无法推测你实际属于其中的哪个 DC。

你说什么头像域名？

比如你访问 [https://t.me/durov](https://t.me/durov)，然后你会发现它头像的 URL 是 https://cdn1.telegram-cdn.org 开头的。  
这说明，这张图片存储在 DC1 的服务器上。  
而头像通常和用户在同一个 DC，所以看头像的域名就可以知道用户所在的 DC 了。  
而 Telegram Login Widget 返回的头像地址其实是 t.me 开头的，这就需要先访问它，得到重定向的地址。  
由于 t.me 没有设置 CORS 策略，这听起来不太可能，不过配合 iframe 和 CSP，利用浏览器的一些侧信道泄漏，我们还是可以做到这一点。

使用该页面你不需要有用户名，但你需要**有头像，并设置为对 Everyone 可见**。

如果想换账号，请到 Telegram 客户端中点击 Terminate session 。

请点击上面的登录按钮以开始。

<table><thead><tr><th>数据中心</th><th>AS 号</th><th>位置</th></tr></thead><tbody><tr data-dc="1"><td>DC1</td><td>AS59930</td><td>迈阿密</td></tr><tr data-dc="2"><td>DC2</td><td>AS62041</td><td>阿姆斯特丹</td></tr><tr data-dc="3"><td>DC3</td><td>AS59930</td><td>迈阿密</td></tr><tr data-dc="4"><td>DC4</td><td>AS62041</td><td>阿姆斯特丹</td></tr><tr data-dc="5"><td>DC5</td><td>AS62014</td><td>新加坡</td></tr></tbody></table>

为了你的隐私安全，本页没有追踪代码，所有测试均在你的浏览器中运行。  
所有信息的使用都遵守 Telegram 提供的 Login Widget 功能限制。

拓展阅读：[Telegram DC 分析报告 | Hertz Blog](https://blog.hertz.zone/2021-11-07_TGDC), [Telegram DC 之都市传说 - Coxxs](https://dev.moe/2564)。

Created by [oott123](https://oott123.com)。