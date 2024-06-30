> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [pickstar.today](https://pickstar.today/2023/01/%E8%AE%A9%E4%BD%A0%E7%9A%84vps%E6%8B%A5%E6%9C%89%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83%EF%BC%88ubuntu-debian%E7%AF%87%EF%BC%89/)

> 多数时候，我们使用 ssh 来管理远程服务器，可是有时候还是觉得有一个桌面环境更加得心应手，这时候我们就可以通过一 […]

本文最后更新于 482 天前，其中的信息可能已经有所发展或是发生改变。

多数时候，我们使用 ssh 来管理远程服务器，可是有时候还是觉得有一个桌面环境更加得心应手，这时候我们就可以通过一点简单的配置，让其变身为可视化服务器，并且能够通过微软的远程桌面连接直接操作。这里我们以 Ubuntu22 和 Debian11 为例，来体验服务器可视化环境的乐趣。

由于可视化桌面环境需要占用较多的服务器资源，远程桌面也需要稳定高速的网络环境，因此尽量保证你的 vps 要满足以下两点

*   性能相对充足，尽量 2 核 2G 以上
*   网络连接较好，或有网络质量好的服务器来做流量中转

首先更新系统

```
	sudo apt update -y && sudo apt upgrade -y

```

然后选择要安装的桌面环境，选择 Gnome，Xfce 或是 KDE 桌面都可以，看个人喜好

如果是 Ubuntu 的话，执行下面的命令安装，安装一个就可以了

```
	sudo apt install ubuntu-desktop -y # Gnome
	sudo apt install xubuntu-desktop -y # Xfce
	sudo apt install kubuntu-desktop -y # KDE

```

如果是 Debian 的话，同样安装一个就可以了，不过 debian11 无法使用 xrdp 远程 Gnome 桌面，尽管下面列出了且可以安装，但用不了，所以请不要选择

```
	sudo apt install task-gnome-desktop -y # Gnome
	sudo apt install task-xfce-desktop -y # Xfce
	sudo apt install kde-plasma-desktop -y # KDE

```

启动时默认启动图形化桌面环境

```
	sudo systemctl set-default graphical.target

```

为了安全，我们新建一个用户并赋予 sudo 权限用来登录桌面环境

这里以创建 testuser 为用户名的用户为例

```
	sudo useradd -d /home/testuser -m testuser
	sudo passwd testuser
	usermod -a -G sudo testuser

```

桌面环境配置好了，现在需要安装 rdp 服务器供远程访问

查看 xrdp 运行状态

```
	sudo systemctl status xrdp

```

如果看到状态是 active，就没有问题

```
	● xrdp.service - xrdp daemon
	     Loaded: loaded (/lib/systemd/system/xrdp.service; enabled; vendor preset: enabled)
	     Active: active (running) since Sat 2022-09-03 04:02:52 CEST; 7min ago
	       Docs: man:xrdp(8)
	             man:xrdp.ini(5)
	   Main PID: 107484 (xrdp)
	      Tasks: 1 (limit: 4620)
	     Memory: 1.7M
	     CGroup: /system.slice/xrdp.service
	             └─107484 /usr/sbin/xrdp

```

默认情况下，Xrdp 使用 /etc/ssl/private/ssl-cert-snakeoil.key, 它仅仅对 “ssl-cert” 用户组成语可读。运行下面的命令，将 xrdp 用户添加到这个用户组：

```
	sudo adduser xrdp ssl-cert  

```

重启 xrdp 服务

```
	sudo systemctl restart xrdp

```

xrdp 服务搭建好了，默认运行在 3389 端口，需要在防火墙放行此端口。不过需要注意，直接这么暴露端口在公网下是很危险的，推荐设定 ip 白名单。

如果需要完全开放 3389 端口的话，根据你正在使用的防火墙放行端口

```
	# ufw
	sudo ufw allow 3389
	# nft
	sudo nft add rule inet filter input tcp dport 3389 ct state new,established counter accept
	# iptables
	iptables -I INPUT -p tcp --dport 3389 -j ACCEPT
	# firewalld
	firewall-cmd --zone=public --add-port=3389/tcp --permanent

```

如果只想允许特定的 ip 或 ip 段访问的话，以 iptables 为例

```
	iptables -A INPUT -p tcp --dport 3389 -s YOUR_IP -j ACCEPT
	iptables -A INPUT -p tcp --dport 3389 -j DROP

```

将 YOUR_IP 更换为允许访问的 IP

虽然现在桌面环境和远程服务已经配置好了，但是如果直接使用的话，远程桌面的声音是无法播放的，如果想要正常播放声音，就需要安装声音转发模块。xrdp 服务中并未包含此模块，需要手动编译安装。

首先安装依赖

```
	sudo apt install git build-essential autoconf libtool automake pkg-config libssl-dev libpam0g-dev libx11-dev libxfixes-dev libxrandr-dev libpulse-dev -y

```

拉取源代码

```
	cd ~
	git clone https://github.com/neutrinolabs/pulseaudio-module-xrdp.git
	cd pulseaudio-module-xrdp

```

准备安装环境

```
	scripts/install_pulseaudio_sources_apt_wrapper.sh

```

这个脚本需要执行的时间比较长，请耐心等待，执行完后能看到

```
	- Creating focal build root. Log file in /var/tmp/pa-build-testuser-debootstrap.log
	- Creating schroot config file /etc/schroot/chroot.d/pa-build-testuser.conf
	[sudo] password for testuser: 
	- Copying /etc/apt/sources.list to the root
	- Creating the build directory /build
	- Copying the wrapped script to the build directory
	- Building PA sources. Log file in /var/tmp/pa-build-testuser-schroot.log
	- Copying sources out of the build root
	- Removing schroot config file and build root
	- All done. Configure PA xrdp module with PULSE_DIR=/home/testuser/pulseaudio.src

```

注意最后一行的输出，复制下 PULSE_DIR=/home/testuser/pulseaudio.src，粘贴在 configure 后，例如

```
	./bootstrap && ./configure PULSE_DIR=/home/testuser/pulseaudio.src

```

编译并安装

在远程桌面看，你会发现音频输出设备变成了 xrdp sink。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/640182c191bc5.webp)

如此以来，就可以将远程桌面的声音在本地播放了。

安装中文字体

```
	sudo apt install fonts-arphic-ukai fonts-arphic-uming fonts-ipafont-mincho fonts-ipafont-gothic fonts-unfonts-core -y

```

然后只需执行

```
	sudo dpkg-reconfigure locales

```

会看到

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/640182d56d5cd.webp)

向下找到 zh_CN.UTF-8，按空格选中后回车

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/640182ea543c9.webp)

系统默认语言也选中 zh_CN.UTF-8 即可

如果有的桌面环境中没有安装浏览器的话，可以这样一键安装 firefox

Ubuntu：

```
	sudo apt install firefox firefox-locale-zh-hans -y

```

Debian：

```
	sudo apt install firefox-esr -y

```

在远程访问的时候，可能会出现一些小问题，下面汇总几个常见的

（1）登录成功后就闪退

解决办法：在 ssh 中使用需要被远程登录的账户，按照你安装的桌面执行下面的命令，一般来说就没问题了。

```
	# gnome桌面
	echo gnome-session > ~/.xsession
	# xfce桌面
	echo xfce4-session >~/.xsession
	# kde桌面
	# echo "/usr/bin/startplasma-x11" > ~/.xsession

```

（2）登录成功后报错：something has gone wrong. A problem has occurred and the system can't recover.

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/640182fe8e4f1.webp)

我所遇到的情况是因为默认选择的桌面环境不对，用如下方法修复：

注：如果是 Debian11 且选择 Gnome 桌面看到这个错误，请放弃并选择其他桌面

```
	ls -l /etc/alternatives/x-session-manager
	sudo update-alternatives --config x-session-manager

```

例如我安装的额 xfce 桌面环境，就将默认值改为 3

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/64018318a3c47.webp)

选择号正确的桌面环境，重新登陆就能解决

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/6401832e53e21.webp)

Ubuntu Gnome

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/64018341e16d7.webp)

Debian Xfce

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/6401835b4c695.webp)

Debian kde

-=||=-__收藏