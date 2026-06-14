# Codex 用户级配置

此文档记录当前桌面系统上 Codex 的用户级配置经验，尤其是默认 full access、全局规则文件、用户问题 skill 入口与系统级修复权限的边界。若需要同时初始化 Claude Code、opencode 等工具的全局提示词，读取 [agent-global-prompts.md](agent-global-prompts.md)。

## 配置位置

Codex 用户级配置文件通常在：

```bash
$HOME/.codex/config.toml
```

Codex 用户级全局规则文件通常在：

```bash
$HOME/.codex/AGENTS.md
```

查看当前配置：

```bash
sed -n '1,220p' "$HOME/.codex/config.toml"
sed -n '1,220p' "$HOME/.codex/AGENTS.md"
```

## 默认 full access

若希望 Codex 默认以 full access 文件系统权限启动，在 `$HOME/.codex/config.toml` 中设置：

```toml
sandbox_mode = "danger-full-access"
```

如果希望隐藏 full access 提示，可配置：

```toml
[notice]
hide_full_access_warning = true
```

可以在 TUI 状态栏显示当前权限：

```toml
[tui]
status_line = ["model-with-reasoning", "context-remaining", "five-hour-limit", "thread-title", "weekly-limit", "permissions"]
```

## 权限边界

`danger-full-access` 只表示 Codex 的文件系统沙箱不再限制普通用户可访问的路径；它不等于 root 权限，也不会绕过系统认证、Polkit、`pkexec`、`sudo`、磐石架构保护或维护模式要求。

涉及 `/usr`、`/etc`、`/opt`、系统包、系统服务、设备节点、分区、KSaf 策略等系统级修复时，仍必须先按主 skill 规则检查：

```bash
mm-cli -s
```

只有当前为维护模式时，才继续系统级修改。需要 root 的操作仍使用 `pkexec` 或 `sudo`，并在最终回复中说明执行过的权限动作。

## 常用检查

```bash
rg -n 'sandbox_mode|approval|approvals_reviewer|status_line|hide_full_access_warning' "$HOME/.codex/config.toml"
```

如果用户说“默认不是 full access”或“换目录后权限不对”，优先检查 `$HOME/.codex/config.toml` 是否是用户级配置，而不是只检查当前目录的 `AGENTS.md`。

## 全局用户问题 skill 初始化

如果希望 Codex 在任意工作目录处理 KylinOS Desktop V11、UKUI、KARE、Kaiming、Clash Verge、系统服务、开机自启动、TUN、维护模式、磐石架构等问题时，都默认读取用户问题 skill，可在 `$HOME/.codex/AGENTS.md` 写入全局规则。示例：

```markdown
# 全局 Codex 规则

## Kylin Desktop V11 经验沉淀

- 处理当前桌面操作系统相关问题时，默认使用 `$HOME/kylinos-desktop-v11-skill/SKILL.md` 作为经验入口。
- 开始处理 Kylin Desktop V11、UKUI、KARE、Kaiming、Clash Verge、系统服务、开机自启动、TUN、维护模式、系统保护/磐石架构等问题前，先读取该 `SKILL.md`，再按其中的引用文档继续读取对应 `references/*.md`。
- 问题确认修复完成后，必须主动判断是否产生新的可复用诊断步骤、修复步骤、风险点或系统特性；有则立即更新到该 skill 合集的 `SKILL.md` 或 `references/` 下的对应文档，不等用户追问。
- 最终回复中应说明本次经验是否已记录；如果已记录，给出对应 skill 文档路径；如果没有记录，说明原因是没有新增可复用经验。
- 经验文档应使用中文，避免写入当前用户专属路径、用户名或一次性状态；需要出现用户目录时使用 `$HOME`、`<user>`、`<app-id>`、`<desktop-id>` 等通用占位符。
- 桌面系统经验优先沉淀为可复用流程，不要把临时日志、长篇命令输出、聊天过程或只适用于单次现场的细节写入 skill。
- 涉及系统路径、服务、设备节点、维护模式、KARE/Kaiming 容器路径、UKUI 设置后端等内容时，记录“先诊断、再修改、最后验证”的最小闭环。
- 执行系统级修复前，必须先检查维护模式：运行 `mm-cli -s`，确认当前是 maintain mode 后才能继续修改 `/usr`、`/etc`、`/opt`、系统包、系统服务或设备节点。
- 如果当前不是维护模式，不要继续系统级修复。应让用户执行 `sudo mm-cli -o`，或经用户授权后执行 `pkexec mm-cli -o`；随后明确提醒用户重启系统，重启后重新打开 Codex 再继续修复。
- 切换维护模式并重启前，只允许做诊断、读取状态、模拟安装/卸载等非破坏操作；不要执行实际安装、卸载、写系统路径或改系统服务。
```

`$HOME/.codex/AGENTS.md` 是用户级全局规则，换目录启动 Codex 也应生效；项目目录下的 `AGENTS.md` 只对该目录及其子目录工作区更合适。若两者都存在，桌面系统经验入口应保持一致，避免一个指向 `$HOME/kylinos-desktop-v11-skill/SKILL.md`，另一个指向过期路径。

初始化检查：

```bash
test -f "$HOME/.codex/config.toml" && rg -n 'sandbox_mode|status_line|hide_full_access_warning' "$HOME/.codex/config.toml"
test -f "$HOME/.codex/AGENTS.md" && rg -n 'kylinos-desktop-v11-skill|mm-cli -s|磐石架构' "$HOME/.codex/AGENTS.md"
test -f "$HOME/kylinos-desktop-v11-skill/SKILL.md" && sed -n '1,80p' "$HOME/kylinos-desktop-v11-skill/SKILL.md"
```
