---
name: multicloud-ops
slug: multicloud-ops
displayName: Multicloud Ops 多云运维
version: "0.3.2"
summary: 统一管理九朵云和本地服务器，查询资源、状态与成本，执行运维操作；需配套 noc-ops CLI。
license: Apache-2.0
homepage: https://github.com/pholex/multicloud-ops
tags: [cloud, devops, multicloud, infrastructure]
description: 使用 ops CLI 管理多云资源和内网机器：查询状态、清单、域名证书与成本，执行受支持的实例操作，打开看板、排查出口连通性。当用户提到 noc、ops sync、云运维或这些日常运维任务时使用；依赖 noc-ops CLI。
metadata:
  version: "0.3.1"
  requires:
    bins: ["ops"]
---

# multicloud-ops — 多云运维助手

在 AI 客户端中管理云资源和内网机器：查状态与清单、查看成本、打开运维看板，以及执行受支持的实例操作。需要配套安装 `ops` CLI（noc-ops）。

支持 AWS、阿里云、腾讯云、火山引擎、Google Cloud、Microsoft Azure、Oracle Cloud、Cloudflare、Supabase，以及 VMware、深信服 HCI、群晖和 SSH 主机。各云能力不同，操作前按资源的 `capabilities` 判断。

## 使用示例

安装并配置好账号后，直接在 WorkBuddy、Codex、Claude Code 等客户端对 AI 说：

| 你可以这样问 | 会做什么 |
|---|---|
| 列出我的云服务器，标出停止运行的机器 | 读取已纳管清单，说明数据时间；需要时刷新 |
| 打开运维看板 | 启动本地看板，查看资源、成本与操作日志 |
| 检查为什么 AWS 账号连接失败 | 只读检查配置、CLI 和身份访问，给出下一步 |

这些示例中的自然语言发给 AI 客户端即可，无需在终端运行 `ops "问题"`。

## 安装与初始化

### 1. 检查或安装工具

先运行 `ops --version`。如果能显示版本，跳过安装；如果未安装，macOS / Linux 可运行：

```bash
curl -fsSL https://raw.githubusercontent.com/pholex/noc-ops/main/scripts/install.sh | sh
```

安装脚本检查并补齐缺失工具，保留已安装的工具和已有配置。只需安装、登录实际使用的云 CLI。

### 2. 配置账号

```bash
ops setup   # 接入本机客户端，生成完整版 skill
ops init    # 按提示选择所用云，填写账号配置
```

仅添加某朵云可用 `ops init --cloud aws` 等选项，已有其它账号、备注和计划任务会保留。Cloudflare / Supabase 使用环境变量或 Keychain 保存 Token，配置文件只写 `token_env` 变量名。

### 3. 按场景同步

| 场景 | 命令 | 前提 |
|---|---|---|
| 云资源 | `ops sync` | 已配置账号并完成相应云的登录 |
| 机房或局域网机器 | `ops sync --local` | 已配置本地机器，并接入目标内网或 VPN |
| Tailscale 机器 | `ops sync --tailscale` | 已配置目标并具备网络连通性 |

按实际场景选择，不必依次运行全部命令。同步会生成清单并刷新 SSH 配置；完成后可运行 `ops noc` 打开看板（默认 `http://127.0.0.1:8765`）。

账号配置与清单保存在本机工作区，默认 `~/.noc`。`ops setup` 会将这份引导版替换为含各云速查和 runbook 指引的完整版。

## 常见问题

**必须配置九朵云、安装全部云 CLI 吗？**  不必，只配置自己使用的云。`ops doctor` 会跳过未配置的云；内网机器按实际网络和连接方式配置。

**为什么装了 skill 还需要 ops？**  Skill 告诉 AI 如何完成任务，`ops` 负责确定性的扫描、同步、看板和实例操作。缺少它时先完成安装，不能直接绕过它手动运维。

**刷新成本或额度会收费吗？**  可能。AWS 成本及额度中的 credits 抵扣查询会调用收费的 Cost Explorer；GCP 成本查询使用 BigQuery，也可能计费。其它云按对应接口规则判断。优先读取已有数据，按需限定账号和类型刷新，不频繁全量查询。

**查不到资源，是否说明它已经被删除？**  不一定，也可能未纳管、无权限或采集失败。先核对账号、区域与采集状态；查询失败不能解释为资源为空或花费为零。

**怎么更新？skill 和 ops 的版本必须相同吗？**  `ops update --check` 查稳定版，`ops update --dry-run` 预览；用户确认后用 `ops update --yes` 更新并刷新 skill。保留原安装来源、extra、扩展与版本约束，不自行把本地或固定来源切到 PyPI。手动更新程序后运行 `ops setup`。引导版顶层 `version` 是 SkillHub 包版本，`metadata.version` 是配套 noc-ops 版本；完整版记录生成时的 noc-ops 版本，两种版本号不必相同。

**遇到报错怎么办？**

| 现象 | 下一步 |
|---|---|
| `ops: command not found` | 确认已安装；检查 `~/.local/bin` 是否在 PATH，必要时 `export PATH="$HOME/.local/bin:$PATH"` |
| 某云 CLI 缺失 | 按安装脚本提示补齐所用云的工具；`ops sync` 会跳过缺 CLI 的云 |
| 凭证或访问报错 | 按该云登录指引配置，再运行 `ops doctor --cloud <云>`；AWS 用配置的 profile，腾讯云和火山引擎用默认 CLI 账号 |
| 完整版 skill 未安装或版本落后 | 运行 `ops setup` 重新生成 |
| `ops "问题"` 提示缺少扩展包 | 公开包不含终端自然语言入口；在 AI 客户端里直接提问 |

`ops doctor` 检查已配置账号及所需依赖，只读验证身份、Token 或基础 API 访问；通过不代表所有资源权限均可用。`--offline` 只检查本机配置和依赖、不验证云凭证，`--json` 输出结构化结果。仍无法解决时，说「帮我整理问题反馈」，见文末入口。

## AI 执行规则

### 工具与目标

- 动手前先运行 `ops --version`。命令不可用时给出安装指引并等待用户安装，不自行改用云 CLI 绕过前置条件。
- 使用 `ops` 的确定性子命令。`ops "<自然语言>"` 依赖可选扩展，AI 客户端中不要再调用它套一层 LLM。
- 先用 `ops find <目标> --kind instances|databases|… --cloud <云> --account <账号> --region <区域> --json` 定位；按返回的 `capabilities` 选择操作，`noc://…` 引用可传给 CLI/MCP。重复 ID、同名或区域不明时收窄范围，不能取第一条或猜测对象。
- 操作前回显目标的名称、ID、云、账号与区域。`ops reboot`、`ops db` 支持相同筛选参数；`--dry-run --json` 不读取凭证、不调用云 API。

### 数据与常用命令

先读工作区 `inventory.md` 或 `data.json`。检查 `collection_status`，按云、账号、资源类型引用 `last_success_at`（AWS 实例再分区域）。`error` 表示采集失败、旧数据沿用，`partial` 表示采集不完整；缺时间则说“尚无记录”，不能以 `generated_at` 代替各资源采集时间。只有成功的空查询才清空对应范围；Azure 成本跨订阅共享计费档案，按整批处理。

| 任务 | 入口 |
|---|---|
| 刷新资源 | `ops sync --kind=<类型> --account=<账号>`；类型包括 instances、domains、databases、storage、serverless、cdn、iam、billing、credits 等，以该云支持范围为准 |
| 查实例实时状态 / 重启 | `ops reboot <目标> --status` / `ops reboot <目标>`；重启后等待并复查状态 |
| 看板 | `ops noc`，默认后台；用 `ops noc status`、`ops noc stop`、`ops noc restart` 管理 |
| 出口 IP / 目标连通性 | `ops netprobe` / `ops netprobe --target <host:port>` |
| 桶内只读明细 | AWS `aws s3`、腾讯云 `coscli ls`、火山引擎 `tosutil ls`；tccli / ve 不覆盖桶内对象查询 |
| 定时重启 | 使用已接入的 AWS EventBridge 云端调度；按工具与 runbook 支持的云和账号执行，不能将所有云视为均支持 |

新增账号后按所用场景重扫。清单覆盖不到的查询可用对应云 CLI（aws / aliyun / tccli / ve / gcloud / az / oci）的只读命令。完成新任务后可提议沉淀为 runbook，用户同意后保存。

### 操作边界

- 只读操作自动执行；**用户要求的实例重启免二次确认**。GCP 重启是硬 reset。其它写操作（停机、改配置、删资源、SSH 改动）先回显计划，用户确认后执行。
- Supabase 暂停/恢复先展示 `--dry-run`，确认后传 `--yes`；MCP 计划任务创建/取消、SSH 写操作确认后才传 `confirm=true`。实时状态查询失败时不继续数据库写操作。
- 写操作完成后记审计；`ops reboot` 自带日志，不重复记录，其它未自动记录的写操作用 `ops log <action> <target> --source <你的客户端名>`。
- 若装有 AWS Agent Toolkit 等官方云技能，纳管资产的日常查询、实例操作和同步使用本 skill / `ops`；架构建设、深度排障和成本分析可用官方技能，仍遵守上述写操作边界。官方技能非必需，安装入口见 [AWS Agent Toolkit](https://github.com/aws/agent-toolkit-for-aws)。

## 各云注意事项

### Cloudflare

Containers、D1、DNS zone、R2、Workers、代理 zone CDN 和按量成本只读纳管。Containers 无 SSH，不支持 `ops reboot`；D1 只读元数据，不执行 SQL。

- Containers / D1：用 `ops sync --account=<标签>` 搭配 `--kind=instances` 或 `--kind=databases`，分别需 `Workers Containers Read` / `D1 Read`；账号发现需 `Account Settings Read`。
- 成本：`ops sync --kind=billing --account=<标签>`，使用 Billable Usage v1，需 `Billing Read` 和账号发现权限。近三个月起始的账期按产品、实际账期起始时间和原币种汇总，每日更新。首页单独展示最近账期费用及起始日期，不计入自然月成本；旧汇总刷新后显示实际账期。
- 按量费用不含固定订阅费、税费和历史调整，不是完整账单。查询失败或账号未覆盖时保留旧数据。

### Supabase

托管 Postgres、Storage、Edge Functions 只读纳管；项目暂停/恢复是支持的写操作。`ops db status <名字|ref>` 只读，`ops db pause|resume <名字|ref>` 按上述确认流程执行，并使用 `--source <你的客户端名>`。Token 存环境变量或 Keychain，不写入 TOML。

### Google Cloud 成本

需开启 Cloud Billing 标准或详细 BigQuery 导出，并配置 `billing_export_table`（project.dataset.table）及 `billing_location`。沿用 gcloud 身份，需要数据集 dataViewer 和查询项目 jobUser。

默认单次查询上限 1 GiB，仅显式 `ops sync --account=<项目ID> --kind=billing` 或成本页刷新时查询，可能计费。近三个月按项目、服务、UTC 使用月份汇总 cost + credits，保留原币种；未配置或失败不能解释为零花费。

### CDN 与本地机器

- CDN 覆盖 CloudFront、Google Cloud CDN 后端、阿里云 / 腾讯云 / 火山引擎 CDN、Azure Front Door、Oracle Media Streams CDN 和 Cloudflare 代理 zone。Oracle 仅支持 Media Streams 内的 OCI Edge / Akamai 视频分发配置。
- 本地机器需先接入目标内网、VPN 或 Tailscale；连不上不能直接断言机器故障。操作能力以具体资源为准，不能把深信服 HCI、群晖或 SSH 主机等同于支持 `ops reboot` 的云实例。

## 问题反馈

安装、使用问题或功能建议，到 [SkillHub 技能页](https://skillhub.cn/skills/indiv-xy/multicloud-ops)的「评论」留言，也可对 AI 说「帮我整理问题反馈」。

AI 优先使用当前对话信息，必要时只读检查系统和 `ops --version`；整理系统、客户端、ops 与 skill 版本、步骤、预期结果、实际现象及必要报错。未知项注明未知，不要求先修好工具才能反馈。

草稿须隐去 Token、密码、私钥、真实账号与资源标识、内网地址和个人路径，不附完整配置、清单或原始日志。交给用户检查并自行提交，不自动发布评论，也不为收集反馈重做重启、停机等写操作。

本 skill 以 Apache License 2.0 发布，与 noc-ops CLI 同许可；许可证与版权说明见 `LICENSE.md`、`NOTICE.md`。
