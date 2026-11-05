# VMUTEX 分流配置

`VMUTEX.stoverride` 是 Stash 的规则、策略组与 DNS 覆写；`VMUTEX.yaml` 是 Mihomo 版本；`vmutex-loon.lcf` 是 Loon 版本。公开文件不包含私人节点订阅地址。`VMUTEX-Local.yaml` 仅存本地，已在 `.gitignore` 中排除；请勿用强制添加方式上传。

## 分流策略

三个客户端的核心出口均为总出口、`Oracle`、`AI`、`Crypto`。Stash 与 Mihomo 的总出口叫 `PROXY`，Loon 使用 `Proxy`，避免与 Loon 内置的 `PROXY` 策略名冲突。YouTube 和 Telegram 不再单设策略组，相关域名与 IP 规则直接跟随总出口。`Oracle` 默认直连，也可跟随总出口；`Crypto` 可选直连、代理或单独节点。请导入自己的节点后主动检查各组选择；没有选中可用节点时，`REJECT` 用于避免意外直连。

Stash 与 Mihomo 使用相同顺序的主要业务规则，并共用 `Crypto`、`BiliIntl`、`NetflixIP`、`TelegramIP` 等远程列表。Oracle 的自有 IP 与域名由 `Rules/oracle.list` 管理，Loon 也直接订阅这份规则。Loon 不直接读取 Stash 的 GeoSite 数据，因此使用 Loon 格式的远程列表近似对应；来源和命中范围可能不同，不能保证逐域名完全一致。Loon 的中国大陆直连订阅使用可莉的 `REGION_SPLITTER.lsr`，并将可莉的国内域名合集 `ChinaMaxNoIP.lsr` 放在远程规则倒数第二项、`Global` 之前；仍未命中的流量由本地 `FINAL,Proxy` 兜底。其命中范围与 Stash 的 `GEOSITE,cn`、`GEOIP,CN` 可能不同。B 站国际版的 MetaCubeX 列表已转为 Loon 本地规则，避免 Loon 的普通 B 站列表将国际版域名直连。

## DNS

Stash 关闭 `follow-rule`；Mihomo 关闭 `respect-rules`。两者使用 Fake IP 保留普通连接的域名，只让私有名称和少量网络连通性检测域名返回真实 IP。需要解析公网真实 IP 时使用单一阿里 DoH；私有名称使用系统 DNS；代理节点的服务器域名使用独立的阿里与 DNSPod DoH。`GEOIP,CN,DIRECT,no-resolve` 只判断已有目标 IP，不为了匹配规则而提前解析域名。

Loon 的全局公网 DNS 也改为单一阿里 DoH，`[Host]` 将常见私有名称交给系统 DNS。Loon 默认在加密 DNS 失败时回退普通 DNS；如需阻止这个回退，须在 App 的“DNS 服务器”页面关闭。Loon 的节点服务器 DNS 可按订阅单独设置，它无法直接复刻 Stash 的 `proxy-server-nameserver`。参见 [Loon DNS 文档](https://nsloon.app/en/docs/DNS/) 与 [订阅节点 DNS 说明](https://nsloon.app/en/docs/Node/subscription/)。

这些配置不能保证绝对零 DNS 泄漏或零 CDN 漂移：系统启动解析、要求真实 IP 的连接以及应用内置 DNS 都可能绕开普通 Fake IP 流程。更新后应在客户端检查 DNS 诊断和实际连接出口。

## 使用与更新

- Stash：先加载自己的节点订阅，再启用 `VMUTEX.stoverride`。覆写会整体替换 DNS、策略组、规则与规则提供器；节点由基础配置提供。[Stash 覆写文档](https://stash.wiki/configuration/override)
- Mihomo：加载 `VMUTEX.yaml` 并自行加入节点或代理提供器；远程规则集每 24 小时检查更新。
- Loon：导入 `vmutex-loon.lcf`，在本地配置自己的节点订阅；远程规则列表需要能从设备访问。公开配置不再内嵌 MitM 证书，若需要 HTTPS 解密，应在自己的设备上单独管理证书。

更新后重新加载配置、清理 DNS 缓存并关闭旧连接，再确认 Oracle、Crypto、AI、国内域名和最终兜底规则的实际命中情况。
