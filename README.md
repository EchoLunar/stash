# Stash 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

使用 `stash.stoverride` 作为 Stash 覆写。

该覆写不定义 `proxies`，因此不会把订阅下发的真实节点清掉；`Proxy` 和 `Polymarket` 使用 `include-all: true` 收纳这些节点。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
