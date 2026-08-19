# Stash 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

使用 `stash.stoverride` 作为 Stash 覆写。

Loon 使用 `loon.conf`。该文件不包含节点，只负责 DNS、策略组、规则和远程规则集；已经在 Loon 中添加的节点会通过 `All` 筛选器加入 `Proxy` 策略组。

Loon 远程配置地址：`https://raw.githubusercontent.com/EchoLunar/stash/main/loon.conf`

使用远程配置时，只需要在 Loon 中添加自己的节点，然后将上述地址作为远程配置导入即可。节点不会上传到 GitHub；远程配置只更新 DNS、策略组、规则和规则集。

`loon.conf` 已针对 macOS 新版 Loon 开启 `ipv6-vif = always`，并将 `2000::/3` 加入 `bypass-tun`，让全球单播 IPv6 走 macOS 原生路由。这样可能会使这部分 IPv6 流量绕过 Loon 的规则和代理。

该覆写不定义 `proxies`，因此不会把订阅下发的真实节点清掉；`Proxy` 和 `Polymarket` 使用 `include-all: true` 收纳这些节点。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
