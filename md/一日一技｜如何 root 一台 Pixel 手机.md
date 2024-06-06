> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/76276)

> 实际上，有关「如何 root 一台手机」的教程在网上已经实在是不一而足，本文更大的目的还是为我一位手持 Pixel 但至今没有弄明白 adb 命令的朋友所写，以如今使用最广泛的 Magisk 为例尽量简明的将 root ......

**2024 年 3 月更新：区分了 Pixel 7 及之后机型使用的不同 fastboot 指令。**

实际上，有关「如何 root 一台手机」的教程在网上已经实在是不一而足，本文更大的目的还是为我一位手持 Pixel 但至今没有弄明白 adb 命令的朋友所写，以如今使用最广泛的 Magisk 为例**尽量简明的将 root 的流程介绍出来**。

如果你更喜欢视频类的材料，那么我会推荐 MaowDroid 与极客湾的这两份教程：

*   [How to Root Google Pixel (2, 3/a, 4/a & XL) on Android 11 - YouTube](https://sspai.com/link?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DsPvnmgon89U)
*   [玩机必看！带你入坑安卓刷机，小白也能看懂的 ROOT 基础指南来啦！- 哔哩哔哩](https://www.bilibili.com/video/BV1BY4y1H7Mc/?spm_id_from=333.999.0.0&vd_source=390bcb7421539a7088d557446a17a7b1)

在电脑上配置 adb 环境
-------------

当然，折腾手机之前，你还需要一台运行正常的电脑—— Windows 或 Mac 均可——和一根功能无恙的数据线。

「adb」即 Android Debug Bridge ，亦称安卓调试桥，是谷歌为安卓开发者提供的开发工具之一，可以让你的电脑以指令窗口的方式控制手机。你可以在安卓开发者网页中的 [SDK 平台工具页面](https://sspai.com/link?target=https%3A%2F%2Fdeveloper.android.com%2Fstudio%2Freleases%2Fplatform-tools)下直接下载对应系统的 adb 配置文件，大小只有几十 MB 。

![](https://cdnfile.sspai.com/2022/10/19/article/bf97c38ea9978d0d965ab8294adf89de)

我们所需的 adb 配置文件会以压缩包的形式下载，将它解压成一个文件夹、保存在你常用的位置即可。至此，电脑端的 adb 环境已经部署完成，可以执行指令操作了。

对于 macOS 而言，你需要右键文件夹，在菜单中的「服务」中选择「新建位于文件夹位置的终端窗口」，将终端（Terminal）的执行路径设置在刚刚下载好的 adb 环境中：

![](https://cdnfile.sspai.com/2022/10/19/article/cab4003336dff4339b3b37f130299bc2)

当终端窗口出现后，你可以使用

```
./adb version
```

指令来验证是否成功的启动了 adb ，如果返回的文字中出现了「Android Debug Version 1.0.XX」之类的字样就说明启动成功、可以进行下一步工作了。

而对于 Windows 而言，则需要使用 Shift + 右键点击 platform-tools 文件夹，选择「在此处打开 PowerShell 窗口」。同样， Windows 可能也会要求你用特殊的语法来执行一个路径中的可执行指令，根据 PowerShell 的提示，使用 .\ 作为指令前缀

```
.\adb version
```

来检测 adb 是否启动成功：

![](https://cdnfile.sspai.com/2022/10/19/article/618d585d4940ddc06ffb80e867e79751)

后续的 adb 指令执行窗口统一以 macOS 示例， Windows 只需要将指令前缀从 ./ 改成 .\ 即可。

在手机上打开 adb 开关、解锁 Bootloader
---------------------------

**⚠️ 解锁 Bootloader 会让手机自动恢复出厂设置，在这之前请备份所有需要的资料**

电脑上的 adb 环境配置完成后，我们还需要在手机上手动启用 adb 调试。在 Pixel 的设置 - 关于手机页面中，连续点击最下面的版本号七八次，直到下面的弹窗提示「您现在处于开发者模式！」，如果你的手机设置了密码的话，则需要输入密码确认后才能启用开发者模式。

此时，在设置 - 系统的页面中就多出来了一行开发者选项的入口。进入开发者选项后，我们需要做三件事：

*   打开 USB 调试开关
*   关闭系统自动更新开关
*   打开 OEM 解锁开关（如果这个开关处于关闭状态并且是灰色的，说明你的 Pixel 是有锁机器，是无法 root 的；但如果是灰色并处于打开位置的话，则 Bootloader 已经解锁，可以直接进入下一步下载系统镜像了）

这时如果我们再将 Pixel 连接到电脑上，手机就会弹出提示窗口询问我们是否允许来自电脑的 USB 调试，勾选「一律允许使用这台计算机进行调试」，点击允许即可。

![](https://cdnfile.sspai.com/2022/10/19/article/232177cd7a8183d7d749d8b984f449fd)

允许来自电脑的 USB 调试之后，在电脑上输入

```
./adb devices
```

指令，如果指令下面返回了一个以你的手机序列号开头的 device ，说明手机上的 adb 也正常开启——在这之后，你就可以使用比如 adb install 、 push 、 reboot 等等指令在电脑上控制手机了。

![](https://cdnfile.sspai.com/2022/10/19/article/be9cd543e20a741790ee7222fbdc9f30)

手机与电脑上的 adb 都正常工作之后，使用

```
./adb reboot bootloader
```

指令将手机重启到 Bootloader（BL）界面。这时手机会进入 fastboot 模式，使用

```
./fastboot devices
```

确认电脑是否仍然可以读取到连接的 Pixel 手机，同样返回序列号说明手机连接正常。

不过在 Windows 中，电脑有可能会因为缺少对应的 USB 驱动而无法识别已经进入 fastboot 模式的手机，这时就需要从 Android 开发者网站下载 [Google USB Driver](https://sspai.com/link?target=https%3A%2F%2Fdeveloper.android.com%2Fstudio%2Frun%2Fwin-usb) ，进入 Windows 设备管理器手动更新外置 USB 设备的驱动程序。

![](https://cdnfile.sspai.com/2022/10/19/article/af465e881e4ee3e4b979bc84e2665691)

更新驱动程序后，电脑应该就可以正常识别处于 fastboot 模式的 Pixel 了。

确认连接正常后，输入

```
./fastboot flashing unlock
```

进行 Bootloader 的解锁，你的 Pixel 会进入一个询问页面，使用音量键选择到「Unlock the bootloader」，按电源键确认。至此，你的 Pixel 就已经解锁了最重要的 Bootloader 、并且自动执行格式化。

等待 Pixel 格式化完成后，你需要重新进行初始设置，并且再次进入设置启用开发者模式，将 USB 调试打开以便进行后续的操作。

下载你需要的系统镜像
----------

对于不同品牌的 Android 手机来说，如何寻找官方的系统镜像有时会成为刷机中最困难的一环，但是对于 Google Pixel 来说则完全不是问题。在 [Google Play services](https://sspai.com/link?target=https%3A%2F%2Fdevelopers.google.com%2Fandroid%2Fimages) 网站中，谷歌上传了从 Nexus S 到 Pixel 7 Pro 的**每一款机型在支持期间内每一个版本的系统**，并且附上了非常详细的刷写教程——对于 root 之外比如手动更新、系统回滚之类的需求，你甚至可以直接使用支持的浏览器在网页上在线刷机——不过对于网络环境的要求也相对较高。

![](https://cdnfile.sspai.com/2022/10/19/article/7e9a3f64305f94130194f191a5839adf)这里不仅有完整镜像，还有 OTA 更新包，等不及谷歌推送每月更新的话就可以自己手动 adb 更新

但是对于我们的 root 工作来说，则需要对谷歌原厂的系统镜像进行一些小的修改，因此我们需要根据右侧列出的机型列表，找到自己的 Pixel 机型当前的系统版本，点击「Link」将完整的系统镜像下载到本地。

我手中这台是 Pixel 5 ，开发代号红鳍鱼（redfin），在手机设置 - 关于手机页面的最底端可以看到当前的版本号 TP1A.220905.004 ，在 Google Play services 的页面中找到对应的镜像下载即可。

![](https://cdnfile.sspai.com/2022/10/19/article/9573e5f29aa742a2f06364d6f246f804)

完整的系统镜像同样是以一个压缩包的格式下载下来的。我们需要将其解压成文件夹，在里面找到一个以「image - 机型代号 - 版本号」命名的压缩包并再次解压，其中包含的这个 boot.img 就是我们稍后需要进行修改的文件。

**但是，如果你的手机是 Pixel 7 及以后的机型，那么就不要找 boot.img ，而是要找 init_boot.img 。**

![](https://cdnfile.sspai.com/2022/10/19/article/259e3efb807cd026c58f14599920814f)

安装 Magisk app
-------------

由知名 Android 开发者 John Wu 牵头开发的 Magisk（面具）是目前使用最广泛的 root 工具之一，在各类「一键 root」的时代已经过去、 Zygisk 生态正在成长的这段时间， Magisk 仍然是相对最容易上手的 root 首选项。

![](https://cdnfile.sspai.com/2022/10/19/article/9e0806c241dd95eb048c8f4a7e029f6c)

你可以直接从 Magisk 的 [GitHub 项目页面](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Ftopjohnwu%2FMagisk)下载最新版本的安装包。而如果你的 Pixel 系统版本是最新的 beta 版的话，则有可能需要参考 release note 在正式版与 Canary 版本（测试版）之间进行选择。

在 Magisk 的 apk 安装包下载完成之后，你可以在手机上直接安装。如果你是在电脑上下载的话，除了传输 apk 安装包之外，还可以用 adb 指令直接安装电脑上保存的安装包：

```
./adb install 安装包保存路径
例如：
./adb install /Users/postmeridy/Downloads/Magisk-25.2\(25200\).apk
```

不会写文件路径？没关系，你可以直接把 apk 安装包拖拽到 Terminal 或者 PowerShell 窗口中，它会被自动转换成文件路径；或者使用 option + 右键（在 Windows 上则是 Shift + 右键），在右键菜单中选择「复制文件路径」。

传输和修补镜像
-------

在手机上安装好最新版本的 Magisk 之后，我们就可以用它来修补之前在完整系统镜像中找到的 boot.img 启动文件了。

由于 macOS 默认不会将 Android 设备作为磁盘挂载，因此除了使用 adb 命令来传输和提取文件之外，更方便的方法往往是用一些第三方的文件管理工具。我个人比较推荐的是锤子的数字遗产 [HandShaker](https://sspai.com/link?target=https%3A%2F%2Fwww.smartisan.com%2Fapps%2F%23%2Fhandshaker) ，界面简洁的同时至少在 macOS Ventura beta 6 之前都保持着非常好的兼容性，是在 Mac 上管理 Android 设备文件的不二之选。

![](https://cdnfile.sspai.com/2022/10/19/article/7f8719829b7781b8a1f5c58bb5bc88b8)

同样，你可以直接在 Pixel 上通过浏览器下载或者用 adb 从电脑安装，将手机端与电脑端的软件都打开之后就可以传输文件了。为了方便起见，可以把 boot.img 存在手机根目录下的 Download 文件夹。

![](https://cdnfile.sspai.com/2022/10/19/article/514c073764de4fb2a3f627def0e539ce)

而如果你特别喜欢 adb 指令的话，则可以用 adb push 来将文件直接传入手机的指定位置：

```
./adb push 电脑上boot.img所在的文件路径 /手机上的保存路径
例如：
./adb push /Users/postmeridy/Downloads/boot.img /sdcard/Download
```

当 boot.img 传输完成后，我们就可以来到 Pixel 上面的 Magisk app 中，点击最顶上 Magisk 板块旁边的安装按钮。这时里面只会有一个名为「选择并修补一个文件」，点击并选中刚刚传输进手机的 boot.img 文件，点击开始后 Magisk 就会自动开始修补、并且将处理好的新 boot 镜像同样保存在根目录的 Download 文件夹里。

![](https://cdnfile.sspai.com/2022/10/19/article/13727b27da954ea9c00c00b688b57470)

当 Magisk 提示 All Done 修补完成之后，你就可以从 HandShaker 里面将修补过的镜像文件提取出来，保存在你熟悉的位置；或者使用 adb 指令：

```
./adb pull /sdcard/Download/修补后的镜像文件全名 ..
./adb pull /sdcard/Download/修补后的镜像文件全名 电脑上的文件路径
例如：
./adb pull /sdcard/Download/magisk_patched-25200_hjElM.img ..
./adb pull /sdcard/Download/magisk_patched-25200_hjElM.img /Users/postmeridy/Downloads/Workbench
```

前者会将新的镜像文件提取到执行这段指令的 platform-tools 文件夹所属的文件夹中，敲起来更快一些；后者则可以将文件提取到电脑上的任意什么位置，只需要将想要粘贴的地方以同样文件路径的形式复制进终端窗口即可。

从新的镜像启动并刷机
----------

将修改后的镜像保存到你指定的位置之后，我们就可以开始真正的刷写工作了。

使用

```
./adb reboot bootloader
```

再次让手机以 fastboot 模式启动到 Bootloader 中，使用

```
./fastboot devices
```

确认电脑正确识别手机。如果你的 Pixel 是 Pixel 6 代及以前的机型，使用：

```
./fastboot flash boot 修补后的镜像在电脑上的保存路径（或者直接拖拽进来）
例如：
./fastboot boot /Users/postmeridy/Downloads/magisk_patched-25200_hjElM.img
```

让你的 Pixel 从 Magisk 修补过的新 boot.img 启动。

如果你的 Pixel 手机是 Pixel 7 代及以后的机型的话，这里的指令就要改成：

```
./fastboot flash init_boot 修补后的镜像
```

这时手机会自动在新镜像传输进手机后重新启动，等到手机重新启动后，再次进入 Magisk app 里我们刚刚修补镜像的界面，不同的是，这次这里多出了两个选项。选择「直接安装（推荐）」， Magisk 就会自动完成剩下的工作：

![](https://cdnfile.sspai.com/2022/10/19/article/f362e2d54fdb18af623b731c16531834)状态栏电量归零是因为在「系统界面演示模式」打开时重启，与 Magisk 无关

至此，你的 Pixel 就已经 root 完成了，点击右下角的重启，让 Magisk 将修改后的启动镜像固化即可。

但有些时候，从新的启动镜像开机后， Magisk app 会提示需要对系统进行一些修补，点击确认后它会让手机再一次重启。那么再次重启后我们需要重复一遍上述的三个指令。因为重启后手机会恢复使用最开始没有被修改过的那个 boot.img ，而非我们需要的这个「有 Magisk 后门」的镜像。

当上述的步骤全部完成之后， Magisk 板块内的「当前」就会从「无法读取」变成 Magisk app 的版本号，表明已经取得 root 权限，同时下方也会显示「卸载 Magisk 」的按钮。

![](https://cdnfile.sspai.com/2022/10/19/article/17f946127e908ac677423a1383cf2adc)

安装模块
----

手机取得 root 权限只是一方面，想要让各类 root 工具大放异彩，还是得依靠各种模块。

以 Pixel 在国内的日常使用为例，如果你只是希望可以正常的使用比如 At a glance 中的天气功能和 Google Feed 之类的话，那么只需要这两个模块即可：

*   [Riru](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FRikkaApps%2FRiru)
*   [Location Report Enabler](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FRikkaApps%2FRiru-LocationReportEnabler)

将这两个模块的压缩包下载下来传到手机里面，在 Magisk 的「模块」页面中选择「从本地安装」，等待指令行跑完并重启就可以让模块生效了——记得先安装作为运行环境的 Riru 再安装别的模块：

![](https://cdnfile.sspai.com/2022/10/19/article/7ff551bf7b8500b3ddc6fc2f7bbf1f33)也可以一次性安装多个模块再重启

所有这些模块都应该是以压缩包的形式下载并通过 Magisk 安装的，如果你的浏览器在模块下载完成之后自动把它们给解压了，则需要调整一下浏览器的设置。

至此，你就拥有了一台 root 完成、可以随心折腾的 Google Pixel 啦。