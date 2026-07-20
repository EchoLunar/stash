# Stash / Clash Verge Rev 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

## Stash

使用 `stash.stoverride` 作为 Stash 覆写。

## Clash Verge Rev

使用 `clash-verge.yaml` 作为订阅的扩展覆写配置：

1. 在 Clash Verge Rev 中导入 3x-ui 订阅。
2. 右键该订阅，进入扩展配置编辑器。
3. 将 `clash-verge.yaml` 的完整内容粘贴并保存。
4. 更新订阅并启用该配置。

该覆写不定义 `proxies`，因此不会把订阅下发的真实节点清掉；`Proxy` 和 `Polymarket` 使用 `include-all: true` 收纳这些节点。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
