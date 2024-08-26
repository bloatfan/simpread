> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.paizishop.com](https://www.paizishop.com/yunwei/2023/04/05/35964.html)

> 宝塔的 NGINX 用的非常多，但是宝塔默认没有禁止别人通过 IP 访问，因此很容易被扫描器扫描到，加上 NGINX 的不是漏洞的漏洞，IP 访问 HTTPS 的话，会自动匹配第一个站点的 SSL 证书给 IP 使用，因此会造成......

2023 年 4 月 5 日 17:06:34 [运维](https://www.paizishop.com/yunwei/)1,408 阅读模式

宝塔的 NGINX 用的非常多，但是宝塔默认没有禁止别人通过 IP 访问，因此很容易被扫描器扫描到，加上 NGINX 的不是漏洞的漏洞，IP 访问 HTTPS 的话，会自动匹配第一个站点的 SSL 证书给 IP 使用，因此会造成 IP 泄露（证书带域名信息，网上有扫描全网 IP 并读取 SSL 证书中域名信息的方法）

因此为了安全起见，我们需要设置两个项目

1、给 IP 配置上一张带错误域名的证书，防止泄露你自己的域名

2、禁止直接访问 IP，将访问 IP 的请求，不管是 HTTP 还是 HTTPS 全部转错误页  返回状态码 444 ERR_EMPTY_RESPONSE

两步其实可以合并成下面步骤操作：

**1、首页在宝塔中创建一个默认站点, 这里域名随意填写，只要不是你的域名就行**

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**2、修改默认站点到新创建的这个域名**

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**3、给 IP 配置上一张带错误域名的证书（给默认站点设置证书）**，我们这里利用了 CLOUDFLARE 来作为错误证书颁发源，利用 CF 接入域名，可以颁发 15 年的仅 CF CDN 网络体系承认的证书的功能。

大家可以使用我们已经生成的这张证书，反正只要域名不是你真实的域名就行了，提供如下

**密钥：**

```
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgK0HE3hTJQDg6p/fj
nS92eSuRKZEZ5F4grT6tWFKNYVmhRANCAAQIP4WfZQx4/3/tIw0QDdt05DRKiIuO
pghp8GVQ94JcS5fmtZqX1yx0hBU4qZ0skIJr5D2M0BmhCBQ9Kulv2YDL
-----END PRIVATE KEY-----

```

**证书：**

```
-----BEGIN CERTIFICATE-----
MIIDITCCAsagAwIBAgIUTcEWLzynkLCFCoAC1iDH2vG3EkYwCgYIKoZIzj0EAwIw
gY8xCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T
YW4gRnJhbmNpc2NvMRkwFwYDVQQKExBDbG91ZEZsYXJlLCBJbmMuMTgwNgYDVQQL
Ey9DbG91ZEZsYXJlIE9yaWdpbiBTU0wgRUNDIENlcnRpZmljYXRlIEF1dGhvcml0
eTAeFw0xOTAxMTMxNDMxMDBaFw0zNDAxMDkxNDMxMDBaMGIxGTAXBgNVBAoTEENs
b3VkRmxhcmUsIEluYy4xHTAbBgNVBAsTFENsb3VkRmxhcmUgT3JpZ2luIENBMSYw
JAYDVQQDEx1DbG91ZEZsYXJlIE9yaWdpbiBDZXJ0aWZpY2F0ZTBZMBMGByqGSM49
AgEGCCqGSM49AwEHA0IABAg/hZ9lDHj/f+0jDRAN23TkNEqIi46mCGnwZVD3glxL
l+a1mpfXLHSEFTipnSyQgmvkPYzQGaEIFD0q6W/ZgMujggEqMIIBJjAOBgNVHQ8B
Af8EBAMCBaAwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMAwGA1UdEwEB
/wQCMAAwHQYDVR0OBBYEFCEZF6Eyem01XPbgwr6DXLZV1qsQMB8GA1UdIwQYMBaA
FIUwXTsqcNTt1ZJnB/3rObQaDjinMEQGCCsGAQUFBwEBBDgwNjA0BggrBgEFBQcw
AYYoaHR0cDovL29jc3AuY2xvdWRmbGFyZS5jb20vb3JpZ2luX2VjY19jYTAjBgNV
HREEHDAaggwqLmRuc3BvZC5jb22CCmRuc3BvZC5jb20wPAYDVR0fBDUwMzAxoC+g
LYYraHR0cDovL2NybC5jbG91ZGZsYXJlLmNvbS9vcmlnaW5fZWNjX2NhLmNybDAK
BggqhkjOPQQDAgNJADBGAiEAnrequCk/QZOOrcPH6C3Hgcy4SPNUy5rQtku/aYkj
qQoCIQCN6IyYNiXuwG+8jUgJrveiirBjiz2jXZSTEfVAyibjTg==
-----END CERTIFICATE-----

```

**4、上面保存后，点击配置文件修改配置**

将前面部分改成：

```
listen 80 default_server;
listen 443 ssl http2 default_server;
 
server_name default.com;
return 444;

```

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/QQ%E5%9B%BE%E7%89%8720230405170413.png)

https://www.paizishop.com/yunwei/2023/04/05/35964.html