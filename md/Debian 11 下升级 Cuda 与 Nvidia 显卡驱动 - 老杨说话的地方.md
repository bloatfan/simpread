> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [yangyq.net](https://yangyq.net/2023/03/debian-11-cuda-nvidia-driver-upgrade.html)

> 前面文章写过，关于如何安装驱动和 Cuda，但是这种方法安装，驱动和 Cuda 的版本来自于软件库，通常来说，软件库的版本落后于最新版本。

[![](https://cdn.yangyq.net/wp-content/uploads/2022/06/debian.jpg)](https://cdn.yangyq.net/wp-content/uploads/2022/06/debian.jpg)

前面文章写过，关于如何[安装驱动和 Cuda](https://yangyq.net/2023/03/debian-11-nvidia-driver-cuda.html)，但是这种方法安装，驱动和 Cuda 的版本来自于软件库，通常来说，软件库的版本落后于最新版本。

部分框架对 Cuda 版本又有要求，太低了不能运行，所以需要对版本进行升级。

升级准备
----

升级前，需要将原有的驱动删除。否则会报错，例如：

```
下列软件包有未满足的依赖关系：
 libnppc11 : 冲突: nvidia-libopencl1 但是 530.30.02-1 正要被安装
E: 错误，pkgProblemResolver::Resolve 发生故障，这可能是有软件包被要求保持现状的缘故。


```

```
sudo apt-get --purge remove nvidia-driver
sudo apt-get --purge remove nvidia-cuda-dev nvidia-cuda-toolkit


```

当然，可以使用命令把所有相关的包全部删除。

```
sudo apt-get --purge remove nvidia*


```

然后再执行：

把不再需要的依赖也删除，否则还会有冲突。

获取安装包
-----

到 Cuda 的[官方网站](https://developer.nvidia.com/cuda-downloads)进行下载。

根据自己的操作系统等信息进行选择后，会给一个提示，这里最好使用 deb（local）的方式下载到本地进行安装，比较快一些。

升级命令
----

接下来就比较简单了。

```
cd ~
wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda-repo-debian11-12-1-local_12.1.0-530.30.02-1_amd64.deb
sudo dpkg -i cuda-repo-debian11-12-1-local_12.1.0-530.30.02-1_amd64.deb
sudo cp /var/cuda-repo-debian11-12-1-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo add-apt-repository contrib
sudo apt-get update
sudo apt-get -y install cuda


```

重启系统后就可以看到更新后的版本了。