> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [help.fanruan.com](https://help.fanruan.com/finereport/doc-view-3539.html)

> 1. 概述用户如需生成 iOS 版 App ，则必须 & nbsp; 上传 iOS 证书。

1. 概述
-----

用户如需生成 iOS 版 App ，则必须 [上传 iOS 证书](https://help.fanruan.com/finereport/doc-view-3471.html)。

用户如需 [获取 iOS 证书](https://help.fanruan.com/finereport/doc-view-3471.html)，则必须拥有企业开发者账号。

2. 账号类型
-------

苹果公司提供三种开发者账户：个人、公司、企业。

帆软 App 打包仅支持「企业开发者账号」下获取的 iOS 证书。

他们的区别和适用场景如下：

<table><tbody><tr><th>账户类型<br></th><th><p>是否支持上架</p><p>&nbsp;App Store</p></th><th><p>是否需要注册</p><p>邓白氏编码</p></th><th><p>最大 UUID</p><p>支持数</p></th><th><p>是否支持</p><p>多人协作</p></th><th>适用场景</th></tr><tr><td>个人</td><td>是</td><td>否</td><td>100</td><td>否</td><td>单人独立制作 App，可上架分享</td></tr><tr><td>公司</td><td>是</td><td>是</td><td>100</td><td>是</td><td>多人协作制作 App，可上架分享，UUID 支持有限制</td></tr><tr><td>企业</td><td>否</td><td>是</td><td>不限</td><td>是</td><td><p>帆软支持，推荐理由：</p><p>不能上线应用到 App Store，适用于不希望公开发布应用且需要内部人员大量安装使用的企业</p><p>注：账号一旦到期，手机上已经安装的 APP 会无法启动，因此账号的按期续费非常重要</p></td></tr></tbody></table>

3. 材料准备
-------

申请企业开发者账号和邓白氏编码，需要提前准备以下材料：

1）公司信息整理：公司中英文名、地址中英文名、邮编、营业执照扫描件及正本照片、公章实体照片

2）申请人信息整理：姓名、电话、办公邮件

3）Visa 或信用卡：用于支付开发者年费

注 1：公章实体照片，是指公章刻字面的照片，而不是盖章的纸的照片。如下图所示：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593840084872899.png)

注 2：卡面同时标注「银联」和「Visa」的信用卡可以保证支付成功，其他 Visa 卡不能保证成功。

4. 申请邓白氏编码
----------

申请企业开发者账号，需要提前至少 7 天申请邓白氏编码。

#### 4.1 注册邓白氏编码  

1）打开 [邓白氏编码申请网站](https://developer.apple.com/enroll/duns-lookup/)，输入 Apple ID，进入邓白氏编码申请界面，使用英文填写相关信息，如下图所示：

注 1：申请邓白氏编码的 Apple ID ，尽量使用专门的公司邮箱全新注册，以防人员变动造成失效。

注 2：填写的信息记录请及时备份，法人实体名称（Legal Entity Name）在开发者账号申请时的填写必须与此一致，如无正式的英文名则按字面翻译，英文名后缀以 Co., Ltd./Inc. 结尾。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593838902312264.png)

### 4.2 审核开始通知

提交后，将会收到一封反馈邮件，无需回复，代表邓白氏公司已收到您的申请并受理。

注 1：邮件中的 request id 可视作申请受理编号，并非邓白氏码。

注 2：若超过 3 天未收到受理邮件，请及时致电 Apple Developer 询问。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593839133385912.png)

### 4.3 回复确认邮件

邓白氏码申请初审通过后，会收到一封来自邓白氏公司的「确认邮件」，请及时填写信息并回信。邮件内容如下图所示：

注：「确认邮件」需要提交公司的详细信息，并且留给用户回复的时间非常短。因此请提前准备好需要填写的信息和材料，收到确认邮件后务必第一时间回复。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593839551605194.jpg)

### 4.4 获取邓白氏编码

回复「确认邮件」后，耐心等待一段时间后将收到「申请成功邮件」，邮件中将附上您的邓白氏编码。如下图所示：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593840452314477.png)

5. 申请企业开发者账号
------------

注：邓白氏编码注册成功后，需要 7-14 天才能同步到 Apple。请至少等待 7 个工作日后再进行尝试注册开发者账号。

### 5.1 填写申请

使用 Mac 设备，打开 [苹果企业开发者中心](https://developer.apple.com/cn/programs/enterprise/)，下滑页面，选择「仅在我的组织内部使用的专属 App」，点击「开始填写申请表格」。如下图所示：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593844126309012.png)

2）输入 Apple ID，进入企业开发者账号申请界面，填写企业信息和邓白氏编码，提交开通开发者账号的申请。如下图所示：

注：法人实体名称（Legal Entity Name）必须与申请邓白氏编码时一致。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593841382909819.png)

### 5.2 支付年费

提交申请后，苹果公司会致电确认企业信息，确认通过后，您将收到 Apple Developer 的邮件。

点击邮件中的网址，进入付费页面，使用一张 VISA/Master + 银联标识信用卡付费即可。

支付开发者费用，苹果公司审核通过后，再次登录 [苹果开发者中心](https://developer.apple.com/)，就可以申请 iOS 企业证书了。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1593841868990528.png)