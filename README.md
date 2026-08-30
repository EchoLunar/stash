# Stash 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

使用 `stash.stoverride` 作为 Stash 覆写。

该覆写不定义真实节点，因此不会把订阅下发的节点清掉。`Proxy` 是兜底代理入口，`AI`、`Talkatone`、`Binance`、`Bybit`、`Polymarket` 和 `TikTok` 使用 `include-all: true` 收纳订阅节点。

当前 `Final` 固定以 `Proxy` 作为兜底，未匹配流量不会因为策略组误选而直连。业务规则优先处理 `YouTube`、`Telegram`、`AI`、`TikTok`、`Talkatone`、`APNs`、`Apple`、`TestFlight`、`Binance`、`Bybit` 和 `Polymarket`；其余流量按中国域名、CDN、`CN_Mainland` 和 `GEOIP,CN` 依次判断，最后进入 `Proxy`。

DNS 使用 Fake IP，代理请求尽量保留域名交给代理链路解析，减少本地 DNS 提前选错 CDN；中国域名通过内置 `geosite:cn` 使用国内 DoH，其他需要真实解析的域名使用境外 DoH。bootstrap 和代理节点解析使用独立的国内 DNS，避免递归查询和境外 bootstrap 在中国网络中不可用。普通 DNS 不跟随代理规则；`GeositeCN_Domain` 仅用于普通分流，避免依赖 Stash 未明确支持的 DNS `rule-set:`。启用轻量嗅探帮助 Stash 从 IP 连接恢复域名，但跳过 Apple、微信、QQ 等对目的地址敏感的服务。国内分流使用 `GeositeCN_Domain`、中国 CDN 规则和 `GEOIP,CN,no-resolve` 兜底；ChinaMax 按上游模板默认保持关闭。`Final` 和 `Proxy` 不应重新加入 `DIRECT`，需要直连的流量应通过规则实现。

配置还拦截 `geosite:category-httpdns-cn`、HTTPDNS 规则集和常见 STUN/TURN 流量，用于降低 HTTPDNS 跟踪和 WebRTC 公网地址泄露风险。STUN 拦截会影响 WebRTC/P2P、部分实时通话以及可能使用 STUN 的 Tailscale 直连；如果需要这些能力，可以移除覆写中的 4 条 WebRTC 规则。Stash 无法阻止应用绕过 VPN，或浏览器自身使用独立 DoH/代理，因此“无泄露”仍需配合系统和应用设置验证。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
该配置面向 Stash iOS/tvOS 3.6+，依赖 `geosite:` DNS 策略、Fake IP 和 `PROTOCOL,STUN` 支持；旧版本请勿直接使用。
