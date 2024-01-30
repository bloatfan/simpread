> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zyixinn.com](https://www.zyixinn.com/archives/centos%E5%8D%87%E7%BA%A7%E5%86%85%E6%A0%B8%E7%9A%84%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F) [随手记](https://www.zyixinn.com/categories/ssj "随手记")

CentOS 升级内核的三种方式
================

![zyixin](/upload/2022/03/%E5%A4%B4%E5%83%8F.jpg) [zyixin](https://www.zyixinn.com/s/about "zyixin") 2022-06-10 / 0 评论 / 0 点赞 / 2,523 阅读 / 1,716 字 06/10  温馨提示： 本文最后更新于 2023-02-09，若内容或图片失效，请留言反馈。部分素材来自网络，若不小心影响到您的利益，请联系我们删除。

CentOS 升级内核的三种方式(yum/rpm/源码)
============================

> 在 CentOS 使用过程中，难免需要升级内核，但有时候因为源码编译依赖问题，不一定所有程序都支持最新内核版本，所以以下将介绍三种升级内核方式。

**注意事项**

> 关于内核种类:  
> kernel-ml 中的ml是英文【 mainline stable 】的缩写，elrepo-kernel中罗列出来的是最新的稳定主线版本。  
> kernel-lt 中的lt是英文【 long term support 】的缩写，elrepo-kernel中罗列出来的长期支持版本。

> #检查内核版本  
> uname -r  
> 3.10.0-1160.53.1.el7.x86_64

一、yum安装
-------

### 1、导入仓库源

```
`rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm` 
```

### 2、查看可安装的软件包

```
`yum --enablerepo="elrepo-kernel" list --showduplicates | sort -r | grep kernel-ml.x86_64` 
```

### 3、选择 ML 或 LT 版本安装

无指定版本默认安装最新

```
`# 安装 ML 版本
yum --enablerepo=elrepo-kernel install  kernel-ml-devel kernel-ml -y   

# 安装 LT 版本，K8S全部选这个
yum --enablerepo=elrepo-kernel install kernel-lt-devel kernel-lt -y` 
```

### 4、查看现有内核启动顺序

```
`awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg` 
```

### 5、修改默认启动项

xxx 为序号数字，以指定启动列表中第x项为启动项，x从0开始计数

```
`grub2-set-default xxxx` 
```

**例如设置以4.4内核启动**

则直接输入“grub2-set-default 0”，下次启动即可从4.4启动

```
`# 查看内核启动序号
[root@ecs-2b3c ~] awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg

CentOS Linux (4.4.179-1.el7.elrepo.x86_64) 7 (Core)

CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)

CentOS Linux (0-rescue-6d4c599606814867814f1a8eec7bfd1e) 7 (Core)


# 设置启动序号
[root@ecs-2b3c ~] grub2-set-default 0

# 重启
reboot

# 检查内核版本
uname -r` 
```

二、RPM安装
-------

检查内核版本

```
`uname -r` 
```

### 1、查找版本

因 ELRepo 源都是最新版本，所以旧版本内核只能手动下载。

查找 kernel rpm 历史版：  
[http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/](http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/)

### 2、共需要下载三个类型 rpm

```
`wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-devel-5.4.203-1.el7.elrepo.x86_64.rpm
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-headers-5.4.203-1.el7.elrepo.x86_64.rpm
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-5.4.203-1.el7.elrepo.x86_64.rpm` 
```

### 3、安装内核

```
`rpm -ivh kernel-lt-5.4.203-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-lt-devel-5.4.203-1.el7.elrepo.x86_64.rpm
或者
#一键安装所有
rpm -Uvh *.rpm` 
```

### 4、确认已安装内核版本

```
`[root@ecs-2b3c ~]# rpm -qa | grep kernel
kernel-lt-devel-5.4.203-1.el7.elrepo.x86_64
kernel-devel-3.10.0-1160.53.1.el7.x86_64
kernel-lt-5.4.203-1.el7.elrepo.x86_64
kernel-3.10.0-1127.el7.x86_64
kernel-devel-3.10.0-1127.el7.x86_64
kernel-headers-3.10.0-1160.53.1.el7.x86_64
kernel-3.10.0-1160.53.1.el7.x86_64
kernel-tools-libs-3.10.0-1160.53.1.el7.x86_64
kernel-tools-3.10.0-1160.53.1.el7.x86_64` 
```

### 5、设置启动

```
`# 查看启动顺序
[root@ecs-2b3c ~]# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (5.4.203-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-1160.53.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-c9b49c6c11334518a7adc404ff6315b6) 7 (Core)

# 设置启动顺序
[root@ecs-2b3c ~]# grub2-set-default 0

# 重启生效
[root@ecs-2b3c ~]# reboot` 
```

三、源码安装
------

### 1、安装核心软件包

```
`yum install -y gcc make git ctags ncurses-devel openssl-devel
yum install -y bison flex elfutils-libelf-devel bc` 
```

### 2、创建内核编译目录

使用 home 下的 kernelbuild 目录

```
`mkdir ~/kernelbuild` 
```

### 3、获取内核源码

> 清华大学镜像站：[https://mirror.tuna.tsinghua.edu.cn/kernel/v4.x/?C=M&O=D](https://mirror.tuna.tsinghua.edu.cn/kernel/v4.x/?C=M&O=D)  
> 其他源码安装包下载地址：[https://mirrors.edge.kernel.org/pub/linux/kernel/](https://mirrors.edge.kernel.org/pub/linux/kernel/)
> 
> *   linux-4.xx.xx.tar.xz
> *   linux-4.xx.xx.tar.gz
> *   这两个格式都可以的，tar.xz压缩率更高，文件更小。

> 在线下载：wget [https://mirror.tuna.tsinghua.edu.cn/kernel/v4.x/linux-4.17.11.tar.xz](https://mirror.tuna.tsinghua.edu.cn/kernel/v4.x/linux-4.17.11.tar.xz)

### 4、解压内核代码

将其解压后进入源码目录:

```
`tar -xvJf linux-4.17.11.tar.xz` 
```

为确保内核树绝对干净，进入内核目录并执行 make mrproper 命令:

```
`cd linux-4.17.11
make clean && make mrproper` 
```

### 5、内核配置

复制当前的内核配置文件  
config-3.10.0-862.el7.x86_64是我当前环境的内核配置文件，根据实际情况修改

```
`cp /boot/config-3.10.0-862.el7.x86_64 .config` 
```

> 高级配置  
> y 是启用, n 是禁用, m 是需要时启用.  
> make menuconfig: 老的 ncurses 界面，被 nconfig 取代  
> make nconfig: 新的命令行 ncurses 界面

### 6、编译和安装

编译内核  
如果你是四核的机器，x可以是8

```
`make -j x` 
```

安装内核  
编译完内核后安装:Warning: 从这里开始，需要 root 权限执行命令，否则会失败.

```
`make modules_install install` 
```

### 7、设置启动

```
`查看启动顺序
# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (4.17.11-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (4.9.9-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-b91f945269084aa98e8257311ee713c5) 7 (Core)

设置启动顺序
# grub2-set-default 0
 
重启生效
# reboot` 
```

四、卸载 / 降级 内核
------------

> 例如:  
> 当系统已存在 LT 内核的 5.4.103 版本时，继续安装 LT 内核的 4.4.215 版本则会提示: package kernel-lt-5.4.103-1.el7.elrepo.x86_64 (which is newer than kernel-lt-4.4.215-1.el7.elrepo.x86_64) is already installed  
> 这时就需要进行内核降级，卸载最新版的内核。

### 1、查看系统当前内核版本

```
`[root@localhost ~]# uname -r
5.4.103-1.el7.elrepo.x86_64` 
```

### 2、查看系统中全部内核

```
`[root@localhost ~]# rpm -qa | grep kernel
kernel-headers-3.10.0-1160.15.2.el7.x86_64
kernel-devel-3.10.0-1160.49.1.el7.x86_64
kernel-tools-libs-3.10.0-957.el7.x86_64
kernel-3.10.0-957.el7.x86_64
kernel-ml-4.9.9-1.el7.elrepo.x86_64
kernel-lt-5.4.103-1.el7.elrepo.x86_64
kernel-tools-3.10.0-957.el7.x86_64
kernel-lt-devel-5.4.103-1.el7.elrepo.x86_64` 
```

### 3、删除指定内核

此处以删除 LT 内核的 5.4.103 版本为例

**注意：无法卸载当前在用的内核版本。卸载完后不一定需要重启**

```
`yum remove -y kernel-lt-devel-5.4.103-1.el7.elrepo.x86_64

yum remove -y kernel-lt-5.4.103-1.el7.elrepo.x86_64` 
```

检查卸载后内核版本

```
`[root@localhost ~]# rpm -qa | grep kernel
kernel-headers-3.10.0-1160.15.2.el7.x86_64
kernel-devel-3.10.0-1160.49.1.el7.x86_64
kernel-tools-libs-3.10.0-957.el7.x86_64
kernel-3.10.0-957.el7.x86_64
kernel-ml-4.9.9-1.el7.elrepo.x86_64
kernel-tools-3.10.0-957.el7.x86_64` 
```

0[](javascript:; "复制文章链接")[](http://service.weibo.com/share/share.php?sharesource=weibo&title=分享：CentOS 升级内核的三种方式，原文链接：https://www.zyixinn.com/archives/centos升级内核的三种方式&pic=https://www.zyixinn.com/upload/2023/02/Linux%E5%86%85%E6%A0%B8%E9%A6%96%E9%A1%B5log.png "分享到新浪微博")[](https://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=https://www.zyixinn.com/archives/centos升级内核的三种方式&sharesource=qzone&title=CentOS 升级内核的三种方式&pics=https://www.zyixinn.com/upload/2023/02/Linux%E5%86%85%E6%A0%B8%E9%A6%96%E9%A1%B5log.png "分享到QQ空间")[](https://connect.qq.com/widget/shareqq/index.html?url=https://www.zyixinn.com/archives/centos升级内核的三种方式&title=CentOS 升级内核的三种方式&pics=https://www.zyixinn.com/upload/2023/02/Linux%E5%86%85%E6%A0%B8%E9%A6%96%E9%A1%B5log.png "分享到QQ")[

微信扫一扫

](javascript:; "分享到微信")版权归属： zyixin 本文链接： [https://www.zyixinn.com/archives/centos升级内核的三种方式](https://www.zyixinn.com/archives/centos升级内核的三种方式) 许可协议： 本文使用《[署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)](//creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)》协议授权