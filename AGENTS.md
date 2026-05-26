# AGENTS.md

## 项目用途

这个仓库用于维护个人 Stash 覆写配置。核心目标是集中管理 Stash 的代理分组、分流规则和远程规则提供者，让不同服务可以按用途或地区选择合适的策略组。

主要配置文件是 `stash.stoverride`。

## 当前项目结构

- `stash.stoverride`：Stash override 配置文件，包含 `proxy-groups`、`rules` 和 `rule-providers`。
- `README.md`：项目简短说明。

## 配置概览

`stash.stoverride` 使用 Stash/Clash 风格的 YAML 配置，并通过 `#!replace` 替换相关配置段。

主要结构：

- `proxy-groups`：定义代理策略组。
  - `Final`：最终入口，可选择 `Proxy` 或 `DIRECT`。
  - `Proxy`：主代理组，聚合各地区组和 `AllServer`。
  - 服务分流组：`AI`、`Apple`、`YouTube`、`Telegram`、`Talkatone`、`Crypto`、`Polymarket`、`GitHub`、`Streaming`、`TikTok`、`Google`、`Microsoft`、`Games`。
  - 地区组：`HongKong`、`Singapore`、`America`、`TaiWan`、`UK`、`Germany`、`Japan`、`Korea`、`SEA`、`Europe`、`AllServer`。
  - 自动组：按地区拆分的 `url-test`、`fallback`、`load-balance` 组。
- `rules`：定义分流顺序，最后使用 `MATCH,Final` 兜底。
- `rule-providers`：引用远程规则集，主要来自 `Coldvvater/Mononoke`。

## 维护约定

- 修改 `rules` 时必须确认目标策略组已在 `proxy-groups` 中存在。
- 新增服务分流时，优先新增独立策略组，再在 `rules` 中引用它。
- 不要随意调整规则顺序；靠前规则优先级更高。
- 保留 `MATCH,Final` 作为最后兜底规则。
- 保持 YAML 缩进一致，列表项使用两个空格缩进。
- 保留 `#!replace` 标记，避免破坏 Stash override 的替换语义。
- 地区过滤正则应覆盖中文、英文、缩写和常见旗帜标识。
- 自动组默认使用 `interval: 300` 和 `lazy: true`，除非有明确理由不要改动。
- 远程规则提供者默认使用 `interval: 86400`。
- 不要把订阅信息、账号信息、密钥或私有节点写入仓库。

## 修改检查清单

修改 `stash.stoverride` 后检查：

- YAML 缩进是否有效。
- 所有 `rules` 引用的策略组是否存在。
- 所有地区组引用的自动组、故障转移组、负载均衡组是否存在。
- 所有 `rule-providers` 名称是否和 `rules` 中的 `RULE-SET` 名称一致。
- 最后一条规则是否仍为 `MATCH,Final`。
- 远程规则 URL 是否可读且文件名拼写正确。
