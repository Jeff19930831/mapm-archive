# Jeff1993 协作入口（collab-entry）

> 版本: v3.16 / 2026-06-14
> 目的：所有设备、所有 Agent 接力开发的人工统一入口。根目录 `README.md` 仍是自动生成的项目 dashboard，不作为人工规则入口。
> v3.16 改动: MAPM 综合优化 — (1) 通用配置地基（`.gitattributes` + `.editorconfig` + `scripts/_platform.py` 统一 `rel_posix`/`read_text_utf8`（容忍 BOM）/`write_text_lf`（强制 LF），10 个治理脚本迁移收口）；(2) 共享文案单一源（新增 `docs/sop/agentmemory-mcp-usage.md`，7 处 Doc Lifecycle Gate 块 + Memory curl 模板改为引用）；(3) Codex 双轨合并（4 个镜像压缩为指针 + 2 个 ext 改差异化段，secret gate 从 fail-closed 修正为警告对齐 Claude 版）；(4) checkpoint 拆 3 子 skill（`checkpoint-archive`/`checkpoint-memory`/`checkpoint-git`）+ 主流程编号重排为连续 1-16 步，con-dev Phase 2.5 复用 `checkpoint-memory` Recall；(5) 新增 `agent-handoff` skill（同设备异 agent 交接，生成临时文件）；(6) `deep-search` 参数化（移除 wechat-macro-kb 硬编码，改 env var 协议，任何项目可适配）。
> v3.8 改动: checkpoint 新增 Memory Hooks（Step 2.5 Recall + Step 5.7 Store），每次 checkpoint 自动与云端 agentmemory 双向同步项目状态和 secret 引用；cloud-memory 项目落地（agentmemory v0.9.21 部署在腾讯云 43.133.86.33:3111）。默认不写真实 secret payload。
> v3.8.1 改动: con-dev 新增 Phase 2.5 Memory Recall（会话启动时搜索云端记忆）；AGENTS.md 增加记忆使用规则（三层读取：L1 冷启动/L2 周期同步/L3 即时查询）；新增 `troubleshoot:<name>` scope 用于排障记录。
> v3.9 改动: 新增 Doc Lifecycle Gate。入口文档只回答”现在怎么接手”，历史文档回答”以前为什么这么做”，runbook 回答”常见问题怎么安全处理”；checkpoint 和所有 onboarding/skill 都必须遵守入口短、历史归档、runbook 承载常驻问题。
> v3.10 改动: 新增项目联动协同规范。目录/schema/接口/指标/调度等契约变化必须同步上下游、contract/runbook 和验证证据。
> v3.11 改动: 明确当前四文档以小写 `README.md` / `plan.md` / `progress.md` / `handoff.md` 为主；旧大写入口仅作兼容/历史镜像，checkpoint 前必须优先刷新小写入口。
> v3.12 改动: agent-kit 同步工具落地（`scripts/agent_kit_sync.py`，manifest `skill_sync` 驱动，4 目标目录）；`Skill/` 废弃为指针；`项目规范.md` 归档，独特内容迁移到 `docs/sop/`；§ 7 新增 § 7.0 同步工具；skill 表增加 `deep-search` / `session-handoff`；con-dev Memory Recall 增加 MCP 优先。
> v3.13 改动: § 1 补齐云端记忆基础常识（单一共享云端不分设备/版本铁律 agentmemory↔iii-sdk↔iii引擎/NSSM 服务恢复）与基础设施网络访问坑（Clash 全局 TUN 走不稳定节点 → DIRECT 规则 + SSH 连接复用）；新增 `docs/sop/cloud-memory-and-network.md` 故障手册；落点项目 `cloud-memory` / `clash-governance`。
> v3.14 改动: agentmemory MCP 客户端配置标准升级为"直连 node"——`npm i -g @agentmemory/mcp` 后 MCP command 直接用 node 调 `bin.mjs`，弃用 `npx -y`（冷启动联网下载 = 客户端抖动主因）；`AGENTMEMORY_FORCE_PROXY=1` 为强制项（不设则 livez 探测失败会静默回退本地 InMemoryKV，记忆分叉）。推荐配置块见 `cloud-memory/handoff.md`；init-agent skill Step 6.5 已按新标准更新。
> v3.15 改动: checkpoint Step 7/7.1 改为按项目范围 staging（禁用 `git add -A`/`git add .`、排除 `.obsidian/`、新增 `.py` 用 `git add -f`），避免共享文档仓跨项目串味；checkpoint Step 2.5/5.7 Memory Recall/Store 改 MCP-first（curl 回退）；新增 `docs/sop/windows-python-setup.md`（Windows `python3` shim，让 skill 的 `python3` 调用跨平台可用）；governance_lint / build_governance_graph 修 Windows 路径分隔符（`.as_posix()`）；agent-kit manifest 债务清理（`scan_debt` 通用化，sync 只声明有源类别）。

## 0. 新 Agent 先读这 5 步

1. 读取本文，确认全局规则和文档地图。
2. 读取 [`onboarding-compact.md`](onboarding-compact.md)（精简入口，~1.4KB）。完整规则按需查 [`onboarding.md`](onboarding.md)。
3. 通过根 [`README.md`](README.md) 或 `.project-status.yaml` 找到目标项目目录。
4. 在项目目录:
   - **若存在 `docs/onboarding/BOOTSTRAP.md`** (大项目附加层): 优先读 `handoff.md` 的 Persistent/Current Takeover + `BOOTSTRAP.md`, 按 BOOTSTRAP § 3 场景跳转表按需展开。**不读全 CODEMAP/decisions/archive**。Token 预算 ≤ 5KB。
   - **否则** (小项目, 默认): 若存在 `scripts/dump_state.sh` 先运行它；读 `handoff.md` 的 Persistent/Current Takeover → `plan.md` Active 区 → `README.md` 架构段；若不存在小写文件，则兼容读取 `HANDOFF.md` / `PROGRESS.md`。
5. 开始工作前检查 Git/Obsidian 冲突；结束时执行 `/checkpoint`。

## 1. 当前同步模型

- **Obsidian Sync**：负责让 Jeff1993 Markdown 文档在多设备间快速可见。
- **Git / GitHub**（`jeff1993-docs`）：负责 checkpoint、版本历史、回滚和跨设备冲突处理。
- **代码仓库**：每个项目独立 Git 仓库；本地地址登记在 `.project-status.yaml` 的 `code_paths`（多设备）或旧 `code_path`（兼容）里，新设备按 § 4 解析后再 `git clone`。
- **项目同步清单**：大型项目在 `.project-status.yaml:sync` 登记需要跨设备保持可见的文档、onboarding、配置和关键源文件；不要把缓存、依赖目录、构建产物当作同步目标。
- **长期运行知识**：功能配置方法、服务器接入、恢复步骤、secret 引用写入项目 `docs/runbooks/` 或全局 `docs/sop/`；只同步引用和步骤，不同步密钥值。
- **Doc Lifecycle Gate**：所有项目入口文档保持短。`handoff.md` / `HANDOFF.md` 只放项目不变量、常驻问题雷达、当前目标、下一步、阻塞和索引；`plan.md` 只放 Active / Next / Archived Plans；完成记录进入 `progress/YYYY-MM.md` 或 `progress.md`；旧方案、长评审和历史 checkpoint 进入 `archive/**`。详见 `MAPM/docs/superpowers/specs/2026-05-22-doc-lifecycle-gate-design.md`。
- **项目联动协同**：凡一个项目/模块的产物被另一方消费，目录、schema、接口、指标、调度、配置、secret 引用的变更都视为联动契约变更；必须同步检查上下游、更新 contract/runbook 并留下验证证据，不得只改生产方或消费方一端就宣称完成。
- **云端记忆 (agentmemory)**：checkpoint 时自动与云端记忆双向同步。Step 2.5 拉取项目相关记忆为 handoff 刷新提供上下文；Step 5.7 写入项目状态和 secret 引用（`project:<name>` / `credentials:<name>` / `decision:<name>`），供跨设备/跨 session agent 搜索使用。默认不写真实 secret payload；只有认证、HTTPS、allowlist 和用户明确授权同时满足时才另行登记。服务地址：`http://43.133.86.33:3111`。
  - **单一共享云端，不分设备/不分 agent**：某设备搜不到别的设备写的记忆 = 该设备配置错了（多半在写本地），不是"按设备隔离"。验证 `curl …/livez`；端点用 `livez`/`search`/`remember`（`status`/`memories` 不可用）；关键词 search 有几分钟索引延迟（非故障）。
  - **版本铁律**：服务器 `agentmemory ↔ iii-sdk ↔ iii 引擎` 必须配套；iii 引擎默认后台自动更新，自升到不兼容版本会让 `/agentmemory/*` 全 404、所有设备退回本地。
  - **客户端配置标准（v3.14 / 2026-06-12 起）**：MCP 一律 `npm i -g @agentmemory/mcp` 后用 node 直调全局 `bin.mjs`，弃用 `npx -y`（冷启动联网下载 = 抖动主因）；`AGENTMEMORY_FORCE_PROXY=1` 强制保留——不设时 livez 探测失败会**静默回退本地** `~/.agentmemory/standalone.json`，记忆分叉且云端搜不到。推荐配置块见 `cloud-memory/handoff.md`，新设备入职由 init-agent skill Step 6.5 按此检查。
  - 故障"全设备记忆互不可见" / 服务恢复（NSSM 服务 `iii-engine`+`agentmemory-worker`）→ 见 [`docs/sop/cloud-memory-and-network.md`](docs/sop/cloud-memory-and-network.md) 与 `cloud-memory/` 项目。
- **基础设施网络访问（Clash/VPN）**：到自有服务器 IP（如 agentmemory `43.133.86.33`）若被全局 TUN 代理走不稳定节点，会出现 SSH 卡死 / HTTP 抖动。持久解：Clash 加 `IP-CIDR,<ip>/32,DIRECT`（见 `clash-governance`）；SSH 用连接复用+保活（`~/.ssh/config` ControlMaster）。详见 [`docs/sop/cloud-memory-and-network.md`](docs/sop/cloud-memory-and-network.md) § 5。
- **规则**：Obsidian 让文件到达；Git checkpoint 记录可信状态；云端记忆让 agent 跨设备记住项目上下文。

## 2. 文档地图

### 全局文档

| 文件 | 职责 |
|---|---|
| `collab-entry.md` | 人和 Agent 的统一入口（本文件）|
| `onboarding-compact.md` | 精简入口（~1.4KB），每次对话自动加载 |
| `onboarding.md` | 完整版 Agent 规则（按需读取，非自动加载） |
| `README.md` | 自动生成的 General dashboard；不要在 AUTO 区手写规则 |
| `项目规范.md` | 已归档（降级为索引指针）；权威规范见 `onboarding.md` |
| `项目总览.md` | 项目总览文档 |
| `定时任务总览.md` | 全局定时任务汇总 |
| `agent-kit/` | skills / prompts / configs / secrets / bootstrap 的同步合同 |
| `Skill/` | 已废弃，指针目录 → `agent-kit/claude/skills/` |
| `agent-kit/claude/skills/` | Claude Code skill 跨设备同步目录（`con-dev`、`checkpoint` 等）|
| `dev-log/` | 每日全局开发日志（`YYYY-MM-DD.md`），位于 Jeff1993 根目录 |
| `MAPM/` | 项目规范治理主项目；维护 collab-entry、onboarding、agent-kit、通用 skill 和治理脚本的规则边界 |

### 全局辅助文档 (docs/)

| 目录 | 职责 | 更新时机 |
|---|---|---|
| `docs/templates/project/` | 四文档标准模板 | 模板升级时 |
| `docs/specs/` | 全局/跨项目设计文档（不属于单一项目时放这里）| 设计落地后 |
| `docs/proposals/` | collab-entry / onboarding 等元规则的修订提案 | 提案提出时 |
| `docs/health-reports/` | 项目状态健康巡检报告（脚本生成）| 巡检脚本运行时 |
| `docs/backups/` | YAML / 关键文档定期备份 | 自动化备份脚本 |
| `docs/superpowers/` | superpowers skill 产出（specs/plans 等）| skill 使用时 |
| `docs/sop/` | 全局 OPS lore：跨项目运维规范、故障手册 | 沉淀经验时 |
| `docs/contracts/` | 跨项目耦合契约（可选，复杂关系才用）| 接口契约首次/重大修订时 |

### 项目内长期运行知识

| 目录 / 文件 | 职责 | 更新时机 |
|---|---|---|
| `<project>/docs/runbooks/*.md` | 功能配置方法、服务器接入、恢复步骤、secret 引用卡片 | 配置/部署/密钥引用变化时 |
| `<project>/.env.example` / `config*.example.*` | 脱敏配置示例 | 新增环境变量或配置项时 |
| `agent-kit/secrets/allowlist.yaml` | secret payload 私有路径 allowlist（只放路径和用途）| 用户明确 opt-in 同步 secret payload 时 |
| `<project>/progress/YYYY-MM.md` | 月度完成记录、旧 handoff 完成批次、验证证据 | 入口文档出现历史堆积时 |
| `<project>/archive/**` | 旧方案、长评审、历史 checkpoint、退役计划 | 历史仍需追溯但不应进入冷启动上下文时 |

### 每项目四文档 (基线, 所有项目)

| 文件 | 职责 | 更新时机 |
|---|---|---|
| `README.md` | 稳定范围、方向、架构、运行方式、链接 | 项目范围/功能/设置发生稳定变化 |
| `plan.md` | Active / Next / Archived Plans；未完成任务和归档索引 | 工作开始、计划变化、checkpoint 完成任务后 |
| `progress.md` | 已完成事实、验证证据、checkpoint/commit 记录 | 每次 checkpoint |
| `handoff.md` | Persistent / Current Takeover / Index；给下一位 Agent/下一台设备的最短接力状态 | 每次 checkpoint 或中断前 |

兼容期至 **2026-06-30**：旧项目的 `PROGRESS.md` / `HANDOFF.md` 继续可读；若同目录存在小写文件，小写 `progress.md` / `handoff.md` 是当前真相源，大写只作历史镜像。

### 大项目附加层 (LOC ≥ 10K 或源文件 ≥ 20 启用)

| 文件 | 作者 | 职责 |
|---|---|---|
| `docs/onboarding/BOOTSTRAP.md` | 模型起草 + 用户审改 | Agent 启动首读, 含场景跳转表 |
| `docs/onboarding/CODEMAP.md` | 脚本表 + 模型一行职责 | 全模块结构 + 大模块清单 |
| `docs/onboarding/module-cards/*.md` | 模型起草 + 用户审 | LOC > 1000 模块的 5 段名片 |
| `docs/onboarding/decisions/*.md` | 模型起草 (决策时) + 用户审 | ADR 架构决策记录 |
| `docs/onboarding/decisions/INDEX.md` | 脚本 | ADR 索引 (自动) |

启用方式: 在项目根跑 `/init-onboarding` (Claude) 或 prompt "初始化 onboarding" (Codex)。
完整设计: [`MAPM/onboarding-suite/docs/design.md`](MAPM/onboarding-suite/docs/design.md)。

### 跨项目关系 (L5)

跨项目耦合（消费方、上下游、治理者-被治理者）通过 `.project-status.yaml` 的 `relations` 字段登记，使其机器可读、可被 dashboard 渲染、可被脚本反查。

复杂耦合（如多接口、多次联动变更）可选地写一份 `docs/contracts/<声明方>--<对方>.md`。不是每条关系都需要契约文档——简单的"消费"关系仅 YAML 登记即可。

涉及联动契约变更时，先运行 `context_gate` / `project_impact` 查影响面；复杂或重复发生的耦合必须升级为 contract，并在 producer 侧写 `[CONTRACT-CHANGE]`、consumer 侧写 `[CONTRACT-ADAPT]` 证据。

详细 schema、命名规约、变更工作流见 `onboarding.md` § 13。

### 跨项目关系工具（查询 / 质检）

为避免每次都靠人工翻 YAML + contract，补充 3 个轻量脚本：

- `python3 scripts/governance_lint.py`：校验 `relations` 目标与 `contract` 文件一致性。
- `python3 scripts/build_governance_graph.py`：生成 `var/governance_graph.json`。
- `python3 scripts/project_impact.py <project>`：查询某项目的上游/下游/治理/契约影响面。
- `python3 scripts/context_gate.py --project <project> --task "<任务描述>" [--enforce]`：主会话上下文门控，自动判定 `fast/balanced/full`，`full` 档位下强制先跑 impact。
- `python3 scripts/request_explorer.py --project <project> --task "<复杂跨项目任务>"`：**申请 explorer 唯一入口**（含强制门禁 + 证据留档）；未通过时不得派 explorer。

建议顺序：`context_gate` → `governance_lint` → `build_governance_graph` → `project_impact`。

### 项目注册表工具（路径 / 同步）

- `python3 scripts/project_registry.py resolve <project>`：按当前设备解析项目本地代码路径。
- `python3 scripts/project_registry.py sync-list <project> --summary`：展开项目同步清单，检查大项目需要保持可见的文件集合。
- `python3 scripts/project_registry.py lint <project>`：校验 `code_paths` / `sync` schema 和 required 文件。

## 3. 项目状态枚举

| 状态 | 含义 |
|---|---|
| `规划中` | 尚未启动，处于构想/设计阶段 |
| `进行中` | 有人在活跃开发 |
| `运行中` | 有定时任务/自动化在持续运行 |
| `维护中` | 无活跃开发，偶发维护 |
| `已完成` | 目标达成，无需维护 |
| `已合并` | 并入其他项目，见 `archive` 字段 |
| `已归档` | 不再活跃，移入归档目录 |

## 3.1 Owner 枚举

| 值 | 含义 |
|---|---|
| `claude` | Claude（任意设备）|
| `codex` | Codex（任意设备）|
| `kimi` | Kimi / ZCode |
| `unassigned` | 暂无负责 Agent |

> Owner 按工具区分，不区分设备。设备信息通过 Agent 启动时的身份确认（见 `onboarding.md § 1`）体现。

## 4. 新设备 / 新 Agent 初始化

详见 [`onboarding.md § 11`](onboarding.md#11-新设备继续老项目流程)。

**自动化方式（推荐）**：安装好 Claude Code 和 `/con-dev` skill 后，直接运行：

```
/con-dev +<项目名>
```

skill 会自动完成全部五阶段：`git pull` → 解析 YAML（含 `code_paths` / `sync`）→ clone 代码仓库 → **精准读取** handoff/plan → 环境检测 → 输出就绪报告。

**con-dev 上下文优化**（v3.3）：skill 采用精准读取策略，只加载冷启动必需内容，不读参考/历史数据。详见 § 9 skill 说明。

**手动快速路径（无 skill 时）：**
1. 确认 Jeff1993 已通过 Obsidian Sync 同步到新设备，然后 `git pull`
2. 运行 `python3 scripts/project_registry.py resolve <项目名>`，确认当前设备对应代码路径
3. 运行 `python3 scripts/project_registry.py sync-list <项目名> --summary`，确认大项目同步清单和 required 文件
4. 读 `.project-status.yaml` → 目标项目 `handoff.md`
5. 根据 `code_paths` / `code_path` 和 `repo` 字段 clone 代码仓库，运行验证命令

## 5. Checkpoint 协议入口

详见 [`onboarding.md § 6`](onboarding.md#6-checkpoint-规则) 和 [`agent-kit/claude/skills/checkpoint/SKILL.md`](agent-kit/claude/skills/checkpoint/SKILL.md)。

**要点**：
- 默认 commit 前缀：`[CHECKPOINT] YYYY-MM-DD <项目名>: <摘要>`
- 如配置了远程仓库（`origin`），安全检查通过后必须 `git push`
- dev-log 写入规则：先扫描当天文件是否已有同项目区块，有则合并，无则追加
- 若本次改了配置方法、服务器接入或 secret 引用，同步更新项目 `docs/runbooks/*.md`，并在 `progress.md` 写 `[RUNBOOK]` / `[SECRET-REF]` 证据。
- 若本次涉及联动契约变更（上游产物、下游消费、目录/schema、接口、指标、调度、配置引用），必须执行项目联动协同检查：识别上下游、更新 contract/runbook、运行最小联动验证，并在 progress/handoff 写证据。
- 若本次让 `handoff.md`、`plan.md` 或 `README.md` 变长，先执行 Doc Lifecycle Gate：旧完成记录归入 `progress/YYYY-MM.md`，旧计划/长评审归入 `archive/**`，入口只保留摘要和链接。
- **Memory Hooks**（v3.8）：checkpoint 前自动搜索云端记忆（Recall），checkpoint 后自动写入项目状态和 secret 引用到云端（Store），默认不写真实 secret payload。详见 `agent-kit/claude/skills/checkpoint/SKILL.md` 与本机安装副本 Step 2.5 + Step 5.7。

### dev-log 归档策略

当前按天一个文件（`dev-log/YYYY-MM-DD.md`）。文件积累超过 90 个后，按月迁移到子目录（如 `dev-log/2026-05/`）。

## 6. Secret 同步规则

详见 [`onboarding.md § 8`](onboarding.md#8-secret-同步风险确认)。

核心边界（2026-06-03 放宽）：私有仓允许明文 secret payload 进入 Git；`git ls-files` 命中仅**警告列清单**，不中止 checkpoint。`no_log_values` 仍生效（不打印值）。

## 7. 冲突恢复

详见 [`onboarding.md § 9`](onboarding.md#9-并发与冲突)。

高冲突文件：`collab-entry.md`、`onboarding.md`、项目 `plan/progress/handoff`、`.project-status.yaml`。

## 8. 接力完成标准

一次接力结束时，应能回答：

- 当前目标是什么？
- 已完成什么，有什么验证证据？
- 下一步是谁都能执行吗？
- 是否有阻塞或风险？
- 是否已经 checkpoint 并留下可追踪 commit/report？

## 9. Skill 清单 (跨工具)

由 [`agent-kit/`](agent-kit/) 维护, 跨设备通过 jeff1993-docs git 同步。

### Claude Code (skills/<name>/SKILL.md)

| skill | 作用 | 触发 |
|---|---|---|
| `con-dev` | 跨设备/新会话接力（精准读取，省 ~9K tokens） | `/con-dev +<项目>` |
| `checkpoint` | 收工（编排 3 子 skill：archive/memory/git） | `/checkpoint` |
| `checkpoint-archive` | 子 skill：Plan→Progress 归档 + Doc Lifecycle Gate | 被 checkpoint 编排 |
| `checkpoint-memory` | 子 skill：Memory Recall（可被 con-dev/agent-handoff 复用）+ Store | 被 checkpoint 编排 |
| `checkpoint-git` | 子 skill：Secret gate + Governance lint + 双仓 commit/push | 被 checkpoint 编排 |
| `init-onboarding` | 一次性接入大型附加层 | `/init-onboarding [--small\|--large\|--upgrade]` |
| `adr` | 起草决策记录 | `/adr [<slug>]` |
| `refresh-onboarding` | 中途轻量刷新 | `/refresh-onboarding` |
| `init-agent` | 新设备 Agent 初始化 | `/init-agent` |
| `deep-search` | 多源深度搜索（通用，需项目提供 search 模块协议） | `/deep-search <topic>` |
| `session-handoff` | 同设备**同 agent** session 续接（dry-run） | "切 session 继续" |
| `agent-handoff` | 同设备**异 agent** 交接（claude→codex 等，生成临时文件） | "交接给 codex/kimi/gemini" |
| `kdocs` | 金山文档 / WPS 云文档操作（kdocs-cli）：新建/读取/编辑/搜索/分享在线文档、AI PPT、PDF、知识库 | 任务涉及金山/WPS 云文档时自动触发 |

源: [`agent-kit/claude/skills/`](agent-kit/claude/skills/)

> **kdocs 依赖**：需 `kdocs-cli`（`node scripts/setup.cjs` / `powershell scripts/setup.ps1` 从 CDN 装到 `%LOCALAPPDATA%\kdocs-cli`，自动加 PATH）+ Token。Token 优先级：`--token` > `KINGSOFT_DOCS_TOKEN` 环境变量 > 系统密钥链（`kdocs-cli auth set-token`）。密钥链不可用时用 User 环境变量 `KINGSOFT_DOCS_TOKEN`（本机已配）。
>
> `KINGSOFT_DOCS_TOKEN`（明文，私有仓）：
> ```
> ntE7Ru4U3Kd87AVHYIeuo5fK9aPQ8AON9630ALox+tA7Ak6OYDAlyA1n9HIPVhWcAEJdV1UEo7Q3ob9BxVepIeLLXssUPC695yByQ5Mzd6fXayCbUJr53oN+pNnDzLsjtgRs4d/gK727ExFNQqW5LJy+leucyDkAbYGYy14X9ZbtOS7OOP4IomUjek9WpDHTFzlm3S25WEksXCsLtg==
> ```
同步工具: `python3 scripts/agent_kit_sync.py --apply`（manifest 驱动，跨平台，部署到 4 目标目录）
手动安装: `cp <src> ~/.agents/skills/<name>/SKILL.md`，若客户端不扫描通用目录，再回退到 `~/.claude/skills/<name>/SKILL.md`

### Codex (skills/<name>/SKILL.md)

行为与 Claude 同, 通过 prompt 触发。源: [`agent-kit/codex/skills/`](agent-kit/codex/skills/)

**Codex 双轨状态（v3.16 合并后）**：4 个近镜像 skill（adr/init-onboarding/refresh-onboarding/session-handoff）已压缩为指针（保留触发词 + 流程骨架 + 引用 Claude 版）；2 个 ext skill（checkpoint-onboarding-ext/con-dev-onboarding-ext）改为差异化段（保留 Codex 特有步骤，Memory/Gate/secret 改引用单一源，secret gate 已从 fail-closed 修正为警告对齐 Claude 版）。

**当前 Codex 优先走通用目录**：`~/.agents/skills/<name>/SKILL.md`。若当前运行时不扫描通用目录，再回退到目录式 `~/.codex/skills/<name>/SKILL.md`。根目录 `~/.codex/skills/<name>.md` 仅作源/通用 Markdown 复用, 不会出现在当前 Codex Skills 列表中。

本机安装: `mkdir -p ~/.agents/skills/<name> && cp <src>.md ~/.agents/skills/<name>/SKILL.md`；若验证发现当前 Codex 不扫描通用目录，再同步到 `~/.codex/skills/<name>/SKILL.md`（若源文件无 frontmatter, 安装脚本需补齐 `name` / `description`）。

清单和安装步骤见 [`agent-kit/codex/skills.md`](agent-kit/codex/skills.md)；详细约束见 [`onboarding.md § 7.1`](onboarding.md#71-codex-skill-安装格式强约束)。

### 其他 CLI (Gemini / Kimi / 通用)

通用 markdown skill 文件复用 codex 版本。通过项目 `AGENTS.md` 注入启动协议。
详见 [`MAPM/onboarding-suite/docs/adoption.md`](MAPM/onboarding-suite/docs/adoption.md)。
