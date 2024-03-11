> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/SoyTechnology/5647088)

> Solana 验证节点搭建（成功验证），搭建验证节点 (成功下载快照) 部署 Solana 验证节点由于项目需求，需要部署一台

RPC 节点，本教程使用的是阿里云的服务器 Ubuntu 20.04，32vCPU 128GiB 内存，2GiB SSD，这是最低配了，低于这个配置可能 RPC 启动不起来。

```
--public-rpc-address <HOST:PORT>
    RPC address for the validator to advertise publicly in gossip. Useful for validators running behind a load balancer or proxy [default: use --rpc-bind-address / --rpc-port]

    验证者在gossip中公开广告的 RPC 地址。 对于在负载均衡器或代理后面运行的验证器很有用 [默认：使用 --rpc-bind-address / --rpc-port]

--rpc-bind-address <HOST>
    IP address to bind the RPC port [default: 127.0.0.1 if --private-rpc is present, otherwise use --bind-address]
    用于绑定 RPC 端口的 IP 地址 [默认值：127.0.0.1 如果存在 --private-rpc，否则使用 --bind-address]

--rpc-port <PORT>
    Enable JSON RPC on this port, and the next port for the RPC websocket
    在此端口上启用 JSON RPC，以及 RPC websocket 的下一个端口

--accounts-db-skip-shrink
    通过跳过收缩，可以更快地启动验证器。此选项用于测试期间。

--incremental-snapshots
    通过设置此标志启用增量快照。启用后，--snapshot-interval-slots将设置增量快照间隔。要设置完整快照间隔，请使用 --full-snapshot-interval-slots。

--minimal-rpc-api
    仅公开向其他节点提供快照所需的 RPC 方法

--only-known-rpc
    仅使用已知验证器的 RPC 服务

--private-rpc
    不要发布 RPC 端口供他人使用

--restricted-repair-only-mode
    不要发布导致验证器在有限范围内运行的 Gossip、TPU、TVU 或维修服务端口减少其对集群其余部分的暴露的容量。 --no-voting 标志是隐式的，当这个标志已启用

--rpc-scan-and-fix-roots
    在启动时验证块存储根并修复任何差距

--skip-poh-verify
    在验证器启动时跳过分类帐验证

--bind-address <HOST>
    绑定验证器端口的 IP 地址 [默认：0.0.0.0]

--dynamic-port-range <MIN_PORT-MAX_PORT>
    用于动态分配端口的范围 [默认值：8000-10000]

--entrypoint <HOST:PORT>
    在这个 gossip 入口点与集群会合

--expected-genesis-hash <HASH>
    要求创世有这个哈希

-gossip-host <HOST>
    Gossip DNS 名称或 IP 地址供验证器在 gossip 中做广告 [默认：ask --entrypoint，或 127.0.0.1 未提供 --entrypoint 时]

--gossip-port <PORT>
    验证器的 gossip 端口号

--rpc-port <PORT>
    在此端口上启用 JSON RPC，以及 RPC websocket 的下一个端口

--vote-account <ADDRESS>
    验证人投票账户公钥。如果未指定投票将被禁用。授权选民为帐户必须是 --identity 密钥对或带有 --authorized-voter 参数

--wait-for-supermajority <SLOT>
    处理完账本后，下一个槽是 SLOT，等到绝大多数股权在开始PoH之前的八卦

--wal-recovery-mode <MODE>
    恢复分类帐数据库预写日志的模式。 [可能的值：tolerance_corrupted_tail_records，绝对一致性、point_in_time、skip_any_corrupted_record]
```