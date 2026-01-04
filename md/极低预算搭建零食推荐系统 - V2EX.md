> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.v2ex.com](https://www.v2ex.com/t/1183060)

> 分享创造 - @zgldh - # 起因我在一家软件公司工作，每个月公司会补充一些零食供大家填牙缝。

**V2EX = way to explore** V2EX 是一个关于分享和探索的地方 [现在注册](/signup) 已注册用户请  [登录](/signin) 爱意满满的作品展示区。 #Wrapper {background-image: url("/static/img/shadow_light.png"), url("//static.v2ex.com/grids/air.png"); background-repeat: repeat-x, repeat; } .box { border-bottom: 1px solid #cdd8ff; } [广告](/advertise) ⚡ Sora 去水印，仅需 5s[![](https://cid.v2ex.pro/ipfs/QmRGKYQ7RurHVShd99w2CvULwPTzakdttPiJNXoqaE3ZVx)](https://nanophoto.ai/remove-watermark-sora-2) ⚡Sora 高清去水印，效果老好了，不信你试试 [去水印，需要积分请留言🎁](https://nanophoto.ai/remove-watermark-sora-2) Promoted by [Sparking](/member/Sparking) [ PRO](/pro/about)

[![](https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=73&d=retro)](/member/zgldh) [V2EX](/)  ›  [分享创造](/go/create)

极低预算搭建零食推荐系统
============

[

](javascript:) [

](javascript:)  [zgldh](/member/zgldh) · 3 小时 17 分钟前 · 385 次点击

起因
==

我在一家软件公司工作，每个月公司会补充一些零食供大家填牙缝。每当采购的时候，我都曾看到管后勤的同事为选品花费好久时间。后面甚至放弃思考，完全重复上个月的订单。

能否创建一个系统，给定预算和需求，即能自动的帮我搭配零食？

尝试
==

第一次尝试做此类系统已经是多年之前了，当时我只会增删改查，向量数据库、推荐系统、人工智能更是仅仅听说过名字。我把从电商页面抓取来的商品，一个个的塞入关系数据表，幻想着用一套复杂的 if/else 来解决这个问题。

很快我发现这个系统的难度超过了我当时的能力水平，遂作罢。但依然每年都会重新思考该系统的可行性。

转机
==

这几年 AI 应用越来越火，我也参与了一些 AI 集成相关的项目。渐渐在运用中解了相关的知识。我突然意识到用向量数据库和大语言模型可以实现这个零食推荐系统了。

入库端：

*   LLM 对商品特征进行提取，生成描述字符串。
*   Embedding 将商品描述字符串转换为向量。
*   向量数据库存储向量和相关元数据。

查询端：

*   LLM 对用户需求进行拆解，生成描述字符串。
*   Embedding 转换为查询向量
*   从向量数据库搜索该查询向量，得到潜在匹配的商品。
*   对匹配到的商品进行组合。

制作
==

由于我是个穷 B ，能有免费的就不想掏一分钱。 于是我盯上了大善人 Cloudflare：

*   Pages 服务，用来展现前端页面、响应查询 API 。
*   Vectorize ，向量数据库。我选用的是 1024 维度余弦库。
*   D1 ，关系数据库，用来存储查询记录。
*   Workers AI ，提供在线文本嵌入和文本生成服务。我选用的嵌入模型是 `@cf/baai/bge-m3`，生成模型是 `@cf/openai/gpt-oss-20b`

虽然上面的服务都有一定免费额度，但我希望把这有限的额度都投入到查询端（客户端）上。入库时的 AI 算力怎么办呢？

于是我买了台搭载 AMD AI 395 处理器，128G 内存的迷你主机来解决入库 AI 算力问题。（ PS：AMD 生态还是不太好。其实用自己的老机器也能跑下面的本地模型。）

每天定时从京东联盟和淘宝联盟抓取热门商品列表，调用本地模型，最后发布到 Cloudflare 向量数据库里。 本地 AI 模型包括：

*   mineru-vllm 做图文识别
*   qwen/qwen3-next-80b 做描述字符串提取

（为了确保文本嵌入一致，入库流程仍调用 Cloudflare 的 `@cf/baai/bge-m3` 模型）

整个制作过程充分依靠了 AI 开发：

*   [Shotgun code](https://github.com/glebkudr/shotgun_code) 能将整个代码库打包成一个提示词，再利用 Gemini 的超长上下文来一口气输出 diff 指令。再回到任意 AI IDE 或插件（ Cline 之类）让他们帮忙把 diff 指令应用到代码上即可。
*   [AI Studio](https://aistudio.google.com/) 免费试用 Gemini 3 Pro ，虽然数据可能会被拿去训练。
*   VS Code 的 Cline 插件，搭配 DeepSeek 或 Gemini 。
*   VS Code 的通义灵码插件。简单修改还挺方便的。

成果
==

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

断断续续制作了两三个月，加上了导出 CVS 列表、换一个商品、限定单价、限定平台、历史记录等功能。现在终于可以见人了。元旦节宅家的零食就是用它帮忙选的。

起了个诨名 零食搭子 [https://snackbuddy.idagou.com/](https://snackbuddy.idagou.com/)

欢迎各路好汉光临！

第 1 条附言  ·  2 小时 37 分钟前

原文截图烂了。重新发个

![](https://imgur.com/gE1W76t.jpg)

[

零食推荐系统](/tag/零食推荐系统)[

AI](/tag/AI)[

向量数据库](/tag/向量数据库) 11 条回复  **•**  2026-01-04 21:43:30 +08:00<table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/941ccc9d892801166897290d388e8133?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 1 <strong><a href="/member/guiys">guiys</a></strong> &nbsp; &nbsp; &nbsp;3 小时 10 分钟前 via Android &nbsp; <img class="" src="https://www.v2ex.com/static/img/heart_20250818.png?v=c3415183a0b3e9ab1576251be69d7d6d"> 1 我觉着，大家的需求是好吃的零食，不是如何搭配零食。<br>搭配出来的零食大家还是只会挑爱吃的好吃的。</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 2 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;3 小时 6 分钟前 @<a href="/member/guiys">guiys</a> 是啊，看到不喜欢的我都会换掉。</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/avatar/d2e2/2de3/20464_normal.png?m=1344483474"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 3 <strong><a href="/member/regent">regent</a></strong> &nbsp; &nbsp; &nbsp;3 小时 4 分钟前 楼主动手能力可以呀，还请问下 AI MAX 395 使用体验如何？您这个需求会占用多少资源？</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 4 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;2 小时 57 分钟前 原文截图怎么烂了。。 补一个<br>![截图]( <a target="_blank" href="https://snackbuddy.idagou.com/screenshot.png" rel="nofollow noopener">https://snackbuddy.idagou.com/screenshot.png</a>)</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/a101f4a233d8c1a4c9d37f8b5aafb595?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 5 <strong><a href="/member/chihiro2014">chihiro2014</a></strong> &nbsp; &nbsp; &nbsp;2 小时 54 分钟前 要是买的多，甚至你可以考虑上淘宝联盟或者京粉，这样还能创造额外收入</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 6 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;2 小时 53 分钟前 @<a href="/member/regent">regent</a> 小主机速度很快，内存带宽较高 256GB/s （还是不如正规显卡）。系统入库时资源占用没量化计算过，反正能边跑 LLM 边玩儿群星。</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 7 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;2 小时 52 分钟前 @<a href="/member/chihiro2014">chihiro2014</a> 你太懂了哈哈。。已经连接上了。其实我首先连接的是拼多多，但是他把我号给封了，元旦后刚解封。。</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/avatar/cbe5/78ea/359387_normal.png?m=1653648081"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 8 <strong><a href="/member/zcf0508">zcf0508</a></strong> &nbsp; &nbsp; &nbsp;2 小时 46 分钟前 有点意思！</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 9 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;2 小时 44 分钟前 @<a href="/member/zcf0508">zcf0508</a> 年货就用这个办了</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/3afd0277cf0413218f8d8880768cd695?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 10 <strong><a href="/member/Aprdec">Aprdec</a></strong> &nbsp; &nbsp; &nbsp;1 小时 10 分钟前 不错, 保持住!</td></tr></tbody></table><table cellpadding="0" cellspacing="0" border="0" width="100%"><tbody><tr><td width="48" valign="top" align="center"><img class="" src="https://cdn.v2ex.com/gravatar/30fce28a60c11ce4e80876628d0a558c?s=48&amp;d=retro"></td><td width="10" valign="top"></td><td width="auto" valign="top" align="left">&nbsp; &nbsp; 11 <strong><a href="/member/zgldh">zgldh</a></strong> &nbsp; OP&nbsp; &nbsp;56 分钟前 @<a href="/member/Aprdec">Aprdec</a> 感谢鼓励</td></tr></tbody></table>[](https://wwads.cn/click/bait)[![](https://cdn.wwads.cn/creatives/PrKzRdnwFgPUKE2tVSqvTb07vqWRcCns1oZNlKHV.jpg)](https://wwads.cn/click/bundle?code=5jyLHiVQP2uvOfXC3X6x9CXhfvQouj)[**NIUSHOP 开源商城 V6 多商户 B2B2C** 100% 开源 文档完善 二开方便 免费商用 欢迎体验](https://wwads.cn/click/bundle?code=5jyLHiVQP2uvOfXC3X6x9CXhfvQouj)[广告](https://wwads.cn/?utm_source=property-124&utm_medium=footer "点击了解万维广告联盟")