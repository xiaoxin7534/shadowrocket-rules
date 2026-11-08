# 完美分流 Workbuddy 版

一套按业务类型分层的分流规则，同时提供 Shadowrocket 与 v2rayN 两个版本，规则结构完全对应。

- **不含任何节点信息**：无服务器地址、端口、UUID、密码，也无节点名称。
- **可直接订阅**：Shadowrocket 通过 URL 导入后，可随时在 App 内更新配置。
- 共 370 条规则、14 个策略段。

---

## 一、订阅

### Shadowrocket

主地址（GitHub 原始文件）：

```
https://raw.githubusercontent.com/xiaoxin7534/shadowrocket-rules/main/workbuddy-split.conf
```

备用地址（jsDelivr CDN，国内网络更稳，缓存约 12 小时）：

```
https://cdn.jsdelivr.net/gh/xiaoxin7534/shadowrocket-rules@main/workbuddy-split.conf
```

App 内：**配置 → 右上角 + → 从 URL 下载**，粘贴其中一个地址。
之后在配置列表里下拉即可更新，无需重复导入。

也可以直接扫码（指向备用地址）：

![二维码](qr-subscribe.png)

### v2rayN

下载 [`v2rayn-workbuddy-routing.json`](v2rayn-workbuddy-routing.json)，在
**设置 → 路由设置 → 导入规则** 中导入，然后重启 v2rayN 生效。

---

## 二、规则结构

规则自上而下匹配，命中即停。**顺序本身就是设计**——放错位置会导致后面的规则永远不生效。

| # | 规则 | 去向 | 说明 |
| --- | --- | --- | --- |
| 1 | 本地与局域网 | 直连 | 私有网段 + `.local/.lan`，最先放行 |
| 2 | 网络连通性探测 | 直连 | `captive.apple.com` 等，走代理会被系统误判为无网络 |
| 3 | QUIC / UDP 443 | 阻断 | 强制回落 TCP，避免 UDP 经代理时不稳定 |
| 4 | 广告与追踪 | 阻断 | `geosite:category-ads-all` 等价集合 |
| 5 | AI 服务 | 代理 | ChatGPT / Claude / Gemini / Perplexity / Cursor 等 60 余项 |
| 6 | 流媒体 | 代理 | Netflix / Disney+ / HBO / Hulu / Prime / YouTube / Twitch / TikTok / Spotify |
| 7 | 社交 | 代理 | Telegram（含 IP 段）/ X / Meta / Discord / Reddit / WhatsApp / Signal |
| 8 | 开发与 Google | 代理 | GitHub / GitLab / Docker / PyPI / npm / Vercel / Google 全家桶 |
| 9 | 交易与行情 | 代理 | TradingView / Binance / OKX / Bybit / 富途 / moomoo 等 30 项 |
| 10 | Apple 与微软 | 直连 | 国内有 CDN，走代理反而更慢 |
| 11 | 游戏社区 | 代理 | Steam 社区在国内不可直连 |
| 12 | 游戏商店与下载 | 直连 | Steam 商店/下载走 CDN，代理会拖慢速度并消耗流量 |
| 13 | 国内 | 直连 | `.cn` + 主要国内服务域名 + `GEOIP,CN` |
| 14 | 兜底 | 代理 | 未命中任何规则的流量 |

### 为什么这样排序

- **1、2 必须最前**：局域网与探测流量一旦被代理，会出现「连不上内网」「Wi-Fi 显示无网络」。
- **3 在业务规则之前**：QUIC 就是 UDP 443，若放在后面，已被前面规则命中的流量仍会走 QUIC，等于没拦。
- **10 在 5 之后**：微软规则覆盖 `copilot.microsoft.com`，若 Apple/微软直连排在 AI 之前，Copilot 会被错误直连而不可用。
- **13 在 14 之前**：国内直连是「例外」，必须排在兜底代理前面。

---

## 三、两个版本怎么选

| 文件 | 策略 | 适用场景 |
| --- | --- | --- |
| `workbuddy-split.conf` | 全部代理流量走 `PROXY`（当前选中节点） | **推荐**。零配置即可用，适合订阅 |
| `workbuddy-split-groups.conf` | 分 AI / 流媒体 / 社交 / 交易 四个策略组 | 需要「AI 走美国、交易走日本」这类精细控制 |

`workbuddy-split-groups.conf` 里的节点名是占位符（`🇺🇸 美国-01`、`🇯🇵 日本-01` 等），
导入前需要替换成你自己订阅里的真实节点名，否则策略组会为空。

> 分组必须写死节点名，而节点名会暴露所用服务商，所以**公开版不提供写死节点名的配置**。

---

## 四、自定义

- **新增直连域名**：加在「国内」段，`DOMAIN-SUFFIX,example.com,DIRECT`
- **新增代理域名**：加在「开发与 Google」段，`DOMAIN-SUFFIX,example.com,PROXY`
- **恢复 QUIC**：删除 `PROTOCOL,QUIC,REJECT` 一行
- **临时全局直连**：把 `FINAL,PROXY` 改成 `FINAL,DIRECT`

---

## 五、说明

- 规则中的域名清单为手工整理，覆盖主流服务，长尾域名可能缺失。
- `geosite` / `geoip` 标签在 v2rayN 版中使用，Shadowrocket 版已展开为等价域名与 IP 规则。
- 广告拦截依赖一条在线规则集（blackmatrix7 Advertising），下载失败时仅影响广告拦截，不影响其他规则。
