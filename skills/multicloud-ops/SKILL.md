---
name: multicloud-ops
slug: multicloud-ops
displayName: Multicloud Ops 多云运维
version: "0.3.1"
summary: 统一管理九朵云和本地服务器，查询资源、状态与成本，执行运维操作；需配套 noc-ops CLI。
license: Apache-2.0
homepage: https://github.com/pholex/multicloud-ops
tags: [cloud, devops, multicloud, infrastructure]
description: 多云运维助手（AWS / 阿里云 / 腾讯云 / 火山引擎 / Google Cloud / Microsoft Azure / Oracle Cloud / Cloudflare / Supabase）+ 内网机器（VMware / 深信服 HCI / 群晖 / 任何能 SSH 的主机）——查实例状态、启停/重启实例、同步清单 + SSH 配置、起运维看板、定时重启、探测本机出口 IP。需配套 `ops` CLI（noc-ops，见下方安装）。当用户提到 noc / ops sync / 云运维，或要查实例状态、重启机器、查域名证书、探测出口 IP 时使用本 skill。
metadata:
  version: "0.3.1"
  requires:
    bins: ["ops"]
---

# multicloud-ops — 多云运维助手

一个 skill 管 9 朵云：**AWS / 阿里云 / 腾讯云 / 火山引擎 / Google Cloud / Microsoft Azure / Oracle Cloud** 可查可操作（实例 / 域名 / 数据库 / 对象存储 / 函数 / CDN / IAM / 成本 / 额度；启停/重启，GCP 重启为硬 reset），**Cloudflare**（Containers / D1 / DNS zone / R2 / Workers / CDN / 按量成本）只读纳管，**Supabase**（托管 Postgres / Storage / Edge Functions）只读 + 项目暂停/恢复（需确认）；内网机器（VMware / 深信服 HCI / 群晖 / 任何能 SSH 的内网主机）在内网或 VPN 内也能纳管。能力随日常运维**沉淀成 runbook**，越用越省事。

---

## ⚠️ 前置：本 skill 依赖 `ops` CLI（noc-ops）

本 skill 是 `ops` CLI 的 agent 入口——确定性的扫描、同步、看板、重启都由 `ops` 完成。**没有装 `ops`，本 skill 无法工作。**

**先确认是否已安装：**
```bash
ops --version     # 打印 noc-ops 版本 = 已装好，直接往下用
```

**没装的话（macOS / Linux）：**
```bash
curl -fsSL https://raw.githubusercontent.com/pholex/noc-ops/main/scripts/install.sh | sh   # 检查 uv / noc-ops / 云 CLI，仅补齐缺失项
ops setup           # 生成或刷新完整版 skill，接入本机客户端
ops init            # 一问一答，填上用户的云账号（需安装并登录所用云的 CLI）
ops sync            # 扫所有云，生成清单
ops sync --local    # 只扫机房里的机器（需在内网或 VPN 里）
```

> `ops` 把代码和你的数据分离：云账号、实例清单、凭证都在**你本机的工作区**（默认 `~/.noc`），不外泄。
> `ops setup` 装的是本 skill 的**完整版**（含 runbook 索引与各云速查），会覆盖这份引导版。
> 本仓引导版通过 `metadata.version` 标注配套发行版本；完整版记录生成时的 noc-ops 版本。更新程序后，手动运行 `ops setup` 刷新 skill。重复执行安装脚本默认保留已安装的工具和已有配置。

---

## 给 AI 助手的执行规则（重要）

1. **动手前先确认 `ops` 可用**：运行 `ops --version`。
   - 若 `command not found` → **不要自己用 aws/tccli/aliyun/gcloud 手动运维**；告诉用户「本 skill 需要先安装 `ops` CLI」，给出上方安装步骤，停下等用户装好。
2. **`ops` 可用时，用它的确定性子命令，而不是再套一层 LLM**：
   - 查实例 / 域名 / 数据库 / 存储 / 函数 / CDN / IAM / 账务清单 → 先读工作区的 `inventory.md`（人读）或 `data.json`（机读）；过时就 `ops sync`（全量）或 `ops sync --kind=<instances|domains|databases|storage|serverless|cdn|iam|billing|credits|…>`（只刷一类），`ops sync --account=<账号>` 只刷一个账号。
   - 对象存储桶的只读明细 → AWS `aws s3`、腾讯云 `coscli ls`、火山引擎 `tosutil ls`（后两者是各自官方工具，tccli / ve 不覆盖桶）。
   - 查单台实例实时状态 → `ops reboot <实例名|ID> --status`（只读）。
   - 重启实例 → `ops reboot <实例名|ID>`（按 cloud 自动选对应云 API；`--dry-run` 只看计划）。
   - Cloudflare 按量成本 → `ops sync --kind=billing --account=<标签>`；使用 Billable Usage v1，需要 `Billing Read` 和账号发现权限。近三个月起始的账期按产品、实际账期起始时间、原币种汇总，每日更新；首页单独展示最近账期费用及起始日期，不计入自然月成本，旧汇总刷新后显示真实账期；不含固定订阅费、税费与历史调整，不是完整账单。查询失败或账号未覆盖时保留旧数据。
   - Cloudflare Containers 容器实例 / D1 数据库 → `ops sync --kind=instances --account=<标签>` / `ops sync --kind=databases --account=<标签>`；Containers 无 SSH，不支持 `ops reboot`，D1 只读元数据、不执行 SQL。token 分别需要 `Workers Containers Read` / `D1 Read`，账号发现需要 `Account Settings Read`。
   - Supabase 项目暂停 / 恢复 → `ops db status|pause|resume <名字|ref>`；pause/resume 是写操作：先 `--dry-run` 给用户看计划，同意后加 `--yes --source <你的客户端名>` 执行；status 只读。
   - 新增实例 / 账号后 → `ops sync`（重扫 + 刷 SSH 配置）；内网机器 `ops sync --local`；Tailscale 组网 `ops sync --tailscale`。
   - 运维看板 → `ops noc`（默认后台；`status` / `stop` / `restart` 管理）。
   - 本机出口 IP / 线路可达性 → `ops netprobe [--target host:port]`。
   - 清单覆盖不到的查询 → 直接用对应云 CLI（aws / aliyun / tccli / ve / gcloud / az / oci）只读命令。
   - `ops "<自然语言>"` 是给**终端用户**的入口，由可选扩展包提供、公开包不含；在 Claude Code / Codex 里**不要**调它，本 skill 就是这里的入口。
3. **安全**：只读自动执行；**实例重启免确认**（低风险可逆）；其它写操作（停机 / 改配置 / 删资源 / SSH 改动）**先回显计划、等用户确认**。写操作做完用 `ops log <action> [target] --source <你的客户端名>` 记一条审计。
4. **解析不到目标就停**：实例 / 区域找不到、有歧义时，如实回报并请用户补充，绝不猜测或对不确定对象动手。
5. **与 aws-*（AWS Agent Toolkit）等官方云技能的分工**：若同时装有 `aws-compute` / `aws-containers` / `aws-cdk` / `aws-serverless` 等——**纳管资产的日常运维**（查状态、启停/重启、清单同步）一律走本 skill / `ops`，即使意图字面上也命中 aws-* 的描述；aws-* 用于**建设 / 深度排障 / 成本分析**（写 CDK/CloudFormation、架构设计、账单优化），与本 skill 互补。不论走哪个技能，凡涉及纳管资产的**写操作**仍要过第 3 条闸门。

---

## 它能做什么（装好 `ops` 后）

- **查状态 / 清单 / 域名**：跨 9 朵云全量盘点——实例状态（含 Cloudflare Containers 的应用 / 镜像 / 规格）、IP、到期域名、数据库（含 Cloudflare D1 的容量 / 只读副本配置）、对象存储（含 COS / TOS / R2 / Supabase Storage）、函数（含 Workers / Edge Functions）、CDN（CloudFront、Google Cloud CDN 后端、阿里云 / 腾讯云 / 火山引擎 CDN、Azure Front Door、Oracle Media Streams CDN（OCI Edge / Akamai 视频分发配置）、Cloudflare 代理 zone）、IAM、AI 密钥元数据、消息队列、Cloudflare DNS zone（记录数 / 代理数 / SSL 模式 / 套餐 / Registrar 到期日）、Supabase 项目（算力 / 库占用 / PITR / 备份 / 网络限制）、各云账单与余额（含 Azure MCA 成本、Oracle 用量、GCP BigQuery 账单导出净成本、Cloudflare 按量费用），一屏看全。
- **重启 / 启停实例**：定位 → 对应云 API 执行 → 等待并验证状态。
- **初始化与体检**：`ops init --cloud <云>` 添加所用云账号，保留其它配置；九朵云均可配置。`ops doctor --cloud <云>` 按配置账号验证，`--offline` 只检查本机，`--json` 输出结构化结果。Cloudflare/Supabase 可用 `token_env` 引用环境变量或用 macOS Keychain，TOML 只存变量名。
- **定位与操作规则**：先 `ops find <目标> --kind instances|databases|… --cloud <云> --account <账号> --region <区域> --json`，按返回的 `capabilities` 使用支持的操作。结果的 `noc://…` 引用可传给 CLI/MCP；重复 ID/同名必须收窄，不能取第一条。`ops reboot`、`ops db` 支持相同筛选参数；`--dry-run --json` 不读取凭证、不调用云 API。重启延续免二次确认规则，数据库暂停/恢复确认后才传 `--yes`；MCP 计划任务创建/取消、SSH 写操作确认后才传 `confirm=true`。
- **更新**：`ops update --check` 查稳定版，`--dry-run` 离线预览；用户确认后用 `ops update --yes` 更新并刷新 skill。保留 uv 安装来源、extra、扩展与版本约束，不自行把固定/本地来源改成 PyPI。
- **GCP 成本接入**：需事先开启 Cloud Billing 标准或详细 BigQuery 导出，并在项目配置中设置 `billing_export_table`（project.dataset.table）及数据集位置 `billing_location`。沿用 gcloud 身份，需数据集 dataViewer 和查询项目 jobUser；默认单次查询上限 1 GiB，仅显式 `ops sync --account=<项目ID> --kind=billing` 或成本页刷新执行（可能计费）。按项目、服务、UTC 使用月份汇总近三个月 cost + credits，按原币种合计；未配置或查询失败不能解释为零花费。
- **同步纳管**：`ops sync` 自动扫描、配 SSH（含跳板）、刷新清单与看板。
- **判断数据是否新鲜**：查询清单前检查 `data.json` 的 `collection_status`。`status=error` 表示采集失败、已有数据沿用，不代表资源已删除；`partial` 表示列表已更新但补充字段不全。按云、账号、资源类型（AWS 实例再分区域）引用 `last_success_at`；缺失时说“尚无记录”，不能把 `generated_at` 当作所有资源的采集时间。只有成功完成的空查询才清空对应范围；Azure 成本跨订阅共享计费档案，按整批成本处理。
- **运维看板**：本地密集表格，多类资源分页 + 筛选 + 导出 CSV + 操作日志审计（`http://127.0.0.1:8765`）。
- **定时重启**：各云统一用 AWS EventBridge 云端调度，本机零常驻。
- **能力沉淀**：做完一项新运维即可存成 runbook，下次自动复用。

---

## 可选增强：AWS 官方 Agent Toolkit 技能

装好 `aws` CLI 后，**推荐（非必需）** 顺手安装 AWS 官方的 agent 技能包——`ops` 不依赖它们照常工作，但它们能补上本 skill 不覆盖的**建设 / 深度排障 / 成本分析**能力：

```bash
# Claude Code：装官方 aws-core 插件
/plugin install aws-core@claude-plugins-official

# 其它客户端：见 https://github.com/aws/agent-toolkit-for-aws
```

装好后按上方执行规则第 5 条分工使用，互不冲突。

---

## 故障速查

- `ops: command not found` → 未安装，或 `~/.local/bin` 不在 PATH：`export PATH="$HOME/.local/bin:$PATH"`。
- 某云 CLI 缺失 → 安装脚本会列出缺失工具并提供安装选择或指引，只需安装所用云的工具；`ops sync` 会跳过没装 CLI 的云。
- 凭证报错 → 按对应云的登录指引配置；`ops doctor` 当前检查部分云 CLI 和 AWS 默认凭证，不代表已验证所有云账号。
- skill 版本未标注或与 noc-ops 不一致 → 更新程序后运行 `ops setup`，重新生成完整版并记录版本。
- Cloudflare / Supabase 的 token → 存系统 Keychain 或环境变量，不进配置文件（见工作区 `ops-config.toml` 注释）。
- `ops "<自然语言>"` 提示「没有安装处理自然语言输入的扩展包」→ 这是预期：终端内的自然语言入口不在公开包里，在 Claude Code / Codex 等客户端里直接用本 skill 即可。

---

本 skill 以 Apache License 2.0 发布，与 `noc-ops` CLI 同许可。
