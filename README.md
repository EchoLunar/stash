# VMUTEX 分流配置

`VMUTEX.stoverride` 是 Stash 的规则、策略组与 DNS 覆写；`VMUTEX.yaml` 是 Mihomo 版本。公开文件不包含私人节点订阅地址。`VMUTEX-Local.yaml` 仅存本地，已在 `.gitignore` 中排除；请勿用强制添加方式上传。

## 分流策略

Stash 与 Mihomo 的分组均为 `PROXY`、`Oracle`、`AI`、`Crypto` 和 `Apple`。`Oracle` 可选直连或跟随 `PROXY`；Stash 的 `Crypto` 与 `Apple` 和 `Oracle` 一样，仅可选 `DIRECT` 或 `PROXY`，默认直连；Mihomo 的 `Crypto` 与 `Apple` 还可选具体节点。请导入自己的节点后检查各组选择。Stash 的 `PROXY` 与 `AI` 仅收集全部节点和代理集，不提供 `REJECT` 选项；AI 直接选择具体节点或代理集。

规则先匹配 Oracle、Crypto、局域网和私有名称，再匹配 AI 与 Apple 专属分组。其余流量仅通过 `GEOSITE,cn,DIRECT` 和 `GEOIP,CN,DIRECT,no-resolve` 判断国内直连，最后由 `MATCH,PROXY` 兜底。已移除各服务单独指向 `PROXY` 的规则、微软与 B 站单独直连规则，以及比亚迪临时规则。服务的国内资源可能因此直连；未被国内域名集收录且没有真实目标 IP 的连接仍会走代理。

两份配置保留相同规则顺序，仅局域网 IP 分类分别使用 Stash 的 `LAN` 和 Mihomo 的 `private`。远程规则提供器只保留 `Crypto` 与 `oracle`；Oracle 的自有 IP 与域名由 `Rules/oracle.list` 管理。

## DNS

Stash 关闭 `follow-rule`；Mihomo 关闭 `respect-rules`。两者使用 Fake IP 保留普通连接的域名，只让私有名称和少量网络连通性检测域名返回真实 IP。需要解析公网真实 IP 时使用单一阿里 DoH；私有名称使用系统 DNS；代理节点的服务器域名使用独立的阿里与 DNSPod DoH。`GEOIP,CN,DIRECT,no-resolve` 只判断已有目标 IP，不为了匹配规则而提前解析域名。

这些配置不能保证绝对零 DNS 泄漏或零 CDN 漂移：系统启动解析、要求真实 IP 的连接以及应用内置 DNS 都可能绕开普通 Fake IP 流程。更新后应在客户端检查 DNS 诊断和实际连接出口。

## 使用与更新

- Stash：先加载自己的节点订阅，再启用 `VMUTEX.stoverride`。覆写会整体替换 DNS、策略组、规则与规则提供器；节点由基础配置提供。[Stash 覆写文档](https://stash.wiki/configuration/override)
- Mihomo：加载 `VMUTEX.yaml` 并自行加入节点或代理提供器；远程规则集每 24 小时检查更新。

更新后重新加载配置、清理 DNS 缓存并关闭旧连接，再确认 Oracle、Crypto、AI、Apple、国内域名和最终兜底规则的实际命中情况。
