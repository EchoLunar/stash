# Stash 覆写配置

本仓库用于配合 3x-ui 订阅：订阅负责下发真实节点，覆写负责 DNS、策略组、分流规则和远程规则集。

使用 `stash.stoverride` 作为 Stash 覆写。

该覆写不定义真实节点，因此不会把订阅下发的节点清掉；`Proxy` 是兜底代理入口，`AI`、`Talkatone`、`Binance` 和 `Bybit` 使用 `include-all: true` 收纳订阅节点。

当前 `Final` 固定以 `Proxy` 作为兜底，未匹配流量不会因为策略组误选而直连。业务规则优先处理 `YouTube`、`Telegram`、`AI`、`TikTok`、`Talkatone`、`APNs`、`Apple`、`TestFlight`、`Binance` 和 `Bybit`；其余流量按中国域名、CDN、`CN_Mainland` 和 `GEOIP,CN` 依次判断，最后进入 `Proxy`。

DNS 使用 Redir-Host；参考 Accademia 的 WhiteList-02-Min 模板，内置 4,843 条手工展开的 GeositeCN DNS 策略，命中域名使用阿里 DNS，其他域名使用境外 DoH。普通 DNS 按规则跟随代理，代理节点域名使用独立的 `proxy-server-nameserver` 解析。国内分流使用 `GeositeCN_Domain`、中国 CDN 规则和 `GEOIP,CN` 兜底；ChinaMax 按上游模板默认保持关闭。`Final` 和 `Proxy` 不应重新加入 `DIRECT`，需要直连的流量应通过规则实现。

注意：不要把它当成独立订阅配置直接导入，因为文件本身不包含节点。
