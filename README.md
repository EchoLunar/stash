# 多平台代理配置

维护 Stash、Clash by Hako / mihomo、Loon 等代理软件的同一套分流思路和平台适配配置。

## 配置文件

- [Stash 配置](vmutex.yaml)
- [Clash by Hako 配置](vmutex-hako.yaml)
- [Loon 配置](vmutex-loon.lcf)

节点订阅不放入仓库。Stash 和 Loon 可以在客户端单独导入节点；Hako 需要在完整 mihomo 配置中提供 `proxies:` 或 `proxy-providers:`，不能仅依赖规则模板。

Loon 版使用 `[Remote Filter]` 汇总客户端当前可见节点，使用 `[Remote Rule]` 引用 Loon 兼容的文本规则；DNS 使用全局国外 DoH，并通过 `[Host]` 为高频国内域名指定国内 DoH。Loon 的远程规则只负责流量策略，不能直接作为 DNS 上游分流条件。

## 命名建议

当前建议的项目名称是 `proxy-configs`。公开 GitHub 仓库的最终名称需要在 GitHub 侧确认后再改，避免直接改变已有链接。
