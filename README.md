# multicloud-ops —— 在 Claude Code / Codex 里，一句话运维你所有的云和机房

> An agent skill for day-to-day operations across AWS, Alibaba Cloud, Tencent Cloud, Volcengine, Google Cloud, Azure, Oracle Cloud, Cloudflare, Supabase and your own server room, in plain language from Claude Code, Codex, Gemini CLI or Kiro. The skill tells the agent what to call and what to ask you first; the scanning, dashboard, reboots and audit log are done by the companion CLI [`noc-ops`](https://github.com/pholex/noc-ops). Reads just run; changes are shown to you first. Nothing leaves your machine. macOS / Linux.

## 它是什么

装好之后，在你已经在用的 agent 客户端里直接说话：

- 「东京有几台机器在运行」
- 「重启 web-01」
- 「哪些域名的证书 30 天内到期」
- 「上个月各家云分别花了多少」
- 「新加了一台机器，同步一下清单」

只查看的事它直接做；会改东西的操作，它先说清要做什么，你点头才动。名字对上好几台机器时它会列出来让你选，不乱猜。

支持 AWS、阿里云、腾讯云、火山引擎、Google Cloud、Azure、Oracle Cloud、Cloudflare、Supabase，以及机房里的 VMware、深信服、群晖和任何能 SSH 的机器。各云支持到什么程度见 [noc-ops 的 README](https://github.com/pholex/noc-ops#支持哪些云)。

## 安装

**第 1 步：装 skill**，一行装进本机所有 agent 客户端（Claude Code / Codex / Gemini CLI / Kiro …）：

```bash
npx skills add pholex/multicloud-ops -g -y
```

**第 2 步：装配套的 `ops` 命令**（必需）：

```bash
curl -fsSL https://raw.githubusercontent.com/pholex/noc-ops/main/scripts/install.sh | sh
```

安装脚本先检查 uv、noc-ops 和各云 CLI；已安装的默认保留，缺失的云工具可按需选择安装，也可自行安装。用到哪家云，就配置哪家的凭证（`aws configure`、`aliyun configure`、`az login` 等），对照表在 [noc-ops 的 README](https://github.com/pholex/noc-ops#安装)。

**第 3 步：配置并同步**

```bash
ops setup           # 生成或刷新完整版 skill
ops init            # 一问一答，填上你的账号
ops sync            # 扫所有云，生成清单（有变化就再跑一次）
ops sync --local    # 只扫机房里的机器（人要在内网或 VPN 里）
```

看板：`ops noc`。哪步卡住，跑 `ops doctor`。

采集失败时保留对应范围的旧数据，成功查到零条才清空。看板「采集状态」显示各账号与资源类型的最近尝试、上次完整成功时间；AWS 实例按区域独立处理，Azure 成本按共享计费档案整体处理以免重复统计。Agent 会先检查采集状态，避免把旧清单当作实时状态。此行为由配套 noc-ops 实现，更新工具后运行 `ops setup` 刷新完整版 skill。

本仓是安装引导版，`metadata.version` 标注配套发行版本，配套版本为 `0.3.0`。运行 `ops setup` 后，会用 noc-ops 自带的完整版替换本机引导版，并记录生成时的 noc-ops 版本。

重复执行安装脚本默认保留已安装的工具和已有配置。更新 noc-ops 后，手动运行 `ops setup` 刷新完整版；旧 skill 没有版本记录时，安装检查会显示“版本未标注”。

## 初始化、定位和更新

`ops init` 支持九朵云，`--cloud azure` 可只添加 Azure 订阅；已有账号、备注和计划任务会保留，写入前自动备份。`ops doctor --offline` 检查本机，`ops doctor --cloud aws` 按配置验证 AWS 账号，未配置的云不会因为缺 CLI 而报错。

`ops find <目标> --cloud <云> --account <账号> --region <区域> --json` 返回完整资源引用与可用操作。CLI 和 MCP 使用一致的定位规则，同名或重复 ID 不会自动取第一条。操作前可用 `--dry-run --json` 预览。

`ops update --check` 查版本，`ops update --dry-run` 看步骤，`ops update` 确认后更新并刷新 skill，保留已有扩展和安装来源。详见 [noc-ops 发布与更新说明](https://github.com/pholex/noc-ops/blob/main/RELEASE.md)。

## 许可

Apache License 2.0。

## 贡献

提交前运行 `sh scripts/install-git-hooks.sh` 启用检查，并遵守 [公开内容检查](PUBLIC_CONTENT.md)。
