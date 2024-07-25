> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.dejavu.moe](https://blog.dejavu.moe/posts/telegram-upload/#telegram-%E4%B8%8B%E8%BD%BD)

> Telegram 文件批量上传 / 下载工具，单文件最大支持 2GB 大小、支持文件分割

前言 [#](#%e5%89%8d%e8%a8%80)
---------------------------

平时为了满足自己的 “松鼠癖” 在服务器上部署了一些文件下载 / 同步服务如：[Aria2 Pro](https://github.com/P3TERX/Aria2-Pro-Docker)、[qBittorrent Enhanced](https://hub.docker.com/r/superng6/qbittorrentee)、[Resilio Sync](https://hub.docker.com/r/linuxserver/resilio-sync) 等。服务器 / VPS 上远程下载好了，如果想要把文件分享给 Telegram 联系人，又不想不通过服务器直链分享文件，就需要手动下载文件再上传到 Telegram；另一种情况就是 Telegram 上收的一些文件需要上传到 OneDrive 进行归档，这样在平时的使用中就会增加不少时间和流量成本。

不久前我发现一个 Python 写的开源项目 [telegram-upload](https://github.com/Nekmo/telegram-upload) 可以很方便的在服务器 / VPS 和 Telegram 之间进行文件的上传和下载操作（上传 OneDrive 下回再讲）

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/1.webp)个人文件管理方案

特性 [#](#%e7%89%b9%e6%80%a7)
---------------------------

[telegram-upload](https://github.com/Nekmo/telegram-upload) 使用个人 Telegram 账户上传和下载文件，单个文件最大支持 2GB（Telegram Bot 限制 50MB）｜ [文档（英文）](https://docs.nekmo.org/telegram-upload/)

*   上传多个文件
*   下载文件
*   上传视频添加缩略图信息（需要 ffmpeg）
*   上传 / 下载完成后删除本地 / 远程文件

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2.webp) telegram-upload

安装 [#](#%e5%ae%89%e8%a3%85)
---------------------------

### 安装 Python 和 Pip [#](#%e5%ae%89%e8%a3%85-python-%e5%92%8c-pip)

服务器需要 安装 Python，下面我将以 Debian 10 下 Python3 安装 [telegram-upload](https://github.com/Nekmo/telegram-upload) 为例，其他操作系统详见 [Python 安装指南](https://docs.python-guide.org/starting/installation/)

> Tips: Python2 已经过时，建议安装使用 Python3

```
# Debian 刷新软件源
sudo apt-get update
# 安装 Python3
sudo apt-get install python3
# 安装 Pip3
sudo apt-get install python3-pip
# 安装完成后验证一下版本
python3 -V
pip3 -V
# 类似下面输出就代表没问题了
root@debian:~# python3 -V
Python 3.7.3
root@debian:~# pip3 -V
pip 18.1 from /usr/lib/python3/dist-packages/pip (python 3.7)


```

### 安装 ffmpeg（可选） [#](#%e5%ae%89%e8%a3%85-ffmpeg%e5%8f%af%e9%80%89)

如果你需要上传视频文件的话，需要确保安装 ffmpeg，否则会导致上传的视频无法在线播放或者显示缩略图

```
# 对于 Debian/Ubuntu，使用下面命令安装 ffmpeg
sudo apt-get update && sudo apt-get install ffmpeg
# 检查 ffmpeg 安装
ffmpeg -version


```

### 安装 telegram-upload [#](#%e5%ae%89%e8%a3%85-telegram-upload)

**安装稳定版**

稳定版是推荐的版本，默认情况下会默认安装最新的稳定版 [telegram-upload](https://github.com/Nekmo/telegram-upload)

```
sudo pip3 install -U telegram-upload


```

**指定版本号**

使用以下命令从 [Pypi](https://pypi.org/) 安装已发布的其他版本，`<version>` 是具体版本号

```
pip install telegram-upload==<version>


```

**从 GitHub 仓库安装**

已经安装有 Git 的情况安装，`<branch>` 是 GitHub 分支名

```
pip install git+https://github.com/Nekmo/telegram-upload.git@<branch>#egg=telegram_upload


```

系统没有安装 Git 的情况下安装，`<branch>` 是 GitHub 分支名

```
pip install https://github.com/Nekmo/telegram-upload/archive/<branch>.zip


```

配置 [#](#%e9%85%8d%e7%bd%ae)
---------------------------

### 注册 Telegram App [#](#%e6%b3%a8%e5%86%8c-telegram-app)

用 Telegram 注册的手机号登录 [My Telegram](https://my.telegram.org/auth)，登录完成后应该会看到如下几个菜单

**Your Telegram Core**

*   [API development tools](https://my.telegram.org/apps)
*   [Delete account](https://my.telegram.org/delete)
*   [Log out](https://my.telegram.org/auth/logout)

进入 [API development tools](https://my.telegram.org/apps)，可以看到下图的界面

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/3.webp)Telegram App 设置

`App api_id` 和 `App api_hash` 是等下需要的应用 ID 和应用机密，复制保留备用；`App title` 和 `Short name` 可以随便填，拉到页面底部 `Save changes` 保存设置

经过和 [xiaoxinhuozhu](https://github.com/xiaoxinhuozhu) 同学的验证，如果此步骤注册 API 应用过程中出现莫名 Error，很可能是因为国内 +86 手机号被 Telegram 限制，建议使用其他国家的手机号，目前使用 Google Voice 美国 VoIP 手机号没问题

请注意，此页面的应用 ID、机密和密钥等信息非常非常非常重要，千万不要泄露！

### 生成配置文件 [#](#%e7%94%9f%e6%88%90%e9%85%8d%e7%bd%ae%e6%96%87%e4%bb%b6)

随便准备一个小文件，上传到自己 Telegram 的收藏夹：

```
# telegram-upload [文件路径] 
telegram-upload /telegram/[ATFMaker]白修女.zip


```

第一次登录会提示输入 Telegram `手机号` 其他 Telegram 设备收到的 `验证码`

> 注意手机号需要带国际电话区号，美国 / 加拿大是 +1…

```
Please enter your phone: +1**********
Please enter the code you received: *****
Signed in successfully as [Telegram Nick Name]


```

文件会开始上传，我们可以再终端里看到上传进度，上传完成后，你应该能看到文件在你的 Telegram 收藏夹了 🤠

```
xxxxxuser@b1s:~telegram-upload /telegram/[ATFMaker]白修女.zip
Uploading "[ATFMaker]白修女.zip"  [####################################]  100%


```

默认情况下我们生成的配置文件路径为 `/root/.config/telegram-upload.json`，该文件包含你的 Telegram App 登录的机密信息，请妥善保存，之后的登录就不需要在输入机密信息了

使用 [#](#%e4%bd%bf%e7%94%a8)
---------------------------

现在我们的服务器上 [telegram-upload](https://github.com/Nekmo/telegram-upload) 安装和配置都没问题了，下面根据 [官方文档](https://docs.nekmo.org/telegram-upload/usage.html) 介绍一下它的用法，这里主要分为 Telegram 上传和 Telegram 下载两部分

### Telegram 上传 [#](#telegram-%e4%b8%8a%e4%bc%a0)

默认情况下，使用 Telegram 个人账户上传一个或多个文件到你的 `收藏夹`，单个文件最大限制 2GB，用法：

```
telegram-upload [选项] [文件路径]


```

**[选项]**

*   `--to <to>`
    
    `电话号码`、`用户名`、`邀请连结` 或 `收藏夹（保存的消息）`，默认为 `收藏夹`
    
*   `--config <config>`
    
    要使用的 Telegram App 配置文件路径，默认为 `/root/.config/telegram-upload.json`
    
*   `-d`, `--delete-on-success`
    
    上传成功后删除本地文件
    
*   `--print-file-id`
    
    上传完成后打印上传文件的 id
    
*   `--force-file`
    
    强制作为文件发送，文件名将被保留，但预览将不可用
    
*   `-f`, `--forward` `<forward>`
    
    将文件转发给聊天（昵称 / ID）或用户（用户名 / 手机号码 / ID），此选项可以多次使用（发送给多个频道 / 群组 / 联系人）
    
*   `--directories` `<directories>`
    
    定义如何处理目录, 默认情况下不接受目录，并且会出现错误提示（fail），可选递归目录（recursive）
    
    选项：`fail`|`recursive`
    
*   `--large-files` `<large_files>`
    
    定义如何处理 Telegram 不支持的大文件，默认情况下，不接受大文件，并且会出现错误提示（fail），可选大文件切割上传（split）
    
    选项：`fail`|`split`
    
*   `--caption` `<caption>`
    
    更改文件描述。默认为文件名。
    
*   `--no-thumbnail`
    
    禁用生成缩略图，对于一些已知的文件格式（如图片、视频等），Telegram 可能仍会生成缩略图或显示预览
    
    注意：此参数与参数互斥：[thumbnail-file]
    
*   `--thumbnail-file` `<thumbnail_file>`
    
    用于指定上传文件的预览文件（缩略图）的路径
    
    注意：此参数与参数互斥：[no-thumbnail]
    
*   `-p`, `--proxy` `<proxy>`
    
    对于一些无法连接 Telegram（如中国）使用代理，支持 `http`、`sock4`、`sock5`、`mtproxy` 等代理类型，具体使用见后文
    
*   `-a`, `--album`
    
    将视频或照片作为相册发送
    

### Telegram 下载 [#](#telegram-%e4%b8%8b%e8%bd%bd)

下载聊天中的所有最新消息，默认情况下从`收藏夹（保存的消息）` 下载，建议将要下载的文件转发到 `收藏夹（保存的消息）` 并使用参数 `--delete-on-success`，下载完成后转发的消息将自动从聊天中删除，例如下载队列，用法：

*   `-f`, `--from` `<from_>`
    
    手机号码、用户名、频道 / 群组 ID 或者 收藏夹（保存的消息），默认为 `收藏夹`
    
*   `--config` `<config>`
    
    要使用的 Telegram App 配置文件路径，默认为 `/root/.config/telegram-upload.json`
    
*   `-d`, `--delete-on-success`
    
    下载成功后删除电报信息，用于创建下载队列
    
*   `-p`, `--proxy` `<proxy>`
    
    对于一些无法连接 Telegram（如中国）使用代理，支持 `http`、`sock4`、`sock5`、`mtproxy` 等代理类型，具体使用见后文
    

### 代理 [#](#%e4%bb%a3%e7%90%86)

`mtproxy` 和 `sock4` 代理可直接在 `-p` 或者 `--proxy` 参数后使用，对于 `http` 或 `sock5` 代理需要安装 `pysocks`

**mtproxy 代理**

```
mtproxy://<secret>@<address>:<port>


```

**其他代理**

```
<protocol>://[<username>:<passwd>@]<address>:<port>


```

比如

```
# http 类型
http://username:passwd@1.2.3.4:1987
# socks4 类型
socks4://username:passwd@1.2.3.4:1980
# socks5 类型
socks5://username:passwd@1.2.3.4:1980


```

上传文件的过程指定代理使用参数 `-p` 或者 `--proxy` 加上代理格式

```
telegram-upload image1.jpg --proxy mtproxy://secret@proxy.my.site:443
telegram-upload image2.jpg --proxy http://username:passwd@1.2.3.4:1987
telegram-upload image3.jpg --proxy socks4://username:passwd@1.2.3.4:1980
telegram-upload image4.jpg --proxy socks5://username:passwd@1.2.3.4:1980


```

**环境变量**

除了设置代理参数你还可以使用环境变量，就可以不用每次都指定代理了，可以在终端定义环境变量以定义一下代理之一：`TELEGRAM_UPLOAD_PROXY`、`HTTPS_PROXY` 或 `HTTP_PROXY`

```
# 导入环境变量代理
export HTTPS_PROXY=socks5://username:passwd@1.2.3.4:1980
# 然后可以直接上传文件无需再设置代理
telegram-upload image.jpg
# 如果需要禁用代理
export TELEGRAM_UPLOAD_PROXY=
export HTTPS_PROXY=
export HTTP_PROXY=


```

注意：代理参数 `-p` 或者 `--proxy` 比环境变量指定的代理拥有更高的优先级，环境变量 `TELEGRAM_UPLOAD_PROXY` 优先级高于 `HTTPS_PROXY` 并且它优先于 `HTTP_PROXY`

防止任务中断 [#](#%e9%98%b2%e6%ad%a2%e4%bb%bb%e5%8a%a1%e4%b8%ad%e6%96%ad)
-------------------------------------------------------------------

telegram-upload 在终端里执行，如果终端断开，其进程就可能中断，因此在 Linux 下我们需要使用 screen 命令来让进程在后台执行会话

对于 Debian/Ubuntu 安装 screen

```
sudo apt-get update
sudo apt-get install screen


```

在安装好 screen 后，可以使用 screen 会话让我们的上传操作在后台进行且不会因为 SSH 断开而终止了

```
# 新建一个名为 tg 的 screen 会话（注意大小写）
screen -S tg  
# 然后在里面执行我们的 telegram-upload 命令，比如：
telegram-upload /telegram/xxx.zip
# 按Ctrl+A，再按"D"键,保留会话在 SSH 断开后继续执行
# 未来需要恢复名为 tg 的会话
screen -r tg
# 任务完成，需要关闭 tg 这个会话
exit


```