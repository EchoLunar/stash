# AGENTS.md

## 项目用途

这个仓库用于维护个人 Stash 覆写配置。核心目标是由 3x-ui 订阅负责下发真实节点，并由覆写统一接管 DNS、代理分组、分流规则和远程规则提供者。

主要配置文件是 `stash.stoverride`。

## 当前项目结构

- `stash.stoverride`：Stash override 配置文件，包含 `proxy-groups`、`rules` 和 `rule-providers`。
- `README.md`：项目简短说明。

## 配置概览

`stash.stoverride` 使用 Stash/Clash 风格的 YAML 配置，并通过 `#!replace` 替换相关配置段。

主要结构：

- `proxy-groups`：定义代理策略组。
  - `Final`：最终入口，可选择 `Proxy` 或 `DIRECT`。
  - `Proxy`：主代理组，通过 `include-all: true` 收纳订阅下发的全部节点。
  - 服务分流组：`AI`、`Apple`、`APNs`、`Amazon`、`YouTube`、`Telegram`、`Talkatone`、`Binance`、`Polymarket`、`GitHub`、`Streaming`、`TikTok`、`Google`、`Microsoft`、`Games`。
  - `Polymarket`：除 `Proxy` 和 `DIRECT` 外，也通过 `include-all: true` 提供单节点选择。
- `dns`：完全替换订阅中的 DNS 配置，使用 Fake IP 和国内 DoH；需要代理的域名不应随意加入 `fake-ip-filter`。
- `rules`：定义分流顺序，最后使用 `MATCH,Final` 兜底。
- `rule-providers`：引用远程规则集，主要来自 `Coldvvater/Mononoke`。

## 维护约定

- 修改 `rules` 时必须确认目标策略组已在 `proxy-groups` 中存在。
- 新增服务分流时，优先新增独立策略组，再在 `rules` 中引用它。
- 不要随意调整规则顺序；靠前规则优先级更高。
- 保留 `MATCH,Final` 作为最后兜底规则。
- 保持 YAML 缩进一致，列表项使用两个空格缩进。
- 保留 `#!replace` 标记，避免破坏 Stash override 的替换语义。
- `Proxy` 和 `Polymarket` 应保留 `include-all: true`，确保能收纳订阅下发的真实节点。
- 对可能受 DNS 污染且需要代理的域名，避免强制使用系统 DNS 或加入 `fake-ip-filter`。
- 远程规则提供者默认使用 `interval: 86400`。
- 不要把订阅信息、账号信息、密钥或私有节点写入仓库。

## 修改检查清单

修改 `stash.stoverride` 后检查：

- YAML 缩进是否有效。
- 所有 `rules` 引用的策略组是否存在。
- 所有策略组内部引用的策略组或内置策略是否存在。
- 所有 `rule-providers` 名称是否和 `rules` 中的 `RULE-SET` 名称一致。
- 最后一条规则是否仍为 `MATCH,Final`。
- 远程规则 URL 是否可读且文件名拼写正确。
