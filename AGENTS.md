# 多平台代理配置项目

## 项目目标

本项目维护同一套网络意图在不同代理客户端中的配置实现，当前包括：

- `vmutex.yaml`：Stash 配置
- `vmutex-hako.yaml`：Clash by Hako / mihomo 配置
- `vmutex-loon.lcf`：Loon 配置

## 修改约定

- 不把节点订阅地址、token、UUID、密码、私钥或其他凭据提交到仓库。
- 跨客户端迁移时保持以下行为一致：自有服务直连、中国域名和中国 IP 直连、指定海外服务代理、默认流量进入 `Proxy`。
- Stash/mihomo 使用 `fake-ip-filter`；Loon 使用 `real-ip`，不要把两者字段直接混用。
- mihomo 的 `GEOSITE`、`RULE-SET` 和 YAML Provider 不能未经转换直接放入 Loon；Loon 应使用 `[Remote Rule]` 和 Loon 兼容的文本规则。
- 远程规则必须使用稳定、公开、可解析的 URL，并在配置中保持正确的策略名称。
- 修改后至少检查 YAML/INI 语法、规则引用、策略组引用和远程资源格式。

## 验证重点

- 配置能被目标客户端载入。
- `Proxy`、`AI`、`Apple`、`APNs`、`TestFlight` 等策略组引用的策略真实存在。
- 会议服务的 STUN/UDP 没有被主动禁用。
- DNS 故障不会让整个配置永久等待；自建 DoH 应保留可用的回落路径。
