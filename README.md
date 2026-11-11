# VMUTEX 分流配置

`VMUTEX-Stash.yaml` 是 Stash 的完整远程主配置，包含规则、分组、DNS 和规则集；`VMUTEX.stoverride` 仅声明两个订阅参数并注入订阅地址；`VMUTEX.yaml` 是 Mihomo 版本。公开文件不包含私人节点订阅地址，两个订阅在 Stash 的覆写参数中填写。历史私人配置文件仍由 `.gitignore` 排除。

## 分流策略

Stash 与 Mihomo 的分组均为 `PROXY`、`Oracle`、`AI`、`Crypto` 和 `Apple`。Stash 的 `PROXY` 与 `AI` 通过 `use` 展开两个订阅中的节点；`Apple` 与 `Crypto` 在同一个分组显示 `DIRECT` 和两个订阅的节点，默认直连；`Oracle` 保持仅可选 `DIRECT` 或 `PROXY`。所有分组均不提供 `REJECT`。Mihomo 保持现有分组选项。启用后请在 `PROXY` 和 `AI` 中选择可用节点。

规则先匹配 Oracle、Crypto、局域网和私有名称，再匹配 AI 与 Apple 专属分组。其余流量按国内域名和已有目标 IP 判断国内直连，最后由 `MATCH,PROXY` 兜底。Stash 使用 MetaCubeX 的 `ChinaDomain` / `ChinaIP` 规则集；Mihomo 使用 `GEOSITE,cn,DIRECT` 和 `GEOIP,CN,DIRECT,no-resolve`。已移除各服务单独指向 `PROXY` 的规则、微软与 B 站单独直连规则，以及比亚迪临时规则。服务的国内资源可能因此直连；未被国内域名集收录且没有真实目标 IP 的连接仍会走代理。

两份配置保留相同业务顺序。Stash 的私有 IP、私有域名、AI、Apple、国内域名和国内 IP 全部使用 MetaCubeX 导出的规则集，不再依赖客户端的 GEOIP / GEOSITE 数据；私有 IP 使用 MetaCubeX 的 `private` 集合，其范围由上游维护。规则集每 24 小时检查更新，域名和 IP 集合分别使用 `domain` 和 `ipcidr` 类型，AI 与 Crypto 使用 `classical` 类型。Oracle 的自有 IP 与域名仍由 `Rules/oracle.list` 管理。Mihomo 保留现有 GEO 规则。

## DNS

Stash 关闭 `follow-rule`；Mihomo 关闭 `respect-rules`。两者使用 Fake IP 保留普通连接的域名，只让私有名称和少量网络连通性检测域名返回真实 IP。需要解析公网真实 IP 时使用单一阿里 DoH；私有名称使用系统 DNS；代理节点的服务器域名使用独立的阿里与 DNSPod DoH。Stash 的 `RULE-SET,ChinaIP,DIRECT,no-resolve` 与 Mihomo 的 `GEOIP,CN,DIRECT,no-resolve` 都只判断已有目标 IP，不为了匹配规则而提前解析域名。Stash 的 `fake-ip-filter` 和 `nameserver-policy` 共用 `rule-set:PrivateDomain`，需要 iOS/tvOS 3.6+ 或 macOS 4.3+。[Stash DNS 规则集支持说明](https://stash.wiki/features/dns-server)

这些配置不能保证绝对零 DNS 泄漏或零 CDN 漂移：系统启动解析、要求真实 IP 的连接以及应用内置 DNS 都可能绕开普通 Fake IP 流程。更新后应在客户端检查 DNS 诊断和实际连接出口。

## 使用与更新

Stash 参数化版本需要支持覆写参数的 **Beta iOS/tvOS 3.7+ 或 macOS 4.4+**。安装方式：

1. 在 Stash 中通过 URL 添加 [VMUTEX-Stash.yaml](https://raw.githubusercontent.com/EchoLunar/stash/refs/heads/main/VMUTEX-Stash.yaml)，作为远程主配置，保留其远程来源以便后续更新。
2. 通过 URL 安装 [VMUTEX.stoverride](https://raw.githubusercontent.com/EchoLunar/stash/refs/heads/main/VMUTEX.stoverride)，作为该主配置使用的订阅参数覆写。
3. 在覆写详情的「参数」中填写「第一个订阅地址」和「第二个订阅地址」，两个参数均为必填。订阅需要返回包含 `proxies` 的 Stash / Clash YAML。
4. 完成参数填写并启用覆写后再启动连接，等待两个代理集及规则集下载，在 `PROXY` 和 `AI` 中选择节点。Apple 和 Crypto 可直接选择 `DIRECT` 或订阅节点。主配置不含可用的订阅 URL，需与已填写参数的覆写配合使用。

更新与更换订阅：

- 规则、分组、DNS 有变化时，只需更新远程主配置 `VMUTEX-Stash.yaml`，无需编辑或重新填写订阅地址。
- 订阅地址始终从启用的参数化覆写注入，主配置更新不会覆盖这些本地参数。
- 覆写本身也可远程更新：更新同一个已安装覆写时，Stash 保留适用的参数值；不要删除后重新安装。后续维护保持 `subscription_a` / `subscription_b` 参数名不变。
- 参数值保存在本设备；官方说明分享或导出覆写只包含原始文件，不包含填写的参数值。换设备时需重新填写。
- 更换订阅只需修改覆写参数并保存应用，不需要改 GitHub 源文件。
- 两个代理集各自每 24 小时检查更新；合并是让分组引用两个代理集，不会生成第三份订阅，也不保证对重名节点去重。
- 参数覆写只递归合并 `SubscriptionA` / `SubscriptionB` 的 `url`，保留主配置的缓存路径和更新周期，不改分组、规则或 DNS。旧版 VMUTEX 规则覆写及其他订阅覆写应停用，避免它们重新覆盖主配置或订阅地址。

[Stash 参数与远程更新说明](https://stash.wiki/configuration/override#覆写参数)、[代理集说明](https://stash.wiki/proxy-protocols/proxy-providers)

- Mihomo：加载 `VMUTEX.yaml` 并自行加入节点或代理提供器；远程规则集每 24 小时检查更新。

更新后重新加载配置、清理 DNS 缓存并关闭旧连接，再确认 Oracle、Crypto、AI、Apple、国内域名和最终兜底规则的实际命中情况。
