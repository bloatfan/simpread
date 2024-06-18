> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.xm.mk](https://blog.xm.mk/posts/0e2c/)

> 迷失的人迷失了，相逢的人会再相逢

> 本文撰写时，我使用的 PVE 版本是 8.0.3。

[](#%e4%b8%8b%e8%bd%bd%e9%95%9c%e5%83%8f%e5%92%8c%e9%a9%b1%e5%8a%a8)下载镜像和驱动
---------------------------------------------------------------------------

1.  访问网站 [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/) 下载 Virtio 驱动，例如 `virtio-win-0.1.240.iso`，然后上传到 PVE。
    
2.  从 [MSDN ITellYou](https://next.itellyou.cn/) 下载 Windows 镜像，上传到 PVE。
    

[](#%e5%bc%80%e5%a7%8b%e5%ae%89%e8%a3%85)开始安装
---------------------------------------------

### [](#%e5%88%9b%e5%bb%ba%e8%99%9a%e6%8b%9f%e6%9c%ba)创建虚拟机

1.  点击「创建虚拟机」，并起名。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.10.46@2x.webp)
2.  加载镜像，并选择对应系统版本，我这里用的是 Windows 10。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.11.42@2x.webp)
3.  机型选择 Q35，BIOS 选择 OVMF，Windows 11 必须添加 TPM，其他可选。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.12.45@2x.webp)
4.  磁盘选择 「VirtlO Block」，设置需要的磁盘大小。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.13.50@2x.webp)
5.  分配 CPU，类别选择 「host」，并在下一步设置内存大小。根据宿主机的 CPU 内存情况和操作系统，可以选择 4C8G 或更高。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.14.42@2x.webp)
6.  添加网络设备，桥接能访问外网的网卡，模型修改为 「VirtIO」。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.26.54@2x.webp)
7.  最后确认配置，但是**不要勾选「创建后启动」**。
    

### [](#%e5%ae%89%e8%a3%85%e7%b3%bb%e7%bb%9f)安装系统

1.  到新建的虚拟机「硬件」页面，添加 CD/DVD，选择之前下载的 virtio 驱动镜像。然后在「选项」-「引导顺序」中，把系统安装 ISO 调整为第一位（此步无图）。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.18.15@2x.webp)
2.  现在可以启动虚拟机了，并连接到控制台，开始安装流程。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.21.16@2x.webp)
3.  安装选项选择自定义。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.22.15@2x.webp)
4.  这里会提示找不到驱动器，点击「加载驱动程序」，选择扫描到的对应系统驱动。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.22.42@2x.webp) ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.23.17@2x.webp)
5.  等待一段时间，走完正常安装流程。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.24.07@2x.webp)

### [](#%e5%90%8e%e7%bb%ad%e9%85%8d%e7%bd%ae)后续配置

#### [](#%e5%ae%89%e8%a3%85%e9%a9%b1%e5%8a%a8)安装驱动

重启开机后，走完开机向导，正常进入系统后，会发现没有网络，这是因为 VirtIO 的驱动还没有安装。按 Win + E 打开资源管理器，打开 virtio 的驱动盘，双击「virtio-win-gt-x64.exe」安装驱动

![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.31.37@2x.webp) ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.31.55@2x.webp)

#### [](#%e5%ae%89%e8%a3%85-qemu-guest-agent)安装 QEMU guest-agent

1.  为了确保系统能正常响应 PVE 的关机等事件，还需要安装 qemu-guest-agent，在 guest-agent 文件夹中，安装「qemu-ga-x86_64.exe」。 ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.36.04@2x.webp)
    
2.  然后在 PVE 的虚拟机「选项」页面，开启 QEMU Guest Agent，重启后，「概要」页面就可以显示虚拟机 IP 信息了。
    
    ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.36.55@2x.webp) ![](https://blog.xm.mk/posts/0e2c/PVE%20%E5%AE%89%E8%A3%85%20Windows.assets/CleanShot%202023-12-09%20at%2023.41.10@2x.webp)

[](#one-more-thing)One more thing
---------------------------------

安装完成后，最好给虚拟机拍个快照或者备份一下，避免以后折腾坏了还要从头开始～

[](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99)参考资料
---------------------------------------------

*   [Qemu-guest-agent - Proxmox VE](https://pve.proxmox.com/wiki/Qemu-guest-agent)
*   [佛西博客 - Proxmox VE 虚拟机无法关机的解决思路](https://foxi.buduanwang.vip/virtualization/pve/1647.html/)