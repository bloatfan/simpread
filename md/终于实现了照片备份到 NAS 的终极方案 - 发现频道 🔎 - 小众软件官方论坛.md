> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [meta.appinn.net](https://meta.appinn.net/t/topic/47549/3) ![](https://raw.githubusercontent.com/lslz627/PicGo/master/44931_2.png)

为什么要存到 NAS
----------

一直不愿意用一些云相册或者网盘之类的服务来备份和保存自己的照片, 主要是两个原因:

1.  不想自己的隐私被这些服务方一直视奸, 你传上去的照片视频肯定会被这些服务方扫描一遍的, 就算没有私密的照片, 我也接受不了隐私被这样侵犯
    
2.  不想自己的数据被绑架, 毕竟数据是别人手上, 哪天别人要跑路或者涨价, 你也没有任何办法
    

过去尝试的方案
-------

### 群晖 moments

最开始 NAS 装了群晖, 于是就用了群晖自带的 moments 来同步照片. 用了一段时间后出现了一些问题:

1.  moments app 几乎不再更新, 体验不算差, 但绝对不好, 老婆总是抱怨 ios 上这不好用那不好用
    
2.  与群晖绑定, 因为必须搭配 moments 服务端一起使用, 所以你没有任何别的选择, 这让我感觉很被动
    

现在似乎群晖已经淘汰 moments 了, 出了新的群晖 photos, 这个我没有试用过, 因为群晖硬件还是太贵了, 现在已经改用 Unraid 了

### PhotoPrism - 超好用

不得不说 PhotoPrism 确实太好用了, 我最喜欢它的一点是他的兼容性很强, 你只要丢给它一个目录, 他就能处理里面的所有照片, 并且可以在各个维度进行检索, 对于超大量的照片来说真的很好用.

但问题就在于 PhotoPrism 只有服务端, 把照片同步到 NAS 这个动作还要我自己想办法来完成

### PhotoSync - 不值得这个价格

这个 app 的功能还是可以的, 但是它的 UI 和交互是在是有点古老, 我还需要专门用一个 app 来进行同步这个事情, 最重要的是要付费才能用, 我觉得不太值得, 放弃.

### Nextcloud - 移动端 app 太差

Nextcloud 作为网盘来说挺好用的, 我尝试使用 Nextcloud 的移动端来同步相册照片. 但我真的安装了安卓端 app 后, 连接了我 https 反代后的 URL 居然直接崩溃了, 完全没法用, 放弃.

Alist + Pho + Rclone + PhotoPrism - 终极方案
----------------------------------------

这个方案最让我喜欢的一点是各个环节都不是耦合的, 去掉其中任何一个环节都不会影响到其他环节, 每个人完全可以根据自己的喜好来替换其中的某个部分.

### Alist

[官网 8](https://alist.nn.ci/zh/): [Home | AList 文档 8](https://alist.nn.ci/zh/)

负责把各种可用的储存映射成`webdav`, 支持各种云盘网盘以及本地储存

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### Pho

[官网 31](https://pho.tools/): [https://pho.tools/ 31](https://pho.tools/)

负责通过`webdav`上传照片到`Alist`映射的储存

它很好的一点是支持加密后上传, 这样就可以在网盘上做二次备份, 在能避免隐私泄露的前提下多一层数据保险

而且我可以在手机上直接用这个 app 来浏览我本地和已经上传的照片, 这个 app 的 UI 和交互都很好, 可以直接用它来代替系统自带的相册

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### Rclone

[官网 6](https://rclone.org/): [https://rclone.org/ 6](https://rclone.org/)

负责把`Alist`的`webdav`映射到 NAS 的文件系统内, 来把照片喂给 PhotoPrism

### PhotoPrism

[官网 26](https://www.photoprism.app/): [https://www.photoprism.app/ 26](https://www.photoprism.app/)

最终的 "集大成者", 全家所有设备备份的照片最终全部喂到这里, 即使是海量照片也能根据它快速索引到自己想找的照片

![](https://raw.githubusercontent.com/lslz627/PicGo/master/photoprism.jpg)

### 最终效果

#### 平时手机上浏览和上传

使用`Pho`浏览本地和最近上传的照片

#### 查找和浏览过去某个时候的照片

根据信息用`PhotoPrism`检索即可

> 注: 以上分享的主要目的在于推广我的个人项目 Pho, 本文的作者即 Pho 的个人开发者.  
> 如果文章对大家有所帮助, 我会继续更新一些此类主题的实战教程.

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1592_2.png)

那个老板，下次推荐的时候… 不要间隔这么近的几天就多次发布，另外也最好读一下规则：

![](https://raw.githubusercontent.com/lslz627/PicGo/master/d0bc44d1e17e5e46a6faadd83d99f73c7047d7c3.png) [内容提交规则（首先阅读）](https://meta.appinn.net/t/topic/43728) [发现频道 🔎](/c/faxian/10)

> 不遵循规则的内容将会被拒绝 1. 人工审核 所有内容均由人工审核，请耐心等待。新用户在审核通过前会被系统临时禁用账号，审核通过后自动解禁。 2. 这里是小众软件网站内容的主要来源 如果你觉得提交到这里的内容被忽略了（很可能），请以合理的方式提醒我们，或直接联系我们。发现频道的内容会以邮件的形式直达编辑邮箱。 但注意，发现频道的内容并不会全部收录。 3. 不要复制 请不要把产品介绍从应用商店或者其他网站中原样拷贝过来。 4. 提供链接 请提供官网链接、应用商店链接，除小程序以外，不要仅提供二维码。手机应用必须提供应用商店地址（优先 App Store 与 Google Play），网盘不被接受（开源且未上架市场的应用除外，但需提供源码地址）。 需要关注并回复可见的链接也不被接受，我们不会去测试这样的软件。 5. 兑换码 付费产品请私信管理员兑换码，降低评测成本。 注意，由于发现频道已经被很多站点采集，请务必不要公开兑换码，这将导致兑换码外流。 6. 不要标题党 标题内请不要出现 【推荐】 【必读】 之类的词语，【】内可出现分类，比如【拍照应用】、【效率工具】。 再…

 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==) [](/t/topic/47549/1 "go to the quoted post")![](https://raw.githubusercontent.com/lslz627/PicGo/master/44931_2.png) fregie:

> 注: 以上分享的主要目的在于推广我的个人项目 Pho, 本文的作者即 Pho 的个人开发者.

推广就大方地说，不把推广内容伪装成普通文章。软广让人讨厌，多次推广更是。如果你不想走滥发广告（spam）的路子，就克制点给大家留个好印象。