# Stash 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

使用 `vmutex.yaml` 作为 Stash 配置/覆写文件。

该覆写不定义真实节点，因此不会把订阅下发的节点清掉。`Proxy` 是兜底代理入口，`AI`、`Talkatone`、`Binance`、`Bybit`、`Polymarket` 和 `TikTok` 使用 `include-all: true` 收纳订阅节点。

当前 `Final` 固定以 `Proxy` 作为兜底，未匹配流量不会因为策略组误选而直连。业务规则优先处理 `YouTube`、`Telegram`、`AI`、`TikTok`、`Talkatone`、`APNs`、`Apple`、`TestFlight`、`Binance`、`Bybit` 和 `Polymarket`；其余流量按中国域名、CDN、`CN_Mainland` 和 `GEOIP,CN` 依次判断，最后进入 `Proxy`。

DNS 段使用 `dns: #!replace` 完全替换订阅中的 DNS，避免 `system` 或其他上游 DNS 合并进来。启用 Fake IP，`fake-ip-filter` 仅保留局域网、STUN、游戏连通性检测等必须获得真实 IP 的域名，不能使用 `"*"`。中国域名通过 `nameserver-policy`、`GEOSITE,cn,DIRECT` 和加密国内 DoH 处理，尽量保留本地 CDN 定位并避免明文 DNS；其他域名使用基于 IP 的境外 DoH，`follow-rule: true` 让它们按流量规则经由代理出口发送。Stash 3.6+ 的 `proxy-server-nameserver` 单独负责解析代理节点域名，避免 DNS 跟随代理后出现递归查询。

当前配置不拦截 HTTPDNS、STUN/TURN 或应用自建 DoH；Stash 无法阻止应用绕过 VPN，或浏览器自身使用独立 DoH/代理，因此“无泄露”仍需配合系统和应用设置验证。若 DNS 检测仍显示本地运营商，需在 Stash 的 DNS Inspection 中确认实际生效配置，并检查是否同时启用了其他 VPN、Private Relay 或浏览器安全 DNS。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
该配置面向 Stash iOS/tvOS 3.6+，依赖 `geosite:` DNS 策略、Fake IP、`follow-rule` 和 `proxy-server-nameserver`；旧版本请勿直接使用。
