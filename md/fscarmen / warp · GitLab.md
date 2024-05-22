> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [gitlab.com](https://gitlab.com/fscarmen/warp/)

> The script is a convenient tool designed to simplify the process of setting up a secure and reliable ......

[**README.md**](/fscarmen/warp/-/blob/main/README.md?ref_type=heads)

[](#wgcf%E8%BF%9E%E6%8E%A5cf-warp%E4%B8%BA%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%B7%BB%E5%8A%A0ipv4ipv6%E7%BD%91%E7%BB%9C)【WGCF】连接 CF WARP 为服务器添加 IPv4/IPv6 网络
======================================================================================================================================================

* * *

[](#%E7%9B%AE%E5%BD%95)目录
=========================

*   [更新信息](/fscarmen/warp/-/blob/main/README.md#%E6%9B%B4%E6%96%B0%E4%BF%A1%E6%81%AF)
*   [脚本特点](/fscarmen/warp/-/blob/main/README.md#%E8%84%9A%E6%9C%AC%E7%89%B9%E7%82%B9)
*   [WARP 好处](/fscarmen/warp/-/blob/main/README.md#WARP%E5%A5%BD%E5%A4%84)
*   [warp 运行脚本](/fscarmen/warp/-/blob/main/README.md#warp-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)
*   [warp-go 运行脚本](/fscarmen/warp/-/blob/main/README.md#warp-go-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)
*   [Cloudflare api 运行脚本](/fscarmen/warp/-/blob/main/README.md#cloudflare-api-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)
*   [通过 warp 解锁 chatGPT 的方法](/fscarmen/warp/-/blob/main/README.md#%E9%80%9A%E8%BF%87-warp-%E8%A7%A3%E9%94%81-chatgpt-%E7%9A%84%E6%96%B9%E6%B3%95)
*   [刷 Netflix 解锁 WARP IP 的方法](/fscarmen/warp/-/blob/main/README.md#%E5%88%B7-Netflix-%E8%A7%A3%E9%94%81-WARP-IP-%E7%9A%84%E6%96%B9%E6%B3%95)
*   [指定网站分流到 "socks5" 的 xray 配置模板 (适用于 WARP Client Proxy 和 WireProxy)](/fscarmen/warp/-/blob/main/README.md#%E6%8C%87%E5%AE%9A%E7%BD%91%E7%AB%99%E5%88%86%E6%B5%81%E5%88%B0-socks5-%E7%9A%84-xray-%E9%85%8D%E7%BD%AE%E6%A8%A1%E6%9D%BF-%E9%80%82%E7%94%A8%E4%BA%8E-warp-client-proxy-%E5%92%8C-wireproxy)
*   [指定网站分流到 "interface" 的 xray 配置模板 (适用于 WARP Client Warp 和 warp / warp-go 非全局)](/fscarmen/warp/-/blob/main/README.md#%E6%8C%87%E5%AE%9A%E7%BD%91%E7%AB%99%E5%88%86%E6%B5%81%E5%88%B0-interface-%E7%9A%84-xray-%E9%85%8D%E7%BD%AE%E6%A8%A1%E6%9D%BF%E9%80%82%E7%94%A8%E4%BA%8E-warp-client-warp-%E5%92%8C-warp--warp-go-%E9%9D%9E%E5%85%A8%E5%B1%80)
*   [WARP+ License 及 ID 获取](/fscarmen/warp/-/blob/main/README.md#warp-license-%E5%8F%8A-id-%E8%8E%B7%E5%8F%96)
*   [WARP Teams 获取并用于 Linux 的方法](/fscarmen/warp/-/blob/main/README.md#WARP-Teams-%E8%8E%B7%E5%8F%96%E5%B9%B6%E7%94%A8%E4%BA%8E-Linux-%E7%9A%84%E6%96%B9%E6%B3%95)
*   [WARP 网络接口数据，临时、永久关闭和开启](/fscarmen/warp/-/blob/main/README.md#warp-%E7%BD%91%E7%BB%9C%E6%8E%A5%E5%8F%A3%E6%95%B0%E6%8D%AE%E4%B8%B4%E6%97%B6%E6%B0%B8%E4%B9%85%E5%85%B3%E9%97%AD%E5%92%8C%E5%BC%80%E5%90%AF)
*   [WARP 原理](/fscarmen/warp/-/blob/main/README.md#WARP%E5%8E%9F%E7%90%86)
*   [鸣谢 WARP 贡献者和 CloudFlare WARP 全球站点服务状态列表](/fscarmen/warp/-/blob/main/README.md#%E9%B8%A3%E8%B0%A2%E4%B8%8B%E5%88%97%E4%BD%9C%E8%80%85%E7%9A%84%E6%96%87%E7%AB%A0%E5%92%8C%E9%A1%B9%E7%9B%AE)

* * *

[](#%E6%9B%B4%E6%96%B0%E4%BF%A1%E6%81%AF)更新信息
---------------------------------------------

2024.5.5 menu.sh v3.0.6 / warp-go.sh v1.1.8 Support Alpine edge system; 支持 Alpine edge 系统

2024.5.1 menu.sh v3.0.5 Deal with apt library changes for Debian 10 installations of wireguard-tools; 处理 Debian 10 安装 wireguard-tools 的 apt 库变更的问题

2024.4.14 menu.sh v3.0.4 1. Alpine check and update the wget version; 2. Add a message for feedback when connect warp fails; 1. Alpine 检测并更新 wget 版本的； 2. 获取 IP 失败时增加提示信息以便反馈

2024.3.21 menu.sh v3.0.3 / warp-go.sh 1.1.7 1. Update some commands according to warp-cli; 2. Remove the github cdn; 1. 根据 warp-cli 官方更新部分命令； 2. 去掉 Github cdn

历史更新 history（点击即可展开或收起）  

> 2024.2.7 menu.sh v3.0.2 To check if the WireGuard kernel module is already loaded. If not, attempt to load it and recheck; 判断系统是否已经加载 wireguard 内核模块，如果还没有则尝试加载，再重新判断
> 
> 2023.12.19 menu.sh v3.0.1 / warp-go.sh 1.1.6 Add a check to see if udp is allowed, if all endpoints of WARP are unreachable, the script will abort; 增加是否允许 udp 的检测，如果 WARP 的所有 endpoint 均不能连通，脚本将中止
> 
> 2023.8.22 menu.sh v3.0.0 / warp-go.sh 1.1.5 Add Github CDN; 添加 Github CDN
> 
> 2023.8.15 menu.sh v3.0.0 1. Add a non-global working mode, it can be switched use [warp g], which requires a script reinstallation; 2. Support regions sanctioned by Cloudflare, such as Russia, with a shared account; 3. IPv6 only uses the preset nat64 and restores the original nameserver file when uninstalled; 1. 增加 warp 的非全局工作模式，可以通过 [warp g] 切换，需要重装脚本; 2. 支持被 Cloudflare 制裁地区，如俄罗斯，使用共享账户; 3. IPv6 only 使用预设 nat64，卸载时恢复原始 nameserver 文件
> 
> 2023.7.21 menu.sh v3.0.0 beta2 1. If the system supports wireguard kernel and wireguard-go-reserved, it can be switched use [warp k], which requires a script reinstallation; 2. Support Fedora system; 3. Fix switch error caused by client version 2023.7.40-1; 1. 如果系统支持 wireguard kernel 和 wireguard-go-reserved，可以通过 [warp k] 切换，需要重装脚本; 2. 支持 Fedora 系统; 3. 修复 client 2023.7.40-1 版本导致的开关错误
> 
> 2023.6.30 menu.sh v3.0.0 beta IMPORTANT: 1. Use Cloudflare official warp api to replace wgcf; 2. Use wireguard-go with reserved to replace kernel. Make Hong Kong, Los Angeles and other restricted areas use warp; The above are the works of enthusiastic user, I would like to thank this guy and warp-go author coia for their contributions on behalf of all users of this script; 3. Since the changes are too big, please ask users to reinstall, if you have any problems, please feedback, I will deal with it as soon as possible; 重要更新: 1. 全面用 Cloudflare 官方 warp api 替代 wgcf; 2. 使用 wireguard-go with reserved 替代内核。使香港，洛杉矶等受限地区使用 warp; 以上均是热心网友的作品，我谨代表本脚本的所有用户感谢这位网友和 warp-go 作者 coia 的贡献; 3. 由于改动太大，请用户重新安装，如有问题请反馈，我将会尽快处理
> 
> 2023.6.27 menu.sh V2.53 Wireproxy proxy mode supports warp dualstack. From now on wgcf / wireproxy / client all support dual stack; Client Proxy 模式支持 warp 双栈， 从此之后 wgcf / wireproxy / client 通通支持双栈
> 
> 2023.6.21 menu.sh V2.52 1. Client proxy mode supports warp dualstack; 2. Client warp mode supports warp dualstack; 3. Speed up script startup; Thanks to Bro ⑥, WordsWorthLess, us254 and chika0801 for the guidace on the xray template; 1. Client Proxy 模式支持 warp 双栈; 2. Client warp 模式支持 warp 双栈; 3. 加快脚本启动速度; 感谢网友 ⑥哥, WordsWorthLess, us254 and chika0801 关于 xray 模板的指导
> 
> 2023.6.18 menu.sh V2.51 Client supports Debian 12 (bookworm); Client 支持 Debian 12 (bookworm)
> 
> 2023.5.20 menu.sh V2.50 1. Client supports IPv6 only VPS; 2. Support 4 ways to upgrade to teams account including token (Easily available at [https://web--public--warp-team-api--coia-mfs4.code.run](https://web--public--warp-team-api--coia-mfs4.code.run)); 3. Use api to delete warp account while uninstalling; 1. Client 支持 IPv6 only VPS 安装; 2. 支持包括 token 等 4 种方式升级为 teams 账户 (可通过 [https://web--public--warp-team-api--coia-mfs4.code.run](https://web--public--warp-team-api--coia-mfs4.code.run) 轻松获取); 3. 卸载的同时使用 api 删除 warp 账户
> 
> 2023.5.15 Cloudflare api Thanks to badafans open source project and patient guidance. Now released in linux using the Cloudflare WARP api. [badafans open source project](https://github.com/badafans/warp-reg) Use method 感谢大神 badafans 的开源项目及耐心指导，现发布在 linux 下使用的 Cloudflare WARP api，[badafans 的开源项目](https://github.com/badafans/warp-reg) 使用方法
> 
> ```
> wget -N https://gitlab.com/fscarmen/warp/-/raw/main/api.sh && bash api.sh [option]
> ```
> 
> 2023.5.10 warp-go V1.1.4 1. Docking the warp-go official account pool api, wiki: [https://docs.zeroteam.top/apis/warp](https://docs.zeroteam.top/apis/warp); 2. Change non-global from ipv4 only to dualstacks; 3. Fix the bug that the native IPv6 cannot login when using dualstacks; 4. Update the Best-enpoint app; 5. Change ip api; 1. 对接 warp-go 官方账户池 api，wiki: [https://docs.zeroteam.top/apis/warp](https://docs.zeroteam.top/apis/warp); 2. 非全局从 ipv4 only 改为双栈; 3. 修复双栈时使用原生 IPv6 不能登陆的 bug; 4. 更新最佳 Endpoint 应用; 5. 更换 ip api
> 
> 2023.3.26 warp-go V1.1.3 / menu.sh 2.49 1. Change the best Warp endpoint to standard ports [500,1701,2408,4500]; 2. Upgrade the Netflix unlocking section; 1. warp endpoint 优选改为标准端口 [500,1701,2408,4500]; 2. 升级奈飞解锁部分
> 
> 2023.3.14 warp-go V1.1.2 / menu.sh 2.48 To speed up WARP, automatically find the most suitable endpoint for local use and apply it to wgcf, warp-go and client. Thanks to an anonymous and enthusiastic user for the tool; 为了提速 WARP，自动寻找最适合本机使用的 endpoint，应用在 wgcf, warp-go 和 client，感谢匿名的热心网友提供的工具
> 
> 2023.3.2 warp-go V1.1.1 1. warp-go v1.0.8 is supported. Allowing custom MTU values in the configuration file /opt/warp-go/warp.conf; 2. Singbox configuration exports reseved using 3-numeric-array instead of a string; 1. 支持 warp-go v1.0.8 , 允许在配置文件 /opt/warp-go/warp.conf 自定义 MTU 值; 2. Singbox 配置导出 reseved 使用三个数字的数组代替字符串
> 
> 2023.2.22 [Unlock chatGPT without installing warp; 不安装 warp 就能解锁 chatGPT 的方法](/fscarmen/warp/-/blob/main/README.md#%E9%80%9A%E8%BF%87-warp-%E8%A7%A3%E9%94%81-chatgpt-%E7%9A%84%E6%96%B9%E6%B3%95)
> 
> 2023.2.7 menu.sh V2.47 Iptables + dnsmasq + ipset solution supports chatGPT. Install via the 12 option in the menu or `bash menu.sh e`; Iptables + dnsmasq + ipset 方案支持 chatGPT. 安装方式: 菜单 12 选项或者 `bash menu.sh e`
> 
> 2022.12.17 warp-go V1.1.0 Support OpenWrt system; 支持 OpenWrt 系统
> 
> 2022.12.10 warp-go V1.0.9 1.Export wireguard and sing-box config file with [warp-go e]; 2.Teams token website change to [https://web--public--warp-team-api--coia-mfs4.code.run](https://web--public--warp-team-api--coia-mfs4.code.run) 1. 使用 [warp-go e] 导出 wireguard 和 sing-box 配置文件; 2. 获取 teams token 网站更换为: [https://web--public--warp-team-api--coia-mfs4.code.run](https://web--public--warp-team-api--coia-mfs4.code.run)
> 
> 2022.10.19 menu V2.46 / warp-go V1.0.8 Switch the IPv4 / IPv6 priority by [warp s 4/6/d] or [warp-go s 4/6/d]; 通过 [warp s 4/6/d] 或者 [warp-go 4/6/d] 来切换 IPv4 / IPv6 的优先级别
> 
> 2022.10.7 warp-go V1.0.7 1. Further improve the conversion function between accounts. You can even switch from one WARP+ to another; 2. Formatting code； 1. 进一步完善账户间转换功能，你甚至可以从一个 WARP+ 换到另一个; 2. 优化代码
> 
> 2022.10.6 menu V2.45 1. Further improve the conversion function between accounts. You can even switch from one WARP+ to another; 2. Rebuild the account registration module; 1. 进一步完善账户间转换功能，你甚至可以从一个 WARP+ 换到另一个; 2. 重构账户注册模块
> 
> 2022.9.10 Over 2,000 users star. Thank you to every solution creator. I'm just passing these on more widely to serve more players. Thank you to each user for your continued support. I wish you all good health and Happy Mid-Autumn Festival!  
> 项目 star 达 2000。感谢每位解决方案创造者。我只是把这些作更广泛的传递，服务更多玩家。感谢各用户一如既往的支持。祝大家身体健康，中秋节快乐！
> 
> 2022.8.29 warp-go V1.0.6 1.Fixed the bug that routing rules failed after restart in non-global mode; 2.Fixed the bug of not changing IP; 1. 解决了非全局模式重启后，路由规则失效的 bug; 2. 解决了更换不了 IP 的 bug
> 
> 2022.8.27 menu V2.44 Refactoring the uninstallation logic. Dependency uninstallation requires confirmation; 重构卸载逻辑，依赖卸载需要确认
> 
> 2022.8.23 menu V2.43 warp-go V1.0.5 Support NAT VPS. Such as Woiden; 支持 NAT 服务器，例如 Woiden.
> 
> ~2022.8.21 After testing, wgcf warp service is back to normal in Hong Kong and Toronto, etc. 经测试，香港和多伦多等地区 wgcf warp 服务恢复正常~
> 
> 2022.8.21 menu V2.42 1.Add shortcut hints in the menu; 2.Remove the shortcut of S. Single and dual stacks swithing can directly use [warp 4/6/d]; 1. 在菜单中增加快捷方式的提示; 2. 移除快捷方式 s，单双栈相互切换可以直接 [warp 4/6/d]
> 
> 2022.8.20 warp-go V1.0.4 Chinese and English language support. Hello World; 中英双语支持，与世界接轨
> 
> 2022.8.20 warp-go V1.0.3 New feat: Menu + shortcuts for various usage scenarios; 菜单 + 快捷方式，适合各种使用场景
> 
> 2022.8.17 warp-go v1.0.2 1. Add WARP IPv4 non-global and global switch echo other. To use the v2ray/xray configuration file for triage, refer to the template for Client WARP mode on the project homepage; 2. Output wgcf configuration file (warp-go e); 1. 在原来全局的基础上，新增 WARP IPv4 非全局方案，配合 v2ray/xray 配置文件来分流，参考项目主页的 Client WARP 模式的模版; 2. 输出 wgcf 配置文件 (warp-go e)
> 
> 2022.8.13 warp-go v1.0.1 1.New feat: Support WARP+ (warp-go a); 2.Support Teams (warp-go a token). You can easily get the token through [https://warp-team-api.herokuapp.com/](https://warp-team-api.herokuapp.com/) ; 3.Brush unlock Netflix IP (warp-go i); 4.Support AMD v2 v3 v4 instruction set; 1. 新增 WARP+ 升级功能 (warp-go a ); 2. 新增 Teams 升级功能 (warp-go a token), 通过 [https://warp-team-api.herokuapp.com/](https://warp-team-api.herokuapp.com/) 你能轻松获取 token; 3. 新增刷解锁奈飞 IP 功能 (warp-go i); 4. 支持 GOAMD64v4 等指令集，提升功能
> 
> 2022.8.13 First on the whole web, proudly presents [@CoiaPrant](/CoiaPrant "Coia Prant")'s warp-go one-click script. Using various interfaces of CloudFlare-WARP and integrating wireguard-go, it can completely replace WGCF. Save Hong Kong, Toronto, etc., and let VPS without official WARP also get WARP IP. Thanks [@CoiaPrant](/CoiaPrant "Coia Prant") and his team again. Project official address: [https://gitlab.com/ProjectWARP/warp-go/-/tree/master/](https://gitlab.com/ProjectWARP/warp-go/-/tree/master/)
> 
> 全网首发，隆重推出 [@CoiaPrant](/CoiaPrant "Coia Prant") 的 warp-go 一键脚本。使用 CloudFlare-WARP 的各类接口，集成 wireguard-go，可以完全替代 WGCF。 救活了香港、多伦多等，让没有官方 WARP 的 VPS 也可以获取 WARP IP。再次感谢 [@CoiaPrant](/CoiaPrant "Coia Prant") 及其团队。项目地址: [https://gitlab.com/ProjectWARP/warp-go/-/tree/master/](https://gitlab.com/ProjectWARP/warp-go/-/tree/master/)
> 
> ```
> wget -N https://gitlab.com/fscarmen/warp/-/raw/main/warp-go.sh && bash warp-go.sh [option] [lisence]
> ```
> 
> 2022.8.5 2.41 1.Get the traffic quota of WARP+ via API. Thanks to Oreo for technical support; 1. 通过 API 获取 WARP+ 剩余流量, 感谢猫佬的技术支持。
> 
> 2022.6.27 香港 IPv6 only 安装 Client 的方式，转自 LOC jhsyue 的技术贴:[wiki-hk-61.8 开启 warp 教程](https://hostloc.com/thread-1036792-1-1.html)
> 
> 2022.6.11 2.40 1.Support VPS-free LXC VPS; 1. 支持 VPS-free LXC VPS
> 
> 2022.5.25 2.39 1.Automatically sync the latest official versions of wgcf, CloudFlare client, wireguard-go and wireproxy every day, allowing users to have >the best performance with every installation; 2.Change the installation method of CloudFlare client, from APT/YUM repository to Package repository >repository; 1. 每天自动同步官方版本最新版本的 wgcf、 CloudFlare client、wireguard-go 和 wireproxy，让用户每次安装都能获得最佳性能; 2. 更换 CloudFlare client 的安装方式，从 >APT/YUM 库 改到 Package 库
> 
> 2022.5.18 2.38 1. Fully support Ubuntu 22.04 and CentOS Streams 9 LTS; 2. Optimize Debian to speed up installation; 1. 全面支持 Ubuntu 22.04 和 CentOS >Streams 9 LTS; 2. 优化 Debian 以提升安装速度
> 
> 2022.4.21 WARP one-click script on macOS. A VPN that fast,modern,secure by WireGuard tunnel and WARP service 全网首发: macOS 一键脚本， 一个为免费、快速、安全的 > 基于 WireGuard 隧道，WARP 服务的 VPN。你可以理解为白嫖 CloudFlare 的科学服务了，也不需要服务器。
> 
> 项目地址: [https://github.com/fscarmen/warp/tree/main/pc](https://github.com/fscarmen/warp/tree/main/pc)
> 
> 2022.4.8 2.37 1. First publication on a global scale: After WirePorxy, another major technological breakthrough -- WARP-Cli's WARP mode solution. Thanks >to the original creator -- Teacher LUBAN. It solves two major pain points: 1) The instability of the traditional proxy model; 2) Currently HK does not >have a WARP service; 1. 全网首发: 继 WirePorxy 之后，又一重大技术突破，WARP-Cli 的 WARP 模式方案，感谢原创者 LUBAN 老师，引用大神的思路，解决两大通点: 1) 传统 proxy 模式的 > 断流和慢; 2) 解决 HK 没有 WARP 服务
> 
> 2022.3.27 2.36 1. First publication on a global scale. By WireProxy, Wireguard client that exposes itself as a socks5 proxy; Ths Fangliding for the >information:[#113](https://github.com/fscarmen/warp/issues/113); 2. WARP+ and Teams can be used in WireProxy; 3. Systemd and change Netflix IP for >WireProxy. 1. 全网首发: 通过 wireproxy，让 WARP 在本地建议一个 socks5 代理。感谢风扇滑翔翼 提供的资讯:[#113](https://github.com/fscarmen/warp/issues/113); 2. WARP+ > 和 Teams 账户可用于 WireProxy 安装或者升级; 3. WireProxy systemd 进程守护，同时支持更换 Netflix IP
> 
> 2022.3.23 2.35 1.Support WARP on Debian9; 1. 支持 Debian 9 上安装 WARP
> 
> Over 1,000 users star. Thank you for your support. 项目 star 破千，感谢各用户的大力支持。
> 
> 2022.3.19 2.34: 1.Support Arch Linux. Ths @SE_dong; 1. 应呜呜冬 @SE_dong 的要求，新增 Arch Linux 的支持.
> 
> 2022.3.11 2.33: 1.First publication on a global scale. WARP Client support Ubuntu 18.04 and CentOS 7; 2. Open TUN for OVZ. You needn't setting it in the >control panel. Thx @Q_lilll; 1. 全网首发， WARP Client 支持 Ubuntu 18.04 and CentOS 7; 2. 感谢 @Q_lilll 提供方案，为 OVZ VPS 在线打开 TUN, 不需要到面板处理
> 
> 2022.2.25 2.32: 1.Change the WARP endpoint; 2. Sync the Netflix title with lmc999; 1. 更换 WARP 的 endpoint; 2. 同步 lmc999 的 Netflix 检测 title
> 
> 2022.2.15 Happy Lantern Festival. Bring you a new experience of docker unlock, another way to unlock Netflix. Project based on alpine, content wgcf and >unblocking Netflix scripts. Change unlock warp ip automatically. 元宵节快乐。为大家带来个 docker 解锁的全新体验，换个姿势解锁 Netflix。项目以 alpine 为基础系统，内含 >wgcf 和解锁 Netflix 脚本，自动切换解锁 WARP IP  
> [https://github.com/fscarmen/unlock_warp](https://github.com/fscarmen/unlock_warp)
> 
> 2022.2.11 2.31: 1.iptables + dnsmasq + ipset to unlock stream media. (Not available for IPv6 only VPS). It is better than setting the outbound in >xray/v2ray. 1.iptables + dnsmasq + ipset 最小化解锁流媒体，warp 只接管流媒体流量 (不适合 IPv6 only VPS)，比在 xray/v2ray 设置分流的方案要更好
> 
> 2022.1.25 Media unlock daemon. Check it every 5 minutes. If unlocked, the scheduled task exits immediately. If it is not unlocked, it will be swiped >successfully in the background. Advantages: Minimized use of system resources. Please support professional unlock one-click script: >[https://github.com/fscarmen/unlock_warp](https://github.com/fscarmen/unlock_warp)
> 
> 流媒体解锁守护进程, 定时 5 分钟检查一次, 遇到不解锁时更换 WARP IP，直至刷成功。请大家支持一下兄弟项目: [https://github.com/fscarmen/unlock_warp](https://github.com/fscarmen/unlock_warp)
> 
> ```
> bash <(curl -sSL https://raw.githubusercontent.com/fscarmen/unlock_warp/main/unlock.sh)
> ```
> 
> 2022.1.21 2.30: 1.All support WARP single-stack and dual-stack solutions. Switch to each other easily and quickly. Such as [warp s 4],[warp s 6],[warp s >d]; 2.Brush Netflix Unlock IP with the expect area. Such as [warp i hk]. You can use it with crontab,screen,nohup & etc. [Detail](/fscarmen/warp/-/blob/main/README.md#%E5%88%B7-Netflix-%E8%A7%A3%3E%E9%94%81-WARP-IP-%E7%9A%84%E6%96%B9%E6%B3%95); 3.Fixed stuck when brushing Netflix IP 1. 全面支持 WARP 单栈与双栈方案，简单并快速切换，如 [warp s 4],[warp s 6],[warp s d]; 2. 在刷解锁 Netflix WARP >IP 时可以带上期望的地区, 如 [warp i hk]。你可以结合 crontab,screen,nohup & 等方式使用, [详细方法](/fscarmen/warp/-/blob/main/README.md#%E5%88%B7-Netflix-%E8%A7%A3%E9%94%81-WARP-IP-%E7%9A%84%E6%96%B9%E6%B3%95); 3. 修正刷 Netflix IP 时可能发 > 生的卡死不动的 bug
> 
> ~To be updated: huanx and malikshi [#63](https://github.com/fscarmen/warp/issues/63) needs, hope like [P3terx]>([https://github.com/P3TERX/warp.sh/blob/main/warp.sh](https://github.com/P3TERX/warp.sh/blob/main/warp.sh)) scripts, all support WARP single-stack and dual-stack solutions. Plan to rebuild menu modules and >pass parameters with arrays.(DONE)~
> 
> ~待更新：唤醒大神和 malikshi [#63](https://github.com/fscarmen/warp/issues/63) 的需求，希望像 [P3terx](https://github.com/P3TERX/warp.sh/blob/main/warp.sh) 脚本 > 一样，全面支持 WARP 单栈和双栈方案。计划花点时间用数组重构菜单模块和传参。（已完成）~
> 
> 2022.1.11 2.26: 1.Asking the unlock Netflix region where you expect before brushing WARP IP; 2.Single and Dual stack switch to each other quickly. 1. 在刷 > 解锁 Netflix WARP IP 之前，让用户输入想要的区域的简写; 2. 单栈与双栈快速切换;
> 
> WARP docker solution support ARM64,AMD64 and s390x; WARP docker 方案支持 ARM64,AMD64 和 s390x CPU 架构 Dualstack on IPv4 only ,IPv6 only and native dualstack VPS is coming; 双栈 WARP docker 已经有方案，即将推出。(现 IPv6 only 和原生双栈的都能 WARP 双栈，差 IPv4 Only> 了)
> 
> 2022.1.6 重大技术突破，绝对原创，绝对原创，绝对原创。成功把 WGCF 配置文件与环境依赖分离。本着万物皆可 Docker 的思路，以超轻量级的 Alpine 为底包（底包 5M + 依赖 22M=27MB），配置放在 > 映射目录处，解决某些旧系统不能使用 WARP 服务的问题。~docker 安装依赖方案只能是单栈，并不能双栈。该技术已经是 WGCF 和 wireguard 的天花板，不服来辩。~
> 
> ```
> wget -N https://cdn.jsdelivr.net/gh/fscarmen/warp/docker.sh && bash docker.sh [option] [lisence] ## 安装 docker、拉镜像和安装容器
> 
> docker exec -it wgcf sh #部分系统在容器外 docker exec -it wgcf wg-quick up wgcf 不行，一定要分开执行的。进入容器
> 
> wg-quick up wgcf; exit #运行 WGCF 并退出容器。
> ```
> 
> ![](https://user-content.gitlab-static.net/02fd67eba62ada048635bccc80bb79ee8fdbdb08/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3134383334333335382d36376430303839612d353931652d346166322d393135632d6537323534323261356230652e706e67)
> 
> 2022.1.1 1.Happy new year bros. I wish everyone good health and lots of money. Thanks for your support to this project. This project belongs to bros, I >just summarized your fragmented information; 2.Add timestamp and running time while brushing Netflix IP. 1. 元旦快乐，祝各位身体健康，赚钱多多。本项目是属于网友们的，我只是把大家碎片化的信息汇总而已; 2. 新年第一更刷奈飞 IP 时加入时间戳和运行时长
> 
> 2021.12 29 大家可以试试另两位 WARP 作者脚本:  
> 1. 甬哥 `wget -N https://cdn.jsdelivr.net/gh/kkkyg/CFwarp/CFwarp.sh && bash CFwarp.sh`  
> 2.P3terx `bash <(curl -fsSL git.io/warp.sh) menu`
> 
> 2021.12.28 2.25: IMPORTANT: 1.First publication on a global scale. Support architecture s390x for IBM Linux One(Choose WARP ipv6 single stack),thx Brother Big B and Misaka; 2.Support Alpine Linux, thx Dong gua; 3.add whitelist. support Debian bookworm; 重要更新: 1. 全网首发，支持 IBM Linux One 的 s390x 架构 CPU (请选用 WARP ipv6 单栈)，感谢 Misaka 和大 B 哥借机器测试 2. 支持 Alpine Linux 系统，感谢 Dong gua 借机器测试 3. 支持 Debian bookworm 系统，增加白名单，遇到没有大版本号的系统可以往里面放
> 
> 2021.12.24 2.24: 1.The default language will set to the one selected during installation. `echo 'E' >/etc/wireguard/language; warp v`; 2.Support HAX LXC VPS. It needs run `until curl -s4m8 ip.gs; do warp n; done` to brush the warp network; 1. 默认语言设置为安装时候选择的,`中文 echo 'C' >/etc/wireguard/language; warp v`; 2. 支持 HAX LXC VPSlxc 机器母鸡资源不够，warp 需要不停的刷才能获取到 `until curl -s4m8 ip.gs; do warp n; done`
> 
> 2021.12.17 2.23: Support change the Netflix IP not only WGCF but also Socks5 Client. Both will keep the Plus status. Recommand runs under [screen]; 2.Support update to TEAM account online. [URL for you](https://gist.githubusercontent.com/fscarmen/56aaf02d743551737c9973b8be7a3496/raw/16cf34edf5fb28be00f53bb1c510e95a35491032/com.cloudflare.onedotonedotonedotone_preferences.xml) 1. 支持 WARP Interface 和 Socks5 Client 自动更换支持奈飞的 IP，两者都会保留 Plus 的状态，建议在 screen 下在后台运行, 如果是中文，需要 screen -U 解决乱码问题; 2. 支持在线升级为 TEAM 账户。 [这此获取 URL](https://gist.githubusercontent.com/fscarmen/56aaf02d743551737c9973b8be7a3496/raw/16cf34edf5fb28be00f53bb1c510e95a35491032/com.cloudflare.onedotonedotonedotone_preferences.xml)
> 
> 2021.12.14 2.22: 1.First in the whole network. Use WARP Team account instead of Plus. No need to brush Plus traffic any more. 50 user limited. return to version 2.21; 1. 全网首创，使用脚本提供 TEAM 账户替代 Plus，免刷流量。~翻车了，官方说了免费 team 有 50 个账户的限制，我心存侥幸，想着 1 个账户多人用，现在看来是行不通了，暂先回退到 2.21 版本~
> 
> 2021.12.11 2.21: 1.BoringTUN removed because of unstable; 2.Change the DNS to Google first. 3.Count the number of runs1.BoringTUN 因不稳定而移除 2. 域名解析服务器首先谷歌 3. 统计运行次数
> 
> 2021.12.04 2.20: IMPORTANT: First publication on a global scale. Reduce installation time by more than 50% through multi-threading. No need to wait for WGCF registering and MTU value searching time; 2.Recode EN/CH traslation through associative array. Smarter and more efficient. Thx Oreo. 重大更新：1. 全网首创，通过多线程，安装 WARP 时间缩短一半以上，不用长时间等待 WGCF 注册和寻找 MTU 值时间了; 2. 中英双语部分关联数组重构了，更聪明高效，感谢猫大
> 
> 2021.11.30 2.11: 感谢 luoxue-bot 原创，唤醒大神告知。 1.Changing Netflix IP is adapted from other authors [luoxue-bot]; 1. 更换支持 Netflix IP 改编自 [luoxue-bot] 的成熟作品
> 
> 2021.11.11 2.10: 1.Customize the priority of IPv4 / IPv6; 2.Customize the port of Client Socks5(default is 40000); 1. 自定义 IPv4 / IPv6 优先组别; 2. 自定义 Client Socks5 代理端 >> 口，默认 40000
> 
> 2021.11.06 2.09: 1.WARP Linux Client supported.Socks5 proxy listening on: 127.0.0.1:40000. Register and connnect need non-WARP IPv4 interface. Native IPv4 + WARP IPv6 is ok; >2.WARP+ license on Client supported; 3.Customize the WARP+ device name. 1. 支持 WARP Linux Client，Socks5 代理监听: 127.0.0.1:40000, 注册和连接需要非 WARP 的原生 IPv4，可以是：原生 > IPv4+ WARP IPv6; 2.Client 支持 WARP+ 账户升级和安装; 3. 自定义 WARP+ 设备名
> 
> 2021.11.01 2.08: 1.Serching the best MTU value for WARP interface automatically; 2.asn organisation for the VPS; 1. 自动设置最优 MTU; 2. 显示 asn 组织 (线路提供商)
> 
> 2021.10.29 2.07: 1.Support Chinese and English; 2.Optimize running speed; 3.fix startup at reboot bug; 1. 支持中英文，用户可自行选择; 2. 大幅优化速度; 3. 修复重启后启动 WARP 的 bug
> 
> 2021.10.23 2.06: 1. 添加自动检查是否开启 Tun 模块； 2. 提高脚本适配性; 3. 新增 hax、Amazon Linux 2 和 Oracle Linux 支持
> 
> 2021.10.15 2.05: 1.WGCF 自动同步最新的 2.2.9； 2. 升级了重启后运行 Warp 的处理方法，不再依赖另外的文件; 3. 修复 KVM 由免费账户升级为 Warp+ 账户的 bug
> 
> 2021.10.14 2.04: 1.LXC 用户自主选择 BoringTun 还是 Wireguard-go (BoringTun 用 Rust 语言，性能接近内核模块性能 ，稳定性与 VPS 有关；WireGuard-GO 用 Go 语言，性能比前者差点，稳定性高); 2. 增加限 > 制：原生双栈 VPS 只能用 Warp 双栈，bash menu.sh 1 会建议改为 Warp 双栈或退出; 3.Warp 断网后，运行 warp 会自动关闭通道和杀掉进程; 4. 脚本中止后，用 echo $? 显示 1, 即代表不成功 (原来为代表运行成功的 0)
> 
> 2021.10.12 2.03: 1. 对刷网络作了优化，加快了两次尝试之间的间隔时间，不会出现死循环，因为已经限制次数为 10 次，有明确的提示 2. 用 Rust 语言的 BoringTun 替代 Go 语言的 WireGuard-GO
> 
> 2021.10.10 2.02: 上游 ip.gs 用 wget 不稳定导致获取不了 IP 而一直在死刷，弃坑用 curl 替换，脚本检查到没有的话自动安装

[](#%E8%84%9A%E6%9C%AC%E7%89%B9%E7%82%B9)脚本特点
---------------------------------------------

*   支持 WARP+ 账户，附带第三方刷 WARP+ 流量和升级内核 BBR 脚本
*   普通用户友好的菜单，进阶者通过后缀选项快速搭建
*   智能判断 vps 操作系统：Ubuntu 16.04、18.04、20.04; Debian 9、10、11，CentOS 7、8; Alpine 和 Arch Linux，请务必选择 LTS 系统  
    智能判断硬件结构类型：AMD、ARM 和 s390x
*   结合 Linux 版本和虚拟化方式，自动优选三个 WireGuard 方案。  
    网络性能方面：内核集成 WireGuard＞安装内核模块＞BoringTun＞wireguard-go
*   智能判断 WGCF 作者 github 库的最新版本 （Latest release）
*   智能分析内网和公网 IP 生成 WGCF 配置文件
*   输出结果，提示是否使用 WARP IP ，IP 归属地

[](#warp%E5%A5%BD%E5%A4%84)WARP 好处
----------------------------------

*   支持 chatGPT，解锁奈飞流媒体
*   避免 Google 验证码或是使用 Google 学术搜索
*   可调用 IPv4 接口，使青龙和 V2P 等项目能正常运行
*   由于可以双向转输数据，能做对方 VPS 的跳板和探针，替代 HE tunnelbroker
*   能让 IPv6 only VPS 上做的节点支持 Telegram
*   IPv6 建的节点能在只支持 IPv4 的 PassWall、ShadowSocksR Plus+ 上使用

![](https://user-content.gitlab-static.net/655513e86a6f60123631112b86714c423b397645/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3134343633353031342d34633032373634352d306530392d346238342d386237382d3838623431663935303632372e706e67)

[](#warp-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)warp 运行脚本
-------------------------------------------------------

首次运行

```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh [option] [lisence/url/token]
```

再次运行

```
warp [option] [lisence]
```

<table dir="auto" data-sourcepos="257:3-283:35"><thead><tr data-sourcepos="257:3-257:51"><th data-sourcepos="257:4-257:29">[option] 变量 1 变量 2</th><th data-sourcepos="257:31-257:50">具体动作说明</th></tr></thead><tbody><tr data-sourcepos="259:3-259:16"><td data-sourcepos="259:4-259:6">h</td><td data-sourcepos="259:8-259:15">帮助</td></tr><tr data-sourcepos="260:3-260:44"><td data-sourcepos="260:4-260:6">4</td><td data-sourcepos="260:8-260:43">原无论任何状态 -&gt; WARP IPv4</td></tr><tr data-sourcepos="261:3-261:128"><td data-sourcepos="261:4-261:19">4 lisence name</td><td data-sourcepos="261:21-261:127">把 WARP+ Lisence 和设备名添加进去，如 <code data-sourcepos="261:74-261:123">bash menu.sh 4 N5670ljg-sS9jD334-6o6g4M9F Goodluck</code></td></tr><tr data-sourcepos="262:3-262:44"><td data-sourcepos="262:4-262:6">6</td><td data-sourcepos="262:8-262:43">原无论任何状态 -&gt; WARP IPv6</td></tr><tr data-sourcepos="263:3-263:46"><td data-sourcepos="263:4-263:6">d</td><td data-sourcepos="263:8-263:45">原无论任何状态 -&gt; WARP 双栈</td></tr><tr data-sourcepos="264:3-264:72"><td data-sourcepos="264:4-264:6">o</td><td data-sourcepos="264:8-264:71">WARP 开关，脚本主动判断当前状态，自动开或关</td></tr><tr data-sourcepos="265:3-265:21"><td data-sourcepos="265:4-265:6">u</td><td data-sourcepos="265:8-265:20">卸载 WARP</td></tr><tr data-sourcepos="266:3-266:52"><td data-sourcepos="266:4-266:6">n</td><td data-sourcepos="266:8-266:51">断网时，用于刷 WARP 网络 (WARP bug)</td></tr><tr data-sourcepos="267:3-267:39"><td data-sourcepos="267:4-267:6">b</td><td data-sourcepos="267:8-267:38">升级内核、开启 BBR 及 DD</td></tr><tr data-sourcepos="268:3-268:40"><td data-sourcepos="268:4-268:6">a</td><td data-sourcepos="268:8-268:39">免费 WARP 账户升级 WARP+</td></tr><tr data-sourcepos="269:3-269:120"><td data-sourcepos="269:4-269:14">a lisence</td><td data-sourcepos="269:16-269:119">在上面基础上把 WARP+ Lisence 添加进去，如 <code data-sourcepos="269:75-269:115">bash menu.sh a N5670ljg-sS9jD334-6o6g4M9F</code></td></tr><tr data-sourcepos="270:3-270:26"><td data-sourcepos="270:4-270:6">p</td><td data-sourcepos="270:8-270:25">刷 Warp+ 流量</td></tr><tr data-sourcepos="271:3-271:63"><td data-sourcepos="271:4-271:6">c</td><td data-sourcepos="271:8-271:62">安装 WARP Linux Client，开启 Socks5 代理模式</td></tr><tr data-sourcepos="272:3-272:56"><td data-sourcepos="272:4-272:6">l</td><td data-sourcepos="272:8-272:54">安装 WARP Linux Client，开启 WARP 模式</td></tr><tr data-sourcepos="273:3-273:120"><td data-sourcepos="273:4-273:14">c lisence</td><td data-sourcepos="273:16-273:119">在上面基础上把 WARP+ Lisence 添加进去，如 <code data-sourcepos="273:75-273:115">bash menu.sh c N5670ljg-sS9jD334-6o6g4M9F</code></td></tr><tr data-sourcepos="274:3-274:34"><td data-sourcepos="274:4-274:6">r</td><td data-sourcepos="274:8-274:33">WARP Linux Client 开关</td></tr><tr data-sourcepos="275:3-275:37"><td data-sourcepos="275:4-275:6">v</td><td data-sourcepos="275:8-275:36">同步脚本至最新版本</td></tr><tr data-sourcepos="276:3-276:24"><td data-sourcepos="276:4-276:6">i</td><td data-sourcepos="276:8-276:23">更换 WARP IP</td></tr><tr data-sourcepos="277:3-277:65"><td data-sourcepos="277:4-277:6">e</td><td data-sourcepos="277:8-277:64">安装 iptables + dnsmasq + ipset 分流流媒体方案</td></tr><tr data-sourcepos="278:3-278:39"><td data-sourcepos="278:4-278:6">w</td><td data-sourcepos="278:8-278:38">安装 WireProxy 解决方案</td></tr><tr data-sourcepos="279:3-279:26"><td data-sourcepos="279:4-279:6">y</td><td data-sourcepos="279:8-279:25">WireProxy 开关</td></tr><tr data-sourcepos="280:3-280:57"><td data-sourcepos="280:4-280:6">k</td><td data-sourcepos="280:8-280:56">切换 wireguard 内核 / wireguard-go-reserved</td></tr><tr data-sourcepos="281:3-281:76"><td data-sourcepos="281:4-281:6">g</td><td data-sourcepos="281:8-281:75">切换 warp 全局 / 非全局 或首次以 非全局 模式安装</td></tr><tr data-sourcepos="282:3-282:62"><td data-sourcepos="282:4-282:6">s</td><td data-sourcepos="282:8-282:61">s 4/6/d，切换优先级 warp IPv4 / IPv6 / 默认</td></tr><tr data-sourcepos="283:3-283:35"><td data-sourcepos="283:4-283:19">其他或空值</td><td data-sourcepos="283:21-283:34">菜单界面</td></tr></tbody></table>

举例：想为 IPv4 的甲骨文添加 Warp 双栈，首次运行

```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh d
```

刷日本 Netflix 运行

```
warp i jp
```

[](#%E9%80%9A%E8%BF%87-warp-%E8%A7%A3%E9%94%81-chatgpt-%E7%9A%84%E6%96%B9%E6%B3%95)通过 warp 解锁 chatGPT 的方法
---------------------------------------------------------------------------------------------------------

方法原创，转引用请标明本项目出处。  
适合范围: 除大陆、香港和美国 LA 外的所有 VPS，因为这些地方没有 wgcf 的 warp 服务  
思路是使用已经注册的 warp 做链式代理的设置，此解决方法是最轻便的，用户只要有 xray 即可。具体做法是修改 xray 配置文件的 outbound 和 routing，模板如下

```
{
    "outbounds":[
        {
            "protocol":"freedom",
            "tag": "direct"
        },
        {
            "protocol":"wireguard",
            "settings":{
                "secretKey":"YFYOAdbw1bKTHlNNi+aEjBM3BO7unuFC5rOkMRAz9XY=", // 粘贴你的 "private_key" 值
                "address":[
                    "172.16.0.2/32",
                    "2606:4700:110:8a36:df92:102a:9602:fa18/128"
                ],
                "peers":[
                    {
                        "publicKey":"bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
                        "allowedIPs":[
                            "0.0.0.0/0",
                            "::/0"
                        ],
                        "endpoint":"engage.cloudflareclient.com:2408" // 或填写 162.159.193.10:2408 或 [2606:4700:d0::a29f:c001]:2408
                    }
                ],
                "reserved":[78, 135, 76], // 粘贴你的 "reserved" 值
                "mtu":1280
            },
            "tag":"wireguard"
        },
        {
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv4"
            },
            "proxySettings":{
                "tag":"wireguard"
            },
            "tag":"warp-IPv4"
        },
        {
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv6"
            },
            "proxySettings":{
                "tag":"wireguard"
            },
            "tag":"warp-IPv6"
        }
    ],
    "routing":{
        "domainStrategy":"AsIs",
        "rules":[
            {
                "type":"field",
                "domain":[
                    "geosite:openai",
                    "ip.gs"
                ],
                "outboundTag":"warp-IPv4"
            },
            {
                "type":"field",
                "domain":[
                    "geosite:netflix",
                    "p3terx.com"
                ],
                "outboundTag":"warp-IPv6"
            }
        ]
    }
}
```

[](#%E5%88%B7-netflix-%E8%A7%A3%E9%94%81-warp-ip-%E7%9A%84%E6%96%B9%E6%B3%95)刷 Netflix 解锁 WARP IP 的方法
-----------------------------------------------------------------------------------------------------

也可以用另一个通过 WARP 解锁流媒体的一键脚本: [【刷 WARP IP】 - 为 WARP 解锁流媒体而生](https://github.com/fscarmen/unlock_warp)

以刷 香港 hk 为例

*   screen 多会话方式运行，会话任务名为 n

```
screen -USdm n warp i hk  ##创建名为 n 的会话
screen -Udr n  ##进入会话 n 看运行情况
## 按 Ctrl+a 再按 d 退出话 n，返回主界面
screen -ls  ##查看会话窗口列表
screen -SX n quit  ##关闭会议 n，结束运行
```

*   nohup & 后台运行方式，把结果输出到 log 文件

```
nohup warp i hk > logs 2>&1 &   ##放进后台运行
jobs -l | grep warp  ##看后台任务
cat logs  ##查看运行日志文件
kill -9 $(jobs -l | grep warp | awk '{print $2}')  ##结束进程
```

*   crobtab 计划任务

```
echo '@reboot root warp i hk' >>/etc/crobtab   ##在计划任务里加入一条新任务
sed -i '/warp i/d' /etc/crontab   ##删掉计划任务
kill -9 $(pgrep -f warp)   ##杀掉正在运行的进程
```

*   另外遇到问题仍然需要用户有一定的处理能力，如结束时没有网络，可以用 `warp o` 开关来获取，因此并没有写死在脚本里了。
    
*   如果长时间仍然未刷出解锁 IP，可以查查 CloudFlare 当地是否在维护调路由：[https://www.cloudflarestatus.com/](https://www.cloudflarestatus.com/)
    

[](#%E6%8C%87%E5%AE%9A%E7%BD%91%E7%AB%99%E5%88%86%E6%B5%81%E5%88%B0-socks5-%E7%9A%84-xray-%E9%85%8D%E7%BD%AE%E6%A8%A1%E6%9D%BF-%E9%80%82%E7%94%A8%E4%BA%8E-warp-client-proxy-%E5%92%8C-wireproxy)指定网站分流到 socks5 的 xray 配置模板 (适用于 WARP Client Proxy 和 WireProxy)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

本地 [socks5://127.0.0.1:40000](socks5://127.0.0.1:40000) 并安装 [mack-a 八合一脚本](https://github.com/mack-a/v2ray-agent) 为例。编辑 `/etc/v2ray-agent/xray/conf/10_ipv4_outbounds.json`

```
{
    "outbounds":[
        {
            "protocol":"freedom"
        },
        {
            "tag":"warp",
            "protocol":"socks",
            "settings":{
                "servers":[
                    {
                        "address":"127.0.0.1",
                        "port":40000 // 填写你的 socks5 端口
                    }
                ]
            }
        },
        {
            "tag":"WARP-socks5-v4",
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv4"
            },
            "proxySettings":{
                "tag":"warp"
            }
        },
        {
            "tag":"WARP-socks5-v6",
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv6"
            },
            "proxySettings":{
                "tag":"warp"
            }
        }
    ],
    "routing":{
        "rules":[
            {
                "type":"field",
                "domain":[
                    "geosite:openai",
                    "ip.gs"
                ],
                "outboundTag":"WARP-socks5-v4"
            },
            {
                "type":"field",
                "domain":[
                    "geosite:google",
                    "geosite:netflix",
                    "p3terx.com"
                ],
                "outboundTag":"WARP-socks5-v6"
            }
        ]
    }
}
```

[](#%E6%8C%87%E5%AE%9A%E7%BD%91%E7%AB%99%E5%88%86%E6%B5%81%E5%88%B0-interface-%E7%9A%84-xray-%E9%85%8D%E7%BD%AE%E6%A8%A1%E6%9D%BF%E9%80%82%E7%94%A8%E4%BA%8E-warp-client-warp-%E5%92%8C-warp--warp-go-%E9%9D%9E%E5%85%A8%E5%B1%80)指定网站分流到 "interface" 的 xray 配置模板（适用于 WARP Client Warp 和 warp / warp-go 非全局）
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

```
{
    "outbounds":[
        {
            "protocol":"freedom"
        },
        {
            "tag":"WARP-interface-v4",
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv4"
            },
            "streamSettings":{
                "sockopt":{
                    "interface":"CloudflareWARP", // warp 非全局模式填 warp; Client 的 Proxy 模式填 CloudflareWARP; warp-go 填 WARP
                    "tcpFastOpen":true
                }
            }
        },
        {
            "tag":"WARP-interface-v6",
            "protocol":"freedom",
            "settings":{
                "domainStrategy":"UseIPv6"
            },
            "streamSettings":{
                "sockopt":{
                    "interface":"CloudflareWARP",
                    "tcpFastOpen":true
                }
            }
        }
    ],
    "routing":{
        "domainStrategy":"AsIs",
        "rules":[
            {
                "type":"field",
                "domain":[
                    "geosite:google",
                    "geosite:openai",
                    "ip.gs"
                ],
                "outboundTag":"WARP-interface-v4"
            },
            {
                "type":"field",
                "domain":[
                    "geosite:netflix",
                    "p3terx.com"
                ],
                "outboundTag":"WARP-interface-v6"
            }
        ]
    }
}
```

[](#warp-go-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)warp-go 运行脚本
-------------------------------------------------------------

首次运行

```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/warp-go.sh && bash warp-go.sh [option] [lisence]
```

再次运行

```
warp-go [option] [lisence]
```

<table dir="auto" data-sourcepos="545:3-557:35"><thead><tr data-sourcepos="545:3-545:51"><th data-sourcepos="545:4-545:29">[option] 变量 1 变量 2</th><th data-sourcepos="545:31-545:50">具体动作说明</th></tr></thead><tbody><tr data-sourcepos="547:3-547:16"><td data-sourcepos="547:4-547:6">h</td><td data-sourcepos="547:8-547:15">帮助</td></tr><tr data-sourcepos="548:3-548:44"><td data-sourcepos="548:4-548:6">4</td><td data-sourcepos="548:8-548:43">原无论任何状态 -&gt; WARP IPv4</td></tr><tr data-sourcepos="549:3-549:128"><td data-sourcepos="549:4-549:19">4 lisence name</td><td data-sourcepos="549:21-549:127">把 WARP+ Lisence 和设备名添加进去，如 <code data-sourcepos="549:74-549:123">bash wire-go 4 N5670ljg-sS9jD334-6o6g4M9F Goodluck</code></td></tr><tr data-sourcepos="550:3-550:44"><td data-sourcepos="550:4-550:6">6</td><td data-sourcepos="550:8-550:43">原无论任何状态 -&gt; WARP IPv6</td></tr><tr data-sourcepos="551:3-551:46"><td data-sourcepos="551:4-551:6">d</td><td data-sourcepos="551:8-551:45">原无论任何状态 -&gt; WARP 双栈</td></tr><tr data-sourcepos="552:3-552:75"><td data-sourcepos="552:4-552:6">o</td><td data-sourcepos="552:8-552:74">warp-go 开关，脚本主动判断当前状态，自动开或关</td></tr><tr data-sourcepos="553:3-553:24"><td data-sourcepos="553:4-553:6">u</td><td data-sourcepos="553:8-553:23">卸载 warp-go</td></tr><tr data-sourcepos="554:3-554:40"><td data-sourcepos="554:4-554:6">a</td><td data-sourcepos="554:8-554:39">免费 WARP 账户升级 WARP+</td></tr><tr data-sourcepos="555:3-555:145"><td data-sourcepos="555:4-555:18">a lisence name</td><td data-sourcepos="555:20-555:144">在上面基础上把 WARP+ Lisence 和设备名添加进去，如 <code data-sourcepos="555:91-555:140">bash menu.sh a N5670ljg-sS9jD334-6o6g4M9F Goodluck</code></td></tr><tr data-sourcepos="556:3-556:37"><td data-sourcepos="556:4-556:6">v</td><td data-sourcepos="556:8-556:36">同步脚本至最新版本</td></tr><tr data-sourcepos="557:3-557:35"><td data-sourcepos="557:4-557:19">其他或空值</td><td data-sourcepos="557:21-557:34">菜单界面</td></tr></tbody></table>

[](#cloudflare-api-%E8%BF%90%E8%A1%8C%E8%84%9A%E6%9C%AC)Cloudflare api 运行脚本
---------------------------------------------------------------------------

```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/api.sh && bash api.sh [option]
```

<table dir="auto" data-sourcepos="564:3-577:51"><thead><tr data-sourcepos="564:3-564:43"><th data-sourcepos="564:4-564:21">[option] 变量</th><th data-sourcepos="564:23-564:42">具体动作说明</th></tr></thead><tbody><tr data-sourcepos="566:3-566:28"><td data-sourcepos="566:4-566:18">-h/--help</td><td data-sourcepos="566:20-566:27">帮助</td></tr><tr data-sourcepos="567:3-567:150"><td data-sourcepos="567:4-567:18">-f/--file</td><td data-sourcepos="567:20-567:149">保存账户注册信息的文件，支持官方 api，client，wgcf 和 warp-go ，不填则手动输入 device id 和 api token</td></tr><tr data-sourcepos="568:3-568:34"><td data-sourcepos="568:4-568:18">-r/--register</td><td data-sourcepos="568:20-568:33">注册账户</td></tr><tr data-sourcepos="569:3-569:132"><td data-sourcepos="569:4-569:18">-t/--token</td><td data-sourcepos="569:20-569:131">-r 注册时，使用 team token 注册，快速获取: <a rel="nofollow noreferrer noopener" href="https://web--public--warp-team-api--coia-mfs4.code.run" target="_blank">https://web--public--warp-team-api--coia-mfs4.code.run</a></td></tr><tr data-sourcepos="570:3-570:70"><td data-sourcepos="570:4-570:18">-d/--device</td><td data-sourcepos="570:20-570:69">获取账户注册信息，包括 plus 流量等</td></tr><tr data-sourcepos="571:3-571:39"><td data-sourcepos="571:4-571:18">-a/--app</td><td data-sourcepos="571:20-571:38">获取 app 信息</td></tr><tr data-sourcepos="572:3-572:64"><td data-sourcepos="572:4-572:18">-b/--bind</td><td data-sourcepos="572:20-572:63">获取绑定设备信息，包括子设备</td></tr><tr data-sourcepos="573:3-573:40"><td data-sourcepos="573:4-573:18">-n/--name</td><td data-sourcepos="573:20-573:39">修改设备名称</td></tr><tr data-sourcepos="574:3-574:36"><td data-sourcepos="574:4-574:18">-l/--license</td><td data-sourcepos="574:20-574:35">修改 license</td></tr><tr data-sourcepos="575:3-575:34"><td data-sourcepos="575:4-575:18">-u/--unbind</td><td data-sourcepos="575:20-575:33">解绑设备</td></tr><tr data-sourcepos="576:3-576:34"><td data-sourcepos="576:4-576:18">-c/--cancle</td><td data-sourcepos="576:20-576:33">注销账户</td></tr><tr data-sourcepos="577:3-577:51"><td data-sourcepos="577:4-577:18">-i/--id</td><td data-sourcepos="577:20-577:50">显示 cliend id 与 reserved</td></tr></tbody></table>

[](#warp-license-%E5%8F%8A-id-%E8%8E%B7%E5%8F%96)WARP+ License 及 ID 获取
----------------------------------------------------------------------

以下是使用 WARP 和 Team 后 Argo 2.0 的官方介绍:[Argo 2.0: Smart Routing Learns New Tricks](https://blog.cloudflare.com/argo-v2/)

引用 Luminous 大神原话：实际测试 WARP + 在访问非 CF 的网站速度上和免费版没有差异，只有在访问 CloudFlare 的站点时付费版会通过 Argo 类似的技术通过与目标较近的数据中心前往源站，而免费版是仅限于连接地前往源站，仅此而已。

![](https://user-content.gitlab-static.net/3e7aa3f1655734a967363003aca08b4d491fd993/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3133363037303332332d34376632363030612d313365342d346562302d613634642d6437656238303563323865322e706e67)

[](#warp-%E7%BD%91%E7%BB%9C%E6%8E%A5%E5%8F%A3%E6%95%B0%E6%8D%AE%E4%B8%B4%E6%97%B6%E6%B0%B8%E4%B9%85%E5%85%B3%E9%97%AD%E5%92%8C%E5%BC%80%E5%90%AF)WARP 网络接口数据，临时、永久关闭和开启
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

WireGuard 网络接口数据，查看 `wg`

临时关闭和开启 WARP（reboot 重启后恢复开启） `warp o` 官方原始指令 `wg-quick down wgcf` ，恢复启动 `wg-quick up wgcf`

禁止开机启动 `systemctl disable --now wg-quick@wgcf`, 恢复开机启动 `systemctl enable --now wg-quick@wgcf`

[](#warp-teams-%E8%8E%B7%E5%8F%96%E5%B9%B6%E7%94%A8%E4%BA%8E-linux-%E7%9A%84%E6%96%B9%E6%B3%95)WARP Teams 获取并用于 Linux 的方法
-------------------------------------------------------------------------------------------------------------------------

*   通过 Coia 的网站，填入 teams 的组织名、邮箱和验证码获取 token: `https://web--public--warp-team-api--coia-mfs4.code.run/`
    
*   在 vps 里运行以下指令获取 teams 配置的全部信息，保存在文件 `warp-account.conf`
    

```
bash <(wget -qO- https://gitlab.com/fscarmen/warp/-/raw/main/api.sh) -r -t <TOKEN>
```

[](#warp%E5%8E%9F%E7%90%86)WARP 原理
----------------------------------

WARP 是 CloudFlare 提供的一项基于 WireGuard 的网络流量安全及加速服务，能够让你通过连接到 CloudFlare 的边缘节点实现隐私保护及链路优化。

其连接入口为双栈（IPv4/IPv6 均可），且连接后能够获取到由 CF 提供基于 NAT 的 IPv4 和 IPv6 地址，因此我们的单栈服务器可以尝试连接到 WARP 来获取额外的网络连通性支持。这样我们就可以让仅具有 IPv6 的服务器访问 IPv4，也能让仅具有 IPv4 的服务器获得 IPv6 的访问能力。

*   为仅 IPv6 服务器添加 IPv4

原理如图，IPv4 的流量均被 WARP 网卡接管，实现了让 IPv4 的流量通过 WARP 访问外部网络。 ![](https://user-content.gitlab-static.net/ebc3fe70379270422dbba8408c69bae5c1de253e/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3133353733353430342d31333839643032322d653563352d346562382d393635352d6639663036356533633932652e706e67)

*   为仅 IPv4 服务器添加 IPv6

原理如图，IPv6 的流量均被 WARP 网卡接管，实现了让 IPv6 的流量通过 WARP 访问外部网络。 ![](https://user-content.gitlab-static.net/86794b74adb908ae1692354cbf8323589954e2e1/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3133353733353431342d30313332316230622d383837652d343364362d616436382d6137346462323063666538342e706e67)

*   双栈服务器置换网络

有时我们的服务器本身就是双栈的，但是由于种种原因我们可能并不想使用其中的某一种网络，这时也可以通过 WARP 接管其中的一部分网络连接隐藏自己的 IP 地址。至于这样做的目的，最大的意义是减少一些滥用严重机房出现验证码的概率；同时部分内容提供商将 WARP 的落地 IP 视为真实用户的原生 IP 对待，能够解除一些基于 IP 识别的封锁。 ![](https://user-content.gitlab-static.net/6938d7a9e30b03aa50b562387880154de3582df4/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f36323730333334332f3133353733353431392d35303830356564362d323065612d343434302d393362342d3562636336663261636139622e706e67)

*   网络性能方面：内核集成＞内核模块＞wireguard-go

Linux 5.6 及以上内核则已经集成了 WireGuard ，可以用 `hostnamectl`或`uname -r`查看版本。

甲骨文是 KVM 完整虚拟化的 VPS 主机，而官方系统由于版本较低，在不更换内核的前提下选择 "内核模块" 方案。如已升级内核在 5.6 及以上，将会自动选择 “内核集成” 方案。

EUserv 是 LXC 非完整虚拟化 VPS 主机，共享宿主机内核，不能更换内核，只能选择 "wireguard-go" 方案。

[](#%E9%B8%A3%E8%B0%A2%E4%B8%8B%E5%88%97%E4%BD%9C%E8%80%85%E7%9A%84%E6%96%87%E7%AB%A0%E5%92%8C%E9%A1%B9%E7%9B%AE)鸣谢下列作者的文章和项目
-----------------------------------------------------------------------------------------------------------------------------

互联网永远不会忘记，但人们会。

技术文章或相关项目（排名不分先后）:

*   P3terx: [https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html](https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html)
*   P3terx: [https://github.com/P3TERX/warp.sh/blob/main/warp.sh](https://github.com/P3TERX/warp.sh/blob/main/warp.sh)
*   猫大: [https://github.com/Oreomeow](https://github.com/Oreomeow)
*   Luminous: [https://luotianyi.vc/5252.html](https://luotianyi.vc/5252.html)
*   Hiram: [https://hiram.wang/cloudflare-wrap-vps](https://hiram.wang/cloudflare-wrap-vps)
*   Cloudflare: [https://pkg.cloudflareclient.com/](https://pkg.cloudflareclient.com/) [https://blog.cloudflare.com/announcing-warp-for-linux-and-proxy-mode/](https://blog.cloudflare.com/announcing-warp-for-linux-and-proxy-mode/)  
    [https://blog.cloudflare.com/argo-v2/](https://blog.cloudflare.com/argo-v2/)
*   WireGuard: [https://lists.zx2c4.com/pipermail/wireguard/2017-December/002201.html](https://lists.zx2c4.com/pipermail/wireguard/2017-December/002201.html)
*   Parker C. Stephens: [https://parkercs.tech/cloudflare-for-teams-wireguard-config/](https://parkercs.tech/cloudflare-for-teams-wireguard-config/)
*   Anemone: [https://cutenico.best/posts/blogs/cloudflare-warp-fixed-youtube-location/](https://cutenico.best/posts/blogs/cloudflare-warp-fixed-youtube-location/)  
    [https://github.com/acacia233/Project-WARP-Unlock](https://github.com/acacia233/Project-WARP-Unlock)
*   wangying202: [https://blog.csdn.net/wangying202/article/details/113178159](https://blog.csdn.net/wangying202/article/details/113178159)
*   LUBAN: [https://github.com/HXHGTS/Cloudflare_WARP_Connect](https://github.com/HXHGTS/Cloudflare_WARP_Connect)
*   valetzx: [https://gitlab.com/valetzx/pubfile](https://gitlab.com/valetzx/pubfile)
*   badafans cf api: [https://github.com/badafans/warp-reg](https://github.com/badafans/warp-reg)
*   chika0801: [https://github.com/chika0801/Xray-examples/](https://github.com/chika0801/Xray-examples/)
*   所有的热心网友们

服务提供（排名不分先后）:

*   CloudFlare Warp(+): [https://1.1.1.1/](https://1.1.1.1/)
*   WGCF 项目原作者: [https://github.com/ViRb3/wgcf/](https://github.com/ViRb3/wgcf/)
*   Coia 和 warp-go 团队: [https://gitlab.com/ProjectWARP/warp-go](https://gitlab.com/ProjectWARP/warp-go)
*   warp-go api wiki: [https://docs.zeroteam.top/apis/warp](https://docs.zeroteam.top/apis/warp)
*   WireGuard-GO 官方: [https://git.zx2c4.com/wireguard-go/](https://git.zx2c4.com/wireguard-go/)
*   ylx2016 的成熟作品: [https://github.com/ylx2016/Linux-NetSpeed](https://github.com/ylx2016/Linux-NetSpeed)
*   ALIILAPRO 的成熟作品: [https://github.com/ALIILAPRO/warp-plus-cloudflare](https://github.com/ALIILAPRO/warp-plus-cloudflare)
*   mixool 的成熟作品: [https://github.com/azples/across/tree/main/wireguard](https://github.com/azples/across/tree/main/wireguard)
*   luoxue-bot 的成熟作品:[https://github.com/luoxue-bot/warp_auto_change_ip](https://github.com/luoxue-bot/warp_auto_change_ip)
*   lmc999 的成熟作品: [https://github.com/lmc999/RegionRestrictionCheck](https://github.com/lmc999/RegionRestrictionCheck)
*   WireProxy 作者: [https://github.com/pufferffish/wireproxy](https://github.com/pufferffish/wireproxy)
*   获取公网 IP 及归属地查询: [https://ifconfig.co/](https://ifconfig.co/)  
    [https://ip.gs/](https://ip.gs/)  
    [https://ip.sb/](https://ip.sb/)
*   统计 PV 网:[https://hits.seeyoufarm.com/](https://hits.seeyoufarm.com/)
*   Coia 的网页版提出 Teams Token: [https://web--public--warp-team-api--coia-mfs4.code.run](https://web--public--warp-team-api--coia-mfs4.code.run)

CloudFlare WARP 全球站点和服务状态:

*   Operational = 正常。Re-routed = 检修状态: [https://www.cloudflarestatus.com/](https://www.cloudflarestatus.com/)