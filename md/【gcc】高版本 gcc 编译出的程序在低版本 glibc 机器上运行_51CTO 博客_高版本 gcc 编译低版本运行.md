> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/liangchaoxi/4689345)

> 【gcc】高版本 gcc 编译出的程序在低版本 glibc 机器上运行，

**目录**

​[​1. 静态编译（多数场景不行）​](#1.%E9%9D%99%E6%80%81%E7%BC%96%E8%AF%91%EF%BC%88%E5%A4%9A%E6%95%B0%E5%9C%BA%E6%99%AF%E4%B8%8D%E8%A1%8C%EF%BC%89)​

​[​2. 容器发布（部分场景可以使用）​](#2.%E5%AE%B9%E5%99%A8%E5%8F%91%E5%B8%83%EF%BC%88%E9%83%A8%E5%88%86%E5%9C%BA%E6%99%AF%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%EF%BC%89)​

​[​3. 安装部署 devtoolset​](#3.%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2devtoolset)​

​[​4. 打包依赖的 so 发布（通用方案）​](#3.%E6%89%93%E5%8C%85%E4%BE%9D%E8%B5%96%E7%9A%84so%E5%8F%91%E5%B8%83%EF%BC%88%E9%80%9A%E7%94%A8%E6%96%B9%E6%A1%88%EF%BC%89)​

​[​3.1 方式 1 在编译时设置 rpath 和 dynamic linker​](#3.1%20%E6%96%B9%E5%BC%8F1%20%E5%9C%A8%E7%BC%96%E8%AF%91%E6%97%B6%E8%AE%BE%E7%BD%AErpath%E5%92%8Cdynamic%20linker)​

​[​3.2 方式 2 直接修改二进制程序的 rpath 和 interpreter​](#3.2%20%E6%96%B9%E5%BC%8F2%20%E7%9B%B4%E6%8E%A5%E4%BF%AE%E6%94%B9%E4%BA%8C%E8%BF%9B%E5%88%B6%E7%A8%8B%E5%BA%8F%E7%9A%84rpath%E5%92%8Cinterpreter)​

* * *

比如我们用 gcc 9.3.0 编译程序，但需要发布的机器 gcc 版本是 4.8.5，怎么办？

你可能想到如下方法

2.  静态编译
3.  容器发布
4.  打包依赖的 so，使用本地 so 运行程序

1. 静态编译（多数场景不行）
---------------

其中静态编译是行不通的，libstdc++ 是可以静态编译，但是 libc 没有提供这方面的功能，即使你是 cpp 程序，依然会大概率依赖 libc.so

可以通过​`​nm <bin> | grep GLIBC_​`​确定你的程序是否依赖了 glibc，没有的话，你可以考虑直接静态编译 libstdc++。

2. 容器发布（部分场景可以使用）
-----------------

使用携带 gcc9.3.0 环境的容器发布程序，是可以的。但是在一些没有容器且没有 sudo 权限的场合，依然不太友好。

3. 安装部署 devtoolset
------------------

从源码安装的难度不低，向了​`​devtoolset可能是不错的选择​`​

RedHat 推出​`​Software Collections​`​的目的就是为了解决想在​`​RedHat​`​系统下能使用新版本的工具，让同一个工具（如 gcc）的不同版本能在系统中共存，在需要的时候切换到对应的版本中

如何安装和使用​`​devtoolset​`​，可以参考​[​官方指导文档​](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-7/)​。在看装前，我们可以先到​[​Information for build​](https://cbs.centos.org/koji/buildinfo?buildID=23609)​搜索自己需要的 package 对应的源码版本，看​`​devtoolset​`​中的安装包对应的源码版本是多少，是否满足自身的需求。

​[​devtoolset-8-gcc-8.3.1-3.2.el7 | Build Info | CentOS Community Build Service​](https://cbs.centos.org/koji/buildinfo?buildID=29150)​

*   到​[​sclo7-devtoolset-8-rh-candidate​](https://cbs.centos.org/repos/sclo7-devtoolset-8-rh-candidate/x86_64/os/Packages/)​下载全部 RPM 包 (这里偷懒下载全部包，这样就不用处理他们之间复杂的依赖关系)
*   下载完成后，在 RPM 包的目录中执行​`​yum install *.rpm​`​，安装全部
*   然后执行​`​scl enable devtoolset-8 bash​`​切换环境。

```
> stap -V # 我们可以执行这个命令进行验证，我们得到了Systemtap-3.3，支持linux kernel 2.6.28-4.18
Systemtap translator/driver (version 3.3/0.173, rpm 3.3-1.el7)
Copyright (C) 2005-2018 Red Hat, Inc. and others
This is free software; see the source for copying conditions.
tested kernel versions: 2.6.18 ... 4.18-rc0
enabled features: AVAHI BOOST_STRING_REF DYNINST LIBRPM LIBSQLITE3 NLS NSS READLINE
```

 ​[​安装最新版 devtoolset-8 — 源代码​](https://lrita.github.io/2018/11/28/upgrade-newest-devtoolset/)​

4. 打包依赖的 so 发布（通用方案）
--------------------

这个方法虽然听起来不是很优雅，但其实如果你对 elf 文件有一些了解，是不错的方式。下面说下具体的方法。

### 3.1 方式 1 在编译时设置 rpath 和 dynamic linker

当你有条件获得程序源码，并能够重新编译时，可以直接在编译时指定相关参数来解决。

先说编译时要增加的参数：

```
# 绝对路径
gcc -Wl,-rpath='/my/lib',-dynamic-linker='/my/lib/ld-linux.so.2'
```

gcc 参数

> -Wl,option  
> Pass option as an option to the linker.

ld 参数

> -rpath=dir  
> Add a directory to the runtime library search path. This is used when linking an ELF executable with shared objects.  
> --dynamic-linker=file  
> Set the name of the dynamic linker.

这两个参数分别设置的 elf 文件中的 rpath 和 interpreter 字段。

**rpath**

全名​`​run-time search path​`​，是 elf 文件中一个字段，它指定了可执行文件执行时搜索 so 文件的第一优先位置，一般编译器默认将该字段设为空。elf 文件中还有一个类似的字段 runpath，其作用与 rpath 类似，但搜索优先级稍低。搜索优先级:

```
rpath > LD_LIBRARY_PATH > runpath > ldconfig缓存 > 默认的/lib,/usr/lib等
```

如果你需要使用相对路径指定 lib 文件夹，可以使用​[​ORIGIN​](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.technovelty.org%2Flinux%2Fexploring-origin.html)​变量，ld 会将 ORIGIN 理解成可执行文件所在的路径。

```
gcc -Wl,-rpath='$ORIGIN/../lib'
```

**interpreter**

全名​`​elf interpreter​`​，用于加载 elf 文件。这个字段在链接时会帮你自动设置，64bit 程序一般为​`​/lib64/ld-linux-x86-64.so.2​`​。这也是打包 so 的坑之一，很多人（比如我）通过 ldd 找出程序依赖的 so，进行打包后，在目标机器修改 rpath 或者 LD_LIBRARY_PATH 指向本地 lib 目录，但 ldd 程序，发现 / lib64/ld-linux-x86-64.so.2 这个 so 仍然指向系统 so。原因就是这个字段是写死在 elf 文件中的，并不受 LD_LIBRARY_PATH 影响。

编译时带上这两个参数，下面只需要将你程序依赖的 so 打包一份，随程序进行发布即可。

### 3.2 方式 2 直接修改二进制程序的 rpath 和 interpreter

（​`​patchelf​`​具体用法见： 

当你无法编译程序时，也可以通过其他方式修改 rpath 和 interpreter。这种情况需要使用到一个工具​`​patchelf​`​，通过​`​dnf install patchelf​`​即可安装。你可以通过它修改 elf 文件的 rpath 和 interpreter：

```
patchelf --set-rpath /my/lib your_program
patchelf --set-interpreter /my/lib/ld-linux.so.2 your_program
```

除了绝对路径，一种比较常见的方式是在部署前，使用​`​pwd​`​获取当前路径，使用相对路径指向本地 lib。

```
patchelf --set-rpath `pwd`/../lib your_program
patchelf --set-interpreter `pwd`/../lib/ld-linux-x86-64.so.2 ./your_program
```

摘抄和参考：

 ​