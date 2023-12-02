> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.lmshao.com](https://blog.lmshao.com/linux-glibc-version.html)

> glic 是 GNU 项目的标准 C 运行库。glibc 是 Linux 系统中最底层的 API，几乎其它任何运行库都会依赖于 glibc。

**glic** 是 GNU 项目的标准 C 运行库。glibc 是 Linux 系统中最底层的 API，几乎其它任何运行库都会依赖于 glibc。

Linux 各个发行版都是基于 Linux 内核的，也就是说理论上某个发行版编译出来的二进制文件在其他 Linux 发行版都可以运行，但是实际上并非如此，因为动态链接编译出来的可执行文件依赖的库其他系统可能不满足需求（相应的库不存在或者库的版本不满足需求）。

想要一次编译就能在其他 Linux 发行版运行，可以以静态链接的方式进行编译，但是大型软件都会依赖很多其他的第三方库，静态链接的方式可能不太现实。

动态链接的可执行文件在其他系统运行，可能会提示缺少某个动态库或者相应库版本过低，这时能想到的就升级更高版本的库。这种做法很对，但是**千万不要升级 glibc，千万不要升级 glibc，千万不要升级 glibc**，重要的事需要说三遍。

没有金刚钻就不要升级 glibc！因为 glibc 是系统最基础的 C 库，几乎所有的运行库都依赖它，特别是系统命令，一旦升级了 glibc 极有可能会导致很多系统命令都没法正常使用，这个系统基本上就报废了，这是很多 Linux 小白容易遇到的问题。

其他依赖的第三方库都可以升级，唯有 glibc 不建议升级。正确的做法是在同一版本或者更低版本 glibc 的系统上进行编译可执行文件。

下表是主流 Linux 发行版 Ubuntu/CentOS/Debian 系统默认的 glic 版本。

<table><thead><tr><th>Ubuntu</th><th>Debian</th><th>CentOS</th><th>Glibc</th></tr></thead><tbody><tr><td>22.04</td><td>-</td><td>-</td><td>2.34</td></tr><tr><td>20.04</td><td>11</td><td>-</td><td>2.31</td></tr><tr><td>-</td><td>10</td><td>8</td><td>2.28</td></tr><tr><td>18.04</td><td>-</td><td>-</td><td>2.27</td></tr><tr><td>-</td><td>9</td><td>-</td><td>2.24</td></tr><tr><td>16.04</td><td>-</td><td>-</td><td>2.23</td></tr><tr><td>14.04</td><td>8</td><td>-</td><td>2.19</td></tr><tr><td>13.04</td><td>-</td><td>7</td><td>2.17</td></tr><tr><td>12.04</td><td>-</td><td>-</td><td>2.15</td></tr><tr><td>-</td><td>7</td><td>-</td><td>2.13</td></tr><tr><td>-</td><td>-</td><td>6</td><td>2.12</td></tr></tbody></table>

通过上表可以看到 CentOS7 的 glibc 版本为 2.17，如果使用 CentOS7 系统编译的可执行文件，就可以在 Ubuntu13.04 以及更新版本和 Debian8 及更新版本上运行。

查看 Linux 系统 glibc 版本有多种方法：

**方法 1**

sh

```
ldd --version
```

**方法 2**

sh

```
find / -name libc.so.6


/xxxx/libc.so.6


strings libc.so.6 | grep GLIBC
```