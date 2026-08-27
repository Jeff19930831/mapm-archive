# progress

## 2026-06-05

Created this Obsidian-visible pointer folder for `clash-governance`.

Evidence:

- `.project-status.yaml` already pointed to `WinCodex/clash-governance/`, but the folder did not exist.
- The pointer now links back to the code repo at `D:\Workspace\clash-governance`.
- Full checkpoint evidence remains in the code repo `progress.md`.

## 2026-06-05 治理发现：全局 TUN 把基础设施 IP 走了不稳定代理节点

排查 cloud-memory 服务器 (`43.133.86.33`, 腾讯云) 接入抖动时定位到一条 Clash 治理问题（先在 Mac 的 Clash Verge 上发现，规则同样适用于本 Windows 机）：

- **现象**：全局 TUN 把到 `43.133.86.33` 的流量（SSH:22 + HTTP:3111）塞进代理出口节点 → 经代理 HTTP `livez` 5 次 1 次超时、延迟 0.5–3.5s 乱跳；SSH 长会话直接卡死。
- **验证**：改走物理网卡直连（Mac: `route add -host 43.133.86.33 <en0网关>`）后 HTTP `livez` 8/8 成功、稳定 ~0.08s（快 10–40 倍、零失败）。
- **治理规则（建议加入受管 Clash 配置）**：对可直连的国内/基础设施 IP 加 DIRECT 不走代理：
  ```
  IP-CIDR,43.133.86.33/32,DIRECT
  ```
  可扩到其它自有服务器/腾讯云段。临时静态路由重启失效，**DIRECT 规则才是持久解**。
- **落点**：权威修改在代码仓 `D:\Workspace\clash-governance` 的受管 Clash 配置；本 pointer 只记结论。Mac 侧 Clash 也需同样加该 DIRECT 规则。

## 2026-06-11

Pointer refreshed after the code repo checkpoint.

Evidence stays in the code repo:

- DIRECT/TUN fake-ip TLS fix: `D:\Workspace\clash-governance\configs\dns\tun-fake-ip-direct.yaml`
- CC Switch local proxy and Gemini settings recovery: `D:\Workspace\clash-governance\docs\runbooks\cc-switch-local-proxy.md`
- Full checkpoint log: `D:\Workspace\clash-governance\progress.md`

## 2026-07-03 Mac 侧落地：按方案配置本机 Clash Verge Rev（mac-claude）

把代码仓的治理方案应用到本 Mac（此前 Mac 只是指针，扩展链全空）。按 `docs/runbooks/clash-change-control.md` 流程执行。

- **落点**：Clash Verge Rev at `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev`
  - 持久扩展链（治理源）：`profiles/pycoCBrBQfBr.yaml`(proxies) / `rEXGSdoAVULO.yaml`(rules) / `mf9gXxaatHSI.yaml`(merge dns)
  - 运行时镜像：`clash-verge.yaml` + `clash-verge-check.yaml`
- **应用内容**（源自 `configs/`）：
  - 节点 `Claude-Anthropic SOCKS5`（configs/nodes）
  - claude/anthropic → SOCKS5；MCP 域名(exa/context7/grep.app) → 雕云；`43.133.86.33/32` + 国内 AI(kimi/bigmodel/z.ai/ddsst/ppchat/zcode-ai) + Coze → DIRECT（configs/rules）
  - DNS fake-ip blacklist + direct-nameserver（configs/dns）
- **Mac 适配**：剔除工作网 DNS `10.1.18.181`（Windows ejianlong 内网专用），只留公共国内 DNS `119.29.29.29 / 223.5.5.5 / 223.6.6.6`。
- **验证**：`verge-mihomo -t` 通过；socket `PUT /configs` 热加载 HTTP 204；SOCKS5 节点延迟 297ms；`api.anthropic.com` HTTP 405（链路通）；规则表顺序正确（DIRECT 基础设施/国内 → 雕云 MCP → SOCKS5 Claude，均在订阅规则之上）。
- **回滚备份**：`~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-gov-backup-20260703-105123/`
- **权威记录待补**：完整证据仍应写入代码仓 `D:\Workspace\clash-governance\progress.md`（Mac 访问不到该 Windows 仓，需在 Windows 侧补记 + commit）。

## 2026-07-23 Mac 侧落地：TDS / 建龙内网域名 Clash-on 下载修复

在 `tds-tools` 真实下载验证中定位到：登录态有效，但 Clash Verge TUN/Fake-IP + 系统代理会让 `tds-report.ejianlong.com` 走错 DNS/路由，导致 jmreport iframe / Chromium 导航 `net::ERR_CONNECTION_CLOSED`。

本机 live Clash Verge Rev 已修复：

- **DIRECT 规则**：`DOMAIN-SUFFIX,ejianlong.com,DIRECT`、`DOMAIN,tds.ejianlong.com,DIRECT`、`DOMAIN,tds-report.ejianlong.com,DIRECT`、`DOMAIN-KEYWORD,ejianlong,DIRECT`。
- **DNS / Fake-IP**：`+.ejianlong.com`、`tds.ejianlong.com`、`tds-report.ejianlong.com` 加入 fake-ip filter；公司 DNS policy 使用 `10.1.18.181` + `114.114.114.114`。
- **Hosts**：`tds.ejianlong.com` / `tds-report.ejianlong.com` 临时 pin 到公司内网 `10.80.193.3`。
- **TUN route exclude**：排除 `10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16`，避免内网 DIRECT 又被 TUN 回卷。
- **macOS proxy bypass**：Ethernet / Wi-Fi 系统代理例外加入 `*.ejianlong.com`、`tds.ejianlong.com`、`tds-report.ejianlong.com`，并写入 `verge.yaml`。

落点：

- 运行配置：`~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/{clash-verge.yaml,clash-verge-check.yaml,config.yaml,verge.yaml}`
- 持久扩展链：`profiles/mf9gXxaatHSI.yaml`（DNS/hosts）与 `profiles/rEXGSdoAVULO.yaml`（rules）
- 备份：`~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/tds-ejianlong-backup-20260723-090029/`

验证：

```bash
curl -vkI -x http://127.0.0.1:7897 https://tds-report.ejianlong.com/jmreport/view/1055351755192311808
TDS_DIRECT=0 npm run auto
```

结果：Clash TUN + 系统代理开启时，mixed-port 路径返回 HTTP 200；`tds-tools` 在禁用脚本 direct pin 的情况下成功下载 `销售合同跟踪表_20260723.xlsx`。

风险：如果公司内网 IP 变化，需要更新 Clash hosts 中的 `10.80.193.3`；临时可在 `tds-tools` 用 `TDS_HOST_IP=<新IP> npm run auto` 兜底。

## 2026-08-12 Windows checkpoint

- Refreshed this pointer after migrating the active Windows clean profile and its five bound enhancements.
- Fixed the empty Netflix `url-test` construction, validated both generated configs, and hot-reloaded Mihomo.
- Reverified the existing dedicated SOCKS5 as alive and usable; no endpoint or credential update was required.
- Full migration, verification, rollback, and checkpoint evidence: `D:\Workspace\clash-governance\progress.md`.

## 2026-08-27 WeChat Official Account DIRECT checkpoint

- Added exact DIRECT routing for the WeChat API, operator backend, and developer portal in the authority repository.
- Verified both generated configs, named-pipe runtime rules, direct HTTPS transport, and the 10-file rollback snapshot.
- Verified WeWrite run 20260827-155208-af235c completed and created the requested draft.
- Cleared the persistent WeWrite AppID/AppSecret fields after Secret Gate detected them; no credential payload was added to Git or pointer docs.
- The disclosed AppSecret must be rotated before reuse, and the DIRECT public IP must be rechecked before the next publish.
- Authority repository checkpoint: 0133648.
- Full rules, runbook, verification, and rollback evidence: D:\Workspace\clash-governance\progress.md.
