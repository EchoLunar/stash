# VMUTEX 分流配置

`VMUTEX.yaml` 仅面向 Mihomo 内核；`VMUTEX.stoverride` 是独立维护的 Stash 覆写。两者包含分组、连接规则、DNS 和规则提供器，不包含节点凭据。

## Mihomo 使用

为 `PROXY` 选择节点；`AI`、`YouTube`、`Telegram`、`Crypto` 可单独选择节点，也可以选择 `PROXY` 跟随总策略。原 Binance、Bybit 分组合并为 `Crypto`，使用 [MetaCubeX 加密货币合集](https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/classical/category-cryptocurrency.list)，每 24 小时检查更新。合集还涵盖其他交易所、钱包和行情服务，并非仅限 Binance、Bybit。

`Crypto` 同时提供 `DIRECT` 和全部订阅节点。选择 MY 时，业务连接走 MY；需要经代理真实解析的该分类查询使用 Cloudflare / Google DoH 并绑定 `Crypto`。选择 `DIRECT` 时，目标域名改由本地系统 DNS 解析，以减少直连访问的 CDN 选址偏差。没有保存过选择时，`DIRECT` 是组内首项，重载后请确认选择。

业务连接使用 `RULE-SET,Crypto,Crypto`；需要真实解析时，DNS policy 使用同一 `rule-set:Crypto` 集合，并通过 `#Crypto` 绑定代理出口。更新配置后需为新建的 `Crypto` 分组选择所需节点（例如 MY），原分组的选择不会自动迁移；清理 DNS 缓存并关闭旧连接后，新请求才按新配置处理。

Mihomo 改用 `enhanced-mode: fake-ip`，只让 `geosite:private` 返回真实 IP。`direct-nameserver-follow-policy: false` 使直连连接需要真实解析时使用 `direct-nameserver: [system]`，包括 `Crypto` 选 `DIRECT` 的情况；代理连接仍由对应 DNS policy 选择上游。参考：[Mihomo DNS 配置](https://wiki.metacubex.one/config/dns/#direct-nameserver-follow-policy)。

`VMUTEX.yaml` 保留原生 `GEOSITE,telegram`、`GEOIP,telegram`、`GEOSITE,biliintl` 和 `GEOIP,netflix`，并让域名 DNS policy 使用对应 geosite 分类。Telegram 的域名规则与 IP 规则分别覆盖域名连接和直接 IP 连接。

这些扩展分类存在于 MetaCubeX geo 数据中；应使用包含这些分类的完整数据。此前仅因 Stash 兼容而添加的 `TelegramIP`、`NetflixIP`、`BiliIntl` 提供器已从 Mihomo 配置移除。参考：[MetaCubeX 数据说明](https://github.com/MetaCubeX/meta-rules-dat/blob/master/README.md)。

节点域名使用独立的阿里 DNS / DNSPod DoH 直连解析，不依赖代理建立。末尾 `GEOIP,CN,DIRECT,no-resolve` 只匹配已有目标 IP，不会为判断地区而提前解析未命中的域名。

## Stash 使用

1. 使用 Stash iOS/tvOS 3.6+ 或 macOS 4.3+；这里使用了独立节点 DNS 和远程规则集。
2. 在 Stash 中加载自己的节点订阅，再导入并启用 `VMUTEX.stoverride`。它整体替换 DNS、策略组、规则和规则提供器；不要与仓库旧的 `stash.stoverride` 同时启用。
3. 为 `PROXY` 选择节点；`AI`、`YouTube`、`Telegram`、`Crypto` 可分别选择节点或跟随 `PROXY`，`Crypto` 还可选择 `DIRECT`。`REJECT` 是显式拒绝选项，用来避免 Stash 空组按直连处理；请主动选择需要的出口。旧的 Binance / Bybit 分组选择不会自动迁移到 `Crypto`。
4. 重载配置、清理 DNS 缓存并关闭旧连接后，检查新连接的规则与出口。浏览器自带的加密 DNS 会绕开 Stash 的内置 DNS，应让浏览器使用系统 DNS。

官方说明：[覆写合并](https://stash.wiki/configuration/override)、[DNS 配置](https://stash.wiki/features/dns-server)、[规则集合](https://stash.wiki/rules/rule-set)、[策略组](https://stash.wiki/proxy-protocols/proxy-groups)。

## Stash DNS 设计

Stash 默认使用 Fake IP，让应用的域名查询先得到虚拟地址，连接建立时再按域名规则决定出口和是否需要真实解析。`fake-ip-filter` 包含 `geosite:private` 及 Apple、Windows 的少量网络连通性检测域名。私有域名由系统 DNS 返回真实 IP；新增的公网检测域名由阿里 DoH 返回真实 IP，因此这些域名会被本地解析器看到。QQ 与微信的部分本地登录域名已在 `geosite:private` 中，无需重复列出。若某个应用确实不能使用 Fake IP，再为确认过的具体域名增加例外，不对所有域名禁用 Fake IP。

`geosite:private` 是 [V2Fly 的 private 域名清单](https://github.com/v2fly/domain-list-community/blob/master/data/private)，不是私有 IP 网段。当前清单包括 `.lan`、`.localdomain`、`.local`、`home.arpa`、`.internal`、`.localhost`、`.test`、`.example`、`.invalid`，无点主机名、私有地址的反向解析区，以及部分路由器、本地服务和 Tailscale 名称（如 `router.asus.com`、`plex.direct`、`ts.net`）。这些是例子，不是完整且永久不变的清单；未列出的自定义内网域名不会因为解析结果是私有 IP 就自动命中。Stash 的 `GEOSITE` 数据由 V2Fly 社区维护，首次使用时需从 GitHub 加载，见 [Stash 规则文档](https://stash.wiki/rules/rule-types#geosite)。连接目标的私有 IP 则由独立的 `GEOIP,LAN` 规则处理。

Stash 使用 `follow-rule: false`。需要真实 IP 的公网域名统一由直连的阿里 DoH 解析；只保留一个上游，避免两个解析器并发返回不同 CDN 地址。连接规则仍按业务域名选择代理节点，DNS 不按业务组转发。默认 Fake IP 让普通代理请求先保留域名，减少发往本地解析器的查询；这不是“代理域名绝不经本地 DNS”的保证，例如要求真实 IP 的应用请求仍可能使用上述直连 DoH。

`default-nameserver: system` 用于阿里 / DNSPod DoH 服务域名的启动解析；`proxy-server-nameserver` 使用独立、直连的阿里 / DNSPod DoH 解析节点域名，避免建立代理时递归依赖自身。私有域名的 `nameserver-policy` 使用 `system`。末尾的 `GEOIP,CN,DIRECT,no-resolve` 与 Mihomo 保持一致：仅在已有目标 IP 时判断中国大陆归属，不会为了这条规则查询未命中的域名。

两版都依靠前面的域名分类决定直连或代理，但 DNS 实现不同。Stash 的固定直连域名在需要真实 IP 时从本地出口解析，有助于保持本地 CDN 选址；普通代理域名先用 Fake IP 保留名称。仍以域名进入分流、且未命中域名规则的连接会落到 `PROXY`，不再因末尾 GEOIP 规则提前触发本地查询；这可能让少数未收录的国内网站走代理。若应用已自行解析出真实 IP，`GEOIP,CN` 仍可据此选择直连。DoH 服务域名的启动解析、代理节点域名和私有域名仍有独立的直连解析路径；浏览器或应用自带的 DNS 可能绕开这里的设置，因此不能宣称绝对零泄漏或零 CDN 漂移。

## 连接规则与检查

Oracle 分类优先，业务专用分类排在境外、国内和最终兜底分类之前。`Rules/oracle.list` 包含四条 VPS IPv4 / IPv6 和 `oracle.com`、`oraclecloud.com`、`5202323.xyz` 的域名条目。`Oracle` 手选分组只含 `DIRECT` 与 `PROXY`，默认首项为 `DIRECT`；客户端已有保存选择时以客户端为准。切换会同时影响列表内 IP 和域名的所有端口，包括使用 IP 连接的 SSH。需先将 `Rules/oracle.list` 发布到远程仓库，再在客户端更新规则集。

两版业务连接规则按相同的分类和顺序排列。Stash 的 `Crypto` 与 Mihomo 共用同一份 [MetaCubeX 加密货币合集](https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/classical/category-cryptocurrency.list)，不再保留额外的 Binance 关键词兜底；`BiliIntl` 改用 [MetaCubeX 国际站列表](https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/meta/geo/geosite/biliintl.list)。Stash 不支持直接使用 Mihomo 的全部 GeoSite / GeoIP 数据：其普通 `GEOSITE` 使用 V2Fly 数据，`GEOIP` 使用自己的数据库；Netflix / Telegram 使用 MetaCubeX 的独立 IP 列表。因此两版业务规则文本和主要远程来源已对齐，但不能保证所有域名和 IP 在两个内核里命中完全相同。DNS 行为仍按各自配置独立维护。

启用后，在 Stash 中验证私有域名可由系统解析、节点域名能在代理建立前解析、国内域名直连、境外域名及 Crypto / AI 等专用域名按对应分组出站。检查普通域名收到 Fake IP 且实际连接成功；如发现特定应用需要真实 IP，记录域名后再加入 `fake-ip-filter`。还应检查 Stash 的 DNS 诊断和连接日志：静态配置检查不能代替客户端运行验证。远程规则更新可能改变域名归类，节点的 MY/US 名称也不能保证网站采用相同的 IP 地区判定。
