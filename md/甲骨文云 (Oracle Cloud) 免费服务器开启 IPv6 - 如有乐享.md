> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [51.ruyo.net](https://51.ruyo.net/17105.html)

> 2021 年 5 月 12 日 更新：服务器重启后 IPv6 无法自动获取，新增解决方案！参考 系统操作 第④步！ 自从 19 年博客首发了关于甲骨文云免费云服务器！不知不觉快 2 年了！这台 “永久” 免费的服务器还坚挺的呢......

![](https://raw.githubusercontent.com/lslz627/PicGo/master/7b0ac7e1b6b01a75.png)

**2021 年 5 月 12 日 更新：服务器重启后 IPv6 无法自动获取，新增解决方案！参考 [系统操作](#2) 第④步！**

自从 19 年博客首发了关于甲骨文云免费云服务器！不知不觉快 2 年了！这台 “永久” 免费的服务器还坚挺的呢！

21 年 4 月 15 日甲骨文官网突然宣布服务器都支持了 IPv6 了！-> [详细文章](https://blogs.oracle.com/cloud-infrastructure/ipv6-on-oracle-cloud-infrastructure) 

这个还是真的不错哇！虽然现在 IPv6 对国内的路由还是非常一般！但是这么好的资源肯定得试一试啦！

本文主要分为 2 部分操作！第一部分：控制台面板设置 IPv6 相关模块。第二部分：在 Linux 服务器上启动 IPv6。

博主已经把坑踩平了！大家可以试一试了！

**这里无需重新创建服务器即可添加 IPv6，也不用删除子网 (删除子网会导致 IP 变)** 

**博主最近几年陆续更新了一系列关于 [甲骨文云](https://51.ruyo.net/tag/oracle-cloud/) 的文章！也许有你需要的内容哦~**

面板操作
----

下面进入正题！登陆甲骨文后台！

**① 前往 网络 -> 虚拟云网络 -> 选择查看网络详情**

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**② 其实在控制台面板上主要下面 4 个步骤。**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/4102d923101948da.png)

**③ 打开 CIDR 块 -> 点击 【添加 IPv6 CIDR 块】**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ff26735aca77e40d.png)

添加成功后如图！

![](https://raw.githubusercontent.com/lslz627/PicGo/master/41bfc9364475a398.png)

**④  打开子网，编辑子网信息**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/54e28578ade60dbd.png)

勾选 **启用 IPV6 CIDR 块**

输入框随便输入一个值，例如：ee

点击保存！

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**这里如果出现下面的错误：NotAuthorizedOrNotFound，请移步到** [**处理错误**](#3) **部分内容解决！成功后再继续这里的步骤！！**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/14ce6bab26015daf.png)

**⑤ IPv6 CIDR 块添加成功！如图！**

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**⑥ 路由表 -> 路由表详情 -> 添加路由规则，如图设置即可！**

**目的地 CIDR 块：**::/0  (注意 2 个冒号)

**目标类型：**Internet 网关

![](https://raw.githubusercontent.com/lslz627/PicGo/master/d2869b9a9c4e4350.png)

**⑦ 安全列表 -> 查看详情 -> 添加出站规则 和 添加 入站规则**

**目的地类型：**CIDR

**目的地 CIDR：**::/0  (注意 2 个冒号)

**IP 协议：**所有协议

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ea0de4e051585083.png)

![](https://raw.githubusercontent.com/lslz627/PicGo/master/f7b57cd0b3537bd7.png)

**⑧ 查看服务器实例详情 -> 附加的 VNIC -> 点击 VNIC 详情**

右侧可见多了一个 IPv6 地址 的选项！点击 【分配 IPv6 地址】

![](https://raw.githubusercontent.com/lslz627/PicGo/master/d89520cc81edfa38.png)

**⑨ 可以指定一个你想要的 IPv6 格式，不指定会随机分配一个。**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/a570dedf7117decf.png)

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**⑩ 至此面板上的操作基本上完成了**

系统操作
----

主要以 CentOS7 举例。其他 Linux 版本请自行测试！

特别提醒一下，甲骨文的 CentOS 系统重启网卡会报错~ 所以通过重启网卡获取 IP 是行不通的。

**① 获取 IPv6（甲骨文网卡名称默认为 ens3）**

```
dhclient -6 ens3
```

**②查看 IPv6 是否生效**

```
ip add
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**③测试一下 IPv6 网络情况！**

```
ping6 google.com
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/0feb8935e4599a66.png)

**④添加开机启动**

服务器重启后，IPv6 不会动态获取！那么执行下面的脚本。把获取 IPv6 的命令写到开机启动！

```
chmod +x /etc/rc.d/rc.local
echo "dhclient -6 ens3" >> /etc/rc.d/rc.local
```

**处理错误**
--------

添加 IPv6 的时候 提示：NotAuthorizedOrNotFound

据好多童鞋反馈发生这个错误！

有大佬说，是因为没有将免费升级？或者由于试用期已过？这个我也不知道了！

下面说一下解决方案！首选打开 Cloud Shell 执行命令！

![](https://raw.githubusercontent.com/lslz627/PicGo/master/c3209a9857cb19d2.png)

**① 获取 compartment_id**

```
oci iam compartment list
```

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**② 查询子网 (subnet) 列表，获取到子网 ID(红框内) subnet_id**

PS：下面命令中的 [compartment_id] 替换为 上面的 compartment_id，不保留 [] 符号哦~

```
oci network subnet list --compartment-id [compartment_id]
```

![](https://raw.githubusercontent.com/lslz627/PicGo/master/34147b9a0c193932.png)

**如果你的子网是多个的话，这里会获取多个 id，自己创建时间辨别一下到底你操作的是哪个？不知道咋辨别，那就 2 个 ID 都试一试！**

**③ 获取 cidr，如图获取 CIDR 块地址！**

![](https://raw.githubusercontent.com/lslz627/PicGo/master/fe9c0b93e31b6889.png)

**④ 更新子网 (subnet) 信息**

将 [subnet_id] 和 [cidr] 替换一下！

```
oci network subnet update --subnet-id [subnet_id] --ipv6-cidr-block [cidr]
```

如果执行提示错误：The requested ipv6CidrBlock 2603:c1:3:b500::/56 is invalid: Subnet can have only 64 bit IPv6 CIDRs.

需要修改一下 cidr，2603:c1:3:b500::/**56** **->** 2603:c1:3:b500::/**64** 

然后再执行一下就成功啦！！!

**本文部分内容参考自 [@v2ex](https://www.v2ex.com/t/776048) 和 [@Luminous](https://luotianyi.vc/5558.html)**