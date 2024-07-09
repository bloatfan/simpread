> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iZJ8FSJFiFAWdVxKuY_Cqw)

目前 `DockerHub` 仓库镜像服务基本都下线了， `Docker` 可以说是开发人员的必备工具，这个影响还是很大的。对老胡来说，不论是从个人还是技术团队，如何配置 `DockerHub` 镜像加速是个急需解决的问题。

经过调研探索，老胡总结了以下方案：

*   直接用第三方搭建好的
    
*   自建
    

*   基于 Cloudflare Workers 镜像代理
    
*   基于 Github Action 转存同步
    
*   [推荐] 购买服务器
    
*   利用免费平台
    

> 有更好的方案欢迎留言分享

第三方
---

<table><thead><tr><th>加速地址</th><th>说明</th></tr></thead><tbody><tr><td>https://docker.m.daocloud.io</td><td>DaoCloud 驱动</td></tr><tr><td>https://dockerpull.com</td><td>Docker Proxy &nbsp;驱动</td></tr><tr><td>https://atomhub.openatom.cn</td><td>AtomHub 提供，仅有基础镜像</td></tr><tr><td>https://docker.1panel.live</td><td>1panel &nbsp;驱动</td></tr><tr><td>https://dockerhub.jobcher.com</td><td>打工人日报驱动</td></tr><tr><td>https://hub.rat.dev</td><td>耗子面板驱动</td></tr><tr><td>https://docker.registry.cyou</td><td>bestcfipas 驱动</td></tr><tr><td>https://docker.awsl9527.cn</td><td>zeruns 驱动</td></tr><tr><td>https://do.nark.eu.org/</td><td>/</td></tr><tr><td>https://docker.ckyl.me</td><td>/</td></tr><tr><td>https://hub.uuuadc.top</td><td>/</td></tr><tr><td>https://docker.chenby.cn</td><td>/</td></tr><tr><td>https://docker.ckyl.me</td><td>/</td></tr></tbody></table>

以上排名不分先后，为了后续维护方便，老胡弄了个页面维护免费可用的 Docker 镜像代理域名列表👉 https://www.fre321.com/p/docker_proxy_list。

自建
--

### 基于 VPS

购买一台海外 VPS 和域名，通过开源项目自建，老胡认为这是最**稳妥**的方案，这块的开源项目有很多，老胡都试了下，最终选用的是 dqzboy/Docker-Proxy，安装也很简单：

```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/dqzboy/Docker-Proxy/main/install/DockerProxy_Install.sh)"


```

执行跟着配置走就搞定了。

### 基于平台

基于 CF 搭建的话推荐：CF-Workers-docker.io，部署方式支持 `Workers` 和 `Pages`，教程在 README 中，讲得也很清楚。

使用 `Github Action` 推荐 tech-shrimp/docker_image_pusher

*   支持 DockerHub, gcr.io, k8s.io, ghcr.io 等任意仓库
    
*   支持最大 40GB 的大型镜像
    
*   使用阿里云的官方线路，速度快
    

使用
--

不论是用第三方还是自建，使用基本都是类似的：

```
# 通过域名使用
# 说明：library是一个特殊的命名空间，它代表的是官方镜像。如果是某个用户的镜像就把library替换为镜像的用户名
docker pull {domain}/library/alpine:latest

# 设置 registry mirror，推荐
tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://xxx.xxx"]
}
EOF
# 在里面填上代理域名即可，可以多个
systemctl daemon-reload
systemctl restart docker


```