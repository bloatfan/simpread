> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [7-shu.com](https://7-shu.com/%E7%94%A8%E7%8B%AC%E8%A7%92%E6%95%B0%E5%8D%A1%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%8F%91%E5%8D%A1%E7%BD%91%E7%AB%99%EF%BC%8C%E5%AF%B9%E6%8E%A5v%E5%85%8D%E7%AD%BE%E3%80%81%E6%98%93%E6%94%AF%E4%BB%98/)

> https://7-shu.com/2023%e5%a6%82%e4%bd%95%e4%bd%bf%e7%94%a8v%e5%85%8d%e7%ad%be%e5%bd%a9%e8%99%b9%e6%98......

[![](https://7-shu.com/wp-content/uploads/2023/07/950x185_banner.webp)](https://faka8.ink/)

来给这两天折腾的个人支付结个尾，算是实际应用测试吧！前面已经教了大家如何搭建易支付啊 v 免签这些东西，今天就来实际运用一下，看下如何对接系统并实际运用！顺道介绍下 USDT 支付就算结束啦！

那废话不多说，我们正式开始！这里我们选用了 Github 上开源发卡网站程序 **[独角](https://github.com/assimon/dujiaoka)[数](https://github.com/assimon/dujiaoka)[卡](https://github.com/assimon/dujiaoka)** 进行测试，对接前面折腾的 V 免签进行一次实地测试。并且介绍下另一个开源项目 **[epusdt](https://github.com/BlueSkyXN/PHPAPI-for-epusdt)**，进行 USDT 数字货币支付。

首先我们安装 **[独角](https://github.com/assimon/dujiaoka)[数](https://github.com/assimon/dujiaoka)[卡](https://github.com/assimon/dujiaoka)**，v 免签的安装流程前面已经有介绍，这里就不再重复了，有需要的童鞋可以去看上面的文章！本次介绍 **[独角](https://github.com/assimon/dujiaoka)[数](https://github.com/assimon/dujiaoka)[卡](https://github.com/assimon/dujiaoka)** 安装流程以及设置。

安装独角数卡
------

按照作者的描述主机不支持虚拟主机，大概率也不支持 windows 服务器，所以请自备 Linux 系统主机。首先还是将准备好的域名（本次演示使用域名：sk.7-shu.com）解析到主机。主机还是使用宝塔搭建 LNMP 环境，本次演示所使用的环境：php 7.4、mysql 5.7、 nginx 1.21、phpmyadmin 4.9、在此基础之上还需安装 Redis 7.0 和 Supervisor 2.2 其中 Supervisor 可以是宝塔应用管理器，二选一。一切准备就绪开始新建站点，并开启 SSL。

站点新建好以后，对 PHP 进行设置，禁用删除以下函数。

```
putenv，proc_open，pcntl_signal，pcntl_alarm

```

[![](https://7-shu.com/wp-content/uploads/2023/02/1-1.webp)](https://7-shu.com/wp-content/uploads/2023/02/1-1.webp)

结下来给 PHP 安装扩展，还是在 PHP 管理，找到**安装扩展**添加`fileinfo`、`redis`、`opcache(可选安装，性能加强)`三个扩展程序并安装。

[![](https://7-shu.com/wp-content/uploads/2023/02/2-1.webp)](https://7-shu.com/wp-content/uploads/2023/02/2-1.webp)以上扩展安装完以后最好重启、重载一次 PHP。

接下来回到站点根目录，上传或远程下载[**独角数卡最新源码**](https://github.com/assimon/dujiaoka/releases)！

[![](https://7-shu.com/wp-content/uploads/2023/02/3-1.webp)](https://7-shu.com/wp-content/uploads/2023/02/3-1.webp)如果是远程下载就在这个链接上点右键复制并在宝塔粘贴。

对网站运行目录及伪静态进行配置。  
在**网站设置** ➜ **站点目录** ➜ **运行目录**选择 / public 并保存（如下图所示）。

[![](https://7-shu.com/wp-content/uploads/2023/02/4-3.webp)](https://7-shu.com/wp-content/uploads/2023/02/4-3.webp)

在伪静态下拉菜单中选择: laravel5 并保存。

[![](https://raw.githubusercontent.com/bloatfan/PicGo/master/5.webp)](https://7-shu.com/wp-content/uploads/2023/02/5.webp)

配置完成以后，直接访问域名按照提示填写配置安装即可！

[![](https://7-shu.com/wp-content/uploads/2023/02/6.webp)](https://7-shu.com/wp-content/uploads/2023/02/6.webp)

安装完成以后我们需要对其添加守护进程，回到宝塔面板后台，在应用商店找到 **Supervisor** 并设置，然后新建添加一个守护进程（配置如下图）。

[![](https://7-shu.com/wp-content/uploads/2023/02/7-4.webp)](https://7-shu.com/wp-content/uploads/2023/02/7-4.webp)

名称：随意填写  
启动用户：选择 www  
运行目录：选择程序根目录  
启动命令：/www/server/php / 你的 php 版本 / bin/php /www/wwwroot / 你的网站根目录 / artisan queue:work

注意：php 版本和网站根目录，此处根据你的实际情况填写！  
参考格式：

```
/www/server/php/74/bin/php /www/wwwroot/sk.7-shu.com/artisan queue:work

```

添加以后进程显示运行，至此安装完成！如今后数卡后台进行设置以后提示重启，请在这里重启守护进程，否则容易设置不生效！  
独角数卡使用常见问题可以参考：https://github.com/assimon/dujiaoka/wiki/problems

安装 Epusdt 教程
------------

宝塔安装教程：https://github.com/assimon/epusdt/blob/master/wiki/BT_RUN.md  
独角数卡后台设置：https://github.com/assimon/epusdt/tree/master/plugins/dujiaoka

独角数卡 v 免签支付设置
-------------

登录后台以后在左侧找到**配置** ➜ **支付配置** ，默认后台除了数字货币其它支付接口都是开启状态。作者给出了各支付接口对应配置

<table><thead><tr><th>支付选项</th><th>商户 id</th><th>商户 key</th><th>商户密钥</th><th>备注</th></tr></thead><tbody><tr></tr><tr><td>Epusdt</td><td>api 接口认证 token</td><td>空</td><td>epusdt 收银台地址 +/api/v1/order/create-transaction</td><td>如果独角数卡和 epusdt 在同一服务器则填写<code>127.0.0.1</code>不要填域名，例如<code>http://127.0.0.1:8000/api/v1/order/create-transaction</code></td></tr><tr><td>支付宝官方 (当面付、PC、wap)</td><td>支付宝开放平台应用 appid</td><td>支付宝公钥</td><td>商户私钥</td><td></td></tr><tr><td>payjs</td><td>payjs 商户号 (mchid)</td><td>空</td><td>payjs 密钥</td><td></td></tr><tr><td>码支付</td><td>平台商户号</td><td>码支付请求网址</td><td>密钥</td><td>市面上太多码支付了，直接将支付接口网址填入商户 key 就行。只要加密方式一样的就能发起支付，不行就不行了。懒得一家一家对接了</td></tr><tr><td>微信官方</td><td>公众号或小程序 appid</td><td>商户号</td><td>商户 api 密钥</td><td></td></tr><tr><td>麻瓜宝</td><td>商户密钥</td><td>空</td><td>任意字符串</td><td></td></tr><tr><td>paysapi</td><td>商户号</td><td>空</td><td>密钥</td><td></td></tr><tr><td>易支付</td><td>易支付</td><td>易支付请求网址</td><td>密钥</td><td>记得网址后面加 / submit.php，不然请求没有作用！例如 http://xxx.com/submit.php</td></tr><tr><td>V 免签</td><td>V 免签通讯密钥</td><td>空</td><td>V 免签地址</td><td></td></tr><tr><td>Paypal</td><td>商家账号，一般是邮箱</td><td>应用 Client ID</td><td>Secret</td></tr></tbody></table>

这里我们只对 v 免签进行讲解，其它接口请自行查阅。其实不论你是安装 v 免签原版还是二开版本都只需要设置易支付接口。

**v 免签原版设置：**登录你易支付后台，找到**商户管理**点击登录。进入商户页面，在**个人资料**中查看 API 信息并记录。

[![](https://7-shu.com/wp-content/uploads/2023/02/8-2.webp)](https://7-shu.com/wp-content/uploads/2023/02/8-2.webp)记录红框部分的值

回到独角数卡后台**配置** ➜ **支付配置** ➜ **易支付 - 支付宝** 进行配置，配置填写参考下图，这里支付宝和微信配置都一致，点击启用并保存。

[![](https://7-shu.com/wp-content/uploads/2023/02/9-1.webp)](https://7-shu.com/wp-content/uploads/2023/02/9-1.webp)

**v 免签二开设置：**登录 v 免签后台，在系统设置拷贝商户 ID、通讯密钥。回到独角数卡后台，依旧是**配置** ➜ **支付配置** ➜ **易支付 - 支付宝** 参照上图更换相应网址、商户 ID、粘贴 v 免签的通讯密钥，设置完成。微信只需更换名称，其它参考支付宝配置对应填写保持一致即可！

建议上线使用二开，在原版的基础上二开更完善、更简单！如实在看不懂可以点击下方按钮！

**如果觉得文章对你有帮助，欢迎点赞、留言、打赏请我喝咖啡！**

[![](https://7-shu.com/wp-content/uploads/2023/07/950x185_banner.webp)](https://faka8.ink/)