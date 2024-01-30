> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/liugp/p/16950443.html) 目录

*   [一、背景](#一背景)
*   [二、在线 yum 安装](#二在线-yum-安装)
    *   [1）查看当前内核版本信息](#1查看当前内核版本信息)
    *   [2）导入仓库源](#2导入仓库源)
    *   [3）选择 ML 或 LT 版本安装](#3选择-ml-或-lt-版本安装)
    *   [4）设置启动](#4设置启动)
    *   [5）生成 grub 配置文件](#5生成-grub-配置文件)
    *   [6）重启](#6重启)
    *   [7）验证是否升级成功](#7验证是否升级成功)
    *   [8）删除旧内核（可选）](#8删除旧内核可选)
*   [三、离线 rpm 安装](#三离线rpm安装)
    *   [1）下载内核 RPM](#1下载内核-rpm)
    *   [2）安装内核](#2安装内核)
    *   [3）确认已安装内核版本](#3确认已安装内核版本)
    *   [4）设置启动](#4设置启动-1)
    *   [5）生成 grub 配置文件](#5生成-grub-配置文件-1)
    *   [6）重启](#6重启-1)
    *   [7）验证是否升级成功](#7验证是否升级成功-1)
    *   [8）删除旧内核（可选）](#8删除旧内核可选-1)

一、背景
----

在 CentOS 使用过程中，高版本的应用环境可能需要更高版本的内核才能支持，所以难免需要升级内核，所以以下将介绍 **yum 和 rpm 两种升级内核方式**。

关于内核种类:

*   `kernel-ml`——kernel-ml 中的 ml 是英文【 mainline stable 】的缩写，elrepo-kernel 中罗列出来的是最新的稳定主线版本。
    
*   `kernel-lt`——kernel-lt 中的 lt 是英文【 long term support 】的缩写，elrepo-kernel 中罗列出来的长期支持版本。ML 与 LT 两种内核类型版本可以共存，但每种类型内核只能存在一个版本。
    

二、在线 yum 安装
-----------

### 1）查看当前内核版本信息

```
uname -a
# 仅查看版本信息
uname -r
#  通过绝对路径查看查看版本信息及相关内容
cat /proc/version
#  通过绝对路径查看查看版本信息
cat /etc/redhat-release
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/3440a455083b4bc19ceccf5abd87c437.png)

### 2）导入仓库源

```
# 1、更新yum源仓库
yum -y update
# 2、导入ELRepo仓库的公共密钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# 3、安装ELRepo仓库的yum源
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
# 4、查询可用内核版本
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/9e05c46dc50f4db3a243316d6950baca.png)

### 3）选择 ML 或 LT 版本安装

```
# 安装 最新版ML 版本
# yum --enablerepo=elrepo-kernel install  kernel-ml-devel kernel-ml -y
# 安装 最新版LT 版本
# yum --enablerepo=elrepo-kernel install kernel-lt-devel kernel-lt -y

# 不带版本号就安装最新版本，这里我们安装 LT 5.4.225-1.el7.elrepo版本
# 安装 LT 版本，K8S全部选这个
yum --enablerepo=elrepo-kernel install kernel-lt-devel-5.4.225-1.el7.elrepo.x86_64 kernel-lt-5.4.225-1.el7.elrepo.x86_64 -y
```

安装完成后需要设置 grub2，即内核默认启动项

### 4）设置启动

> 内核安装好后，需要设置为默认启动选项并重启后才会生效。

查看系统上的所有可用内核

```
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ba1f5d133d7e4f56b834553ed4eaebcc.png)  
刚刚安装的内核即 0 : `CentOS Linux (5.4.225-1.el7.elrepo.x86_64) 7 (Core)`  
我们需要把 grub2 默认设置为 0  
可以通过 `grub2-set-default 0` 命令或编辑 `/etc/default/grub` 文件来设置

**方法 1：通过 grub2-set-default 0 命令设置**

```
grub2-set-default 0
```

**方法 2：编辑 /etc/default/grub 文件**

```
# 将GRUB_DEFAULT设置为0，如下
vim /etc/default/grub
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/2682d952883b4d788ed9f9b1f2c0879c.png)

### 5）生成 grub 配置文件

GRUB2 的配置文件通常为 /boot/grub2/grub.cfg，虽然此文件很灵活，但是我们并**不需要手写所有内容**。可以通过程序自动生成，或是直接修改生成之后的文件。通常情况下简单配置文件 `/etc/default/grub` ，然后用程序 `grub-mkconfig` 来产生文件 `grub.cfg`。

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 6）重启

```
# 重启(默认30秒)
reboot
# 立即重启
reboot -h now
```

### 7）验证是否升级成功

```
uname -a
# 仅查看版本信息
uname -r
#  通过绝对路径查看查看版本信息及相关内容
cat /proc/version
#  通过绝对路径查看查看版本信息
cat /etc/redhat-release
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ccabd581796a4015a695e2f1dc8931cc.png)

### 8）删除旧内核（可选）

查看系统中的全部内核

```
rpm -qa | grep kernel
# yum remove kernel-版本
yum remove kernel-3.10.0-1160.el7.x86_64 kernel-3.10.0-1160.71.1.el7.x86_64 kernel-tools-3.10.0-1160.71.1.el7.x86_64 kernel-tools-libs-3.10.0-1160.71.1.el7.x86_64
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/6d51796fd922453bb65085e48f31af84.png)

三、离线 rpm 安装
-----------

查找 kernel rpm 历史版本：[http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/](http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/)

### 1）下载内核 RPM

```
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-devel-5.4.225-1.el7.elrepo.x86_64.rpm
wget http://mirrors.coreix.net/elrepo-archive-archive/kernel/el7/x86_64/RPMS/kernel-lt-5.4.225-1.el7.elrepo.x86_64.rpm
```

### 2）安装内核

```
rpm -ivh kernel-lt-5.4.225-1.el7.elrepo.x86_64.rpm
rpm -ivh kernel-lt-devel-5.4.225-1.el7.elrepo.x86_64.rpm
```

### 3）确认已安装内核版本

```
rpm -qa | grep kernel
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/1afb3f5b47504416b11ba6a4708f6674.png)

### 4）设置启动

查看系统上的所有可用内核

```
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/c3c61e9dee524f73b2f4d5599b32f4a9.png)

```
grub2-set-default 0
```

### 5）生成 grub 配置文件

GRUB2 的配置文件通常为 `/boot/grub2/grub.cfg`，虽然此文件很灵活，但是我们并**不需要手写所有内容**。可以通过程序自动生成，或是直接修改生成之后的文件。通常情况下简单配置文件 `/etc/default/grub` ，然后用程序 `grub-mkconfig` 来产生文件 `grub.cfg`。

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 6）重启

```
# 重启(默认30秒)
reboot
# 立即重启
reboot -h now
```

### 7）验证是否升级成功

```
uname -a
# 仅查看版本信息
uname -r
#  通过绝对路径查看查看版本信息及相关内容
cat /proc/version
#  通过绝对路径查看查看版本信息
cat /etc/redhat-release
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/b9af5467035c4a009e1631792765e2b0.png)

### 8）删除旧内核（可选）

查看系统中的全部内核

```
rpm -qa | grep kernel
# yum remove kernel-版本
yum remove kernel-3.10.0-1160.el7.x86_64 kernel-3.10.0-1160.71.1.el7.x86_64 kernel-tools-3.10.0-1160.71.1.el7.x86_64 kernel-tools-libs-3.10.0-1160.71.1.el7.x86_64
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/6d51796fd922453bb65085e48f31af84.png)

Centos7 内核升级（5.4.225）升级就到这里了，有疑问的小伙伴欢迎给我留言，后续更新【云原生 + 大数据】相关的文章，请小伙伴耐心等待~