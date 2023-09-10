> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.vvave.net](https://www.vvave.net/archives/set-up-systemd-log-automatic-delete-use-command-line.html)

> systemd 的日志占用空间会随着时间变长而逐步增长，那么如何进行清理系统日志的占用呢？

[systemd](https://www.vvave.net/tag/systemd/)

systemd 的日志占用空间会随着时间变长而逐步增长，那么如何进行清理系统日志的占用呢？

[systemd](https://en.wikipedia.org/wiki/Systemd) 的日志管理工具 journalctl 相当于 [init](https://en.wikipedia.org/wiki/Init) 的 syslog。

其所有的日志文件默认存放在 `/var/log/journal` 中，默认的配置下，这个文件夹的空间会逐步膨胀。

占用查询
----

使用命令即可查询当前 systemd 日志的空间占用情况

```
sudo du -sh /var/log/journal/
```

然而实际上，journal 自带了空间占用查询的命令

```
sudo journalctl --disk-usage
```

清理日志
----

journal 提供了多种维度的参数来清理日志，比如可以按天清理、按日期清理甚至可以按空间占用进行清理，比如：

*   按时间清理
    
    ```
    sudo journalctl --vacuum-time=7d # 清理7天之前的日志
    sudo journalctl --vacuum-time=2h # 清理2小时之前的日志
    sudo journalctl --vacuum-time=10s # 清理10秒之前的日志
    ```
    
*   按空间清理
    
    ```
    sudo journalctl --vacuum-size=1G # 保留 1GiB 最新的日志，超出的部分清理
    sudo journalctl --vacuum-size=100M # 保留 100MiB 最新的日志，超出的部分清理
    ```
    
*   按文件清理（与自动归档的时间间隔有关，比如当设置为一天时，每一天将生成一个日志文件）
    
    ```
    sudo journalctl --vacuum-files=5 # 仅保留最近的5个日志文件，超出的部分清理
    ```
    

自动化清理
-----

不想手动清理怎么处理？其实程序内已经自带了自动清理功能，只需要修改配置文件 `/etc/systemd/journald.conf` 中的参数即可实现。

```
[Journal]
Storage=auto
Compress=yes
Seal=yes
SplitMode=uid
SyncIntervalSec=5m
#RateLimitIntervalSec=30s
#RateLimitBurst=10000
#SystemMaxUse=100M
```

注意其中的参数 `SystemMaxUse` ，只需要配置这个参数即可实现系统自动滚动删除日志，比如将其配置为 `SystemMaxUse=128M` 即可将系统日志空间占用限制为 128Mb ，超过的部分会自动删除。

* * *

附录
--

### 参考链接

*   [systemctl 日志 自动删除 - 天行常](https://const.net.cn/508.html)

本文由 [柒](https://www.vvave.net/author/2/) 创作，采用 [知识共享署名 4.0](https://creativecommons.org/licenses/by/4.0/) 国际许可协议进行许可。  
转载本站文章前请注明出处，文章作者保留所有权限。  
最后编辑时间： 2022-06-06 11:00 AM