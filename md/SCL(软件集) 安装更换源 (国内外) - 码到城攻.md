> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.codecomeon.com](https://www.codecomeon.com/posts/245/)

> SCL 是 Software Collections 的缩写，其收录了许多程序的新版本，安装的软件可与旧版共存。

SCL(软件集) 安装更换源 (国内外)
====================

[原创](/fromtype/1/ "Linux,SCL") [Linux 平台](/categories/4/) [2022 年 11 月 22 日 00:52](#) [夏至未至](#) [3431](#) 当前内容 1669 字，在路上，马上到，马上到

目录
--

*   [目录](#目录)
*   [更换步骤](#更换步骤)
    *   [更换 yum 国内源](#更换yum国内源)
    *   [安装 SCL 默认源](#安装SCL默认源)
    *   [编辑配置](#编辑配置)
        *   [国内镜像站](#国内镜像站)
        *   [国外镜像站](#国外镜像站)
    *   [刷新缓存](#刷新缓存)
    *   [测试验证](#测试验证)
        

更换步骤
----

### 更换 yum 国内源

这个可以参考：[Cannot download repodata/repomd.xml(更换 yum 源)](https://www.codecomeon.com/posts/240/ "Cannot download repodata/repomd.xml(更换yum源)")

### 安装 SCL 默认源

```
yum install -y centos-release-scl centos-release-scl-rh
```

安装完成后在 /`etc/yum.repos.d` 目录下会出现 `CentOS-SCLo-scl.repo` 和 `CentOS-SCLo-scl-rh.repo` 两个文件，安装后源默认启用。

### 编辑配置

#### 国内镜像站

只用配置如下小结内的即可，此外可点击 [开源镜像站](https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/ "开源镜像站") 查看站点是否可用。  
CentOS-SCLo-scl.repo：

```
[centos-sclo-sclo]
```

CentOS-SCLo-scl-rh.repo：

```
name=CentOS-7 - SCLo sclo
```

#### 国外镜像站

国外：[CentOS](https://vault.centos.org/6.10/sclo/x86_64/rh/ "CentOS")  
CentOS-SCLo-scl.repo：

```
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/
```

CentOS-SCLo-scl-rh.repo：

```
gpgcheck=0
```

### 刷新缓存

```
enabled=1
```

### 测试验证

后续操作，请参考：[CentOS7 快速升级 gcc 到高版本](https://www.codecomeon.com/posts/33/ "CentOS7快速升级gcc到高版本")

本文标题： [SCL(软件集) 安装更换源 (国内外)](/posts/245/) 本文作者： [夏至未至](http://www.codecomeon.com)  发布时间： 2022 年 11 月 22 日 00:52 最近更新： 2022 年 11 月 27 日 15:02 原文链接： [https://www.codecomeon.com/posts/245/](#) 许可协议： [ 署名 - 非商业性 - 禁止演绎 4.0 国际 (CC BY-NC-ND 4.0) ](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) [请按协议转载并保留原文链接及作者](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 

[Linux(24)](/tags/39/ "Linux") [SCL(1)](/tags/169/ "SCL")

[上一个

C++17 STL 标准库学习资料

](/posts/246/ "C++17 STL标准库学习资料")[下一个

Linux 误删除 libc.so.6(急救成功)

](/posts/244/ "Linux误删除libc.so.6(急救成功)")

当前文章评论暂未开放, 请移步至[留言](/posts/1/)处留言。