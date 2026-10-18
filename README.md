# VMUTEX 分流配置

`VMUTEX.yaml` 仅面向 Mihomo 内核；`VMUTEX.stoverride` 是按相同业务分组策略单独适配的 Stash 覆写，包含分组、连接规则、DNS 和规则提供器，不包含节点凭据。

## Mihomo 使用

`VMUTEX.yaml` 保留原生 `GEOSITE,telegram`、`GEOIP,telegram`、`GEOSITE,biliintl` 和 `GEOIP,netflix`，并让域名 DNS policy 使用对应 geosite 分类。Telegram 的域名规则与 IP 规则分别覆盖域名连接和直接 IP 连接。

这些扩展分类存在于 MetaCubeX geo 数据中；应使用包含这些分类的完整数据。此前仅因 Stash 兼容而添加的 `TelegramIP`、`NetflixIP`、`BiliIntl` 提供器已从 Mihomo 配置移除。参考：[MetaCubeX 数据说明](https://github.com/MetaCubeX/meta-rules-dat/blob/master/README.md)。

## Stash 使用

1. 使用 Stash iOS/tvOS 3.6+ 或 macOS 4.3+；这里使用了 `rule-set:` DNS policy、独立节点 DNS 和 inline 规则集。
2. 在 Stash 中加载自己的节点订阅，再导入并启用 `VMUTEX.stoverride`。
3. 为 `PROXY`、`AI`、`YouTube`、`Telegram`、`Binance`、`Bybit` 分别选择节点。`REJECT` 是显式拒绝选项，用来避免 Stash 空组按直连处理。
4. 这份覆写整体替换 DNS、策略组、规则和规则提供器。与仓库旧的 `stash.stoverride` 二选一，避免后应用的覆写覆盖本文件。
5. 重载配置、清理 DNS 缓存并关闭旧连接后，检查新连接的规则和出口。浏览器自带的加密 DNS 会绕开这里按查询域名选择上游的逻辑，应使用系统 DNS。

官方说明：[覆写合并](https://stash.wiki/configuration/override)、[DNS 配置](https://stash.wiki/features/dns-server)、[规则集合](https://stash.wiki/rules/rule-set)、[策略组](https://stash.wiki/proxy-protocols/proxy-groups)。

## 连接与 DNS 出口

两版均让固定直连例外优先，业务专用分类排在境外、国内和最终兜底分类之前。每份配置内部，域名连接规则与 DNS policy 共用自己的规则集和 geosite 分类；平台适配使用的集合名称和内容可能不同。

| 分类 | 业务连接 | Stash DNS 上游及出口 |
| --- | --- | --- |
| vmutex 中的域名、私有域名、Apple、Microsoft、Bilibili、国内域名 | DIRECT | system |
| Binance，包括原有 binance 关键词兜底 | Binance | Cloudflare `cloudflare-dns.com` → Binance |
| Bybit | Bybit | Google `dns.google` → Bybit |
| 境外 AI | AI | Cloudflare `1.0.0.1` + Google `8.8.4.4` → AI |
| YouTube | YouTube | Quad9 无过滤 `dns10.quad9.net` → YouTube |
| Telegram | Telegram | AdGuard 无过滤 `unfiltered.adguard-dns.com` → Telegram |
| 其他境外、Netflix、流媒体、游戏、GitHub、Google FCM、未命中域名 | PROXY | Cloudflare `1.1.1.1` + Google `8.8.8.8` → PROXY |

Stash 版开启 `follow-rule`，并在业务规则前为 DNS 上游指定出口。DNS 上游使用不同的地址，才能按各组分别路由；不能把同一个上游地址重复配置后，只靠查询域名让它走不同策略组。这里所有公共上游均使用 DoH。

Mihomo 版继续为每组使用 Cloudflare + Google，并用 `#组名` 绑定出口。因此两版的域名分流和出口分组对应，公共 DNS 服务商及并发上游数量不完全相同。Quad9 和 AdGuard 选用无过滤服务：[Quad9 地址](https://docs.quad9.net/services/)、[AdGuard 地址](https://adguard-dns.io/en/public-dns.html)。

Stash 使用 `fake-ip-filter: ["*"]` 返回真实 IP；不照搬 Mihomo 的 `enhanced-mode`、`respect-rules`、`direct-nameserver` 或 `empty-fallback` 字段。DNS 服务域名的启动解析和代理节点域名解析保留 system；节点解析走独立路径。system 的具体服务器由系统网络设置决定。

## 已修正的问题和边界

- Bybit 原先缺少专用 DNS policy，查询会落到 PROXY；现在已补齐。
- 当前 Mihomo 配置中，Binance 连接与 DNS 都使用远程 `Binance` 规则集，已移除宽泛的关键词兜底。Stash 版保留共享 inline `BinanceKeyword` 规则集，让该版本的连接和 DNS 使用同一关键词匹配条件。
- Stash 版 Netflix、Telegram 的 IP 路由使用远程列表，私有 IP 分类使用 `GEOIP,LAN`；Mihomo 版保留原生 GeoIP 分类。
- Stash 的 V2Fly 上游没有独立 `biliintl` 分类，Stash 版使用 inline `BiliIntl` 集合列出 `bilibili.tv`、`biliintl.com`、`dreamcast.hk`，优先于 Bilibili 直连规则。Mihomo 的 MetaCubeX 扩展数据包含 `biliintl`，Mihomo 版直接使用该分类。Stash 的这三条域名不代表 MetaCubeX 扩展集合的完整内容。来源：[V2Fly 国际站条目](https://raw.githubusercontent.com/v2fly/domain-list-community/master/data/bilibili)、[MetaCubeX 数据说明](https://github.com/MetaCubeX/meta-rules-dat/blob/master/README.md)。
- 精确域名、通配域名的 DNS policy 优先级高于集合。当前使用集合型 policy，集合之间的顺序与业务域名规则对应；后续增加 policy 时要同时检查两边优先级。
- IP-CIDR 和 GEOIP 需要目标 IP，不能据此选择第一次域名查询的 DNS。纯 IP 连接也没有可匹配的查询域名。
- Mihomo 的 `direct-nameserver` 可以让部分最终直连连接重新用 system 解析。Stash 版没有照搬这个字段：只在最后被 `GEOIP,CN` 判定直连、且此前未命中直连域名分类的连接，初次 DNS 仍可能经 PROXY。这一项不保证两版完全相同。
- Stash 版的 Binance 关键词兜底会匹配任何包含 binance 的域名，含无关域名；Mihomo 版已移除该关键词规则。远程规则和 geosite 数据更新也可能改变归类，不能凭节点的 MY/US 名称保证网站采用相同的 IP 地区判定。

启用后应在连接记录中分别检查 `www.binance.com → Binance`、`cloudflare-dns.com → Binance`，以及 Bybit、AI、YouTube、Telegram 对应的业务和 DNS 连接。配置文件检查不能代替 Stash 运行日志。

## 本次检查

2026-10-18 初次适配检查：YAML 结构、重复键、规则引用和当时两版的 38 个域名分类 DNS 出口对照通过；14 个规则匹配样例通过，覆盖关键词重叠和 Telegram IPv4/IPv6。随后 Mihomo 配置恢复原生 GeoIP / GEOSITE 分类，按独立配置维护，不再要求两版逐项使用同一提供器。

5 个远程规则源可读取且格式正确，Stash 使用的 33 个 geosite 分类均在当前上游存在；8 个公共 DoH 端点均返回有效 DNS 响应，其中 Quad9 使用 HTTP/2 验证。端点检查通过当前电脑网络完成，未在 Stash 中验证所选节点出口或导入兼容性。
