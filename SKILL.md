---
name: kylinos-desktop-v11-skill
description: 处理 KylinOS Desktop V11 桌面系统上的应用兼容、系统服务、网络代理、KARE 打包应用、UKUI 桌面环境和常见系统集成问题。适用于 Clash Verge Rev TUN 模式安装失败、/dev/net/tun 缺失、clash-verge-service 安装失败、KARE 应用路径异常、代理核心 verge-mihomo 丢失或路径不一致、UKUI 开机自启动项不生效或设置界面不显示等麒麟桌面 V11 维护场景。
---

# KylinOS Desktop V11

这是 KylinOS Desktop V11 桌面环境问题的技能合集。先识别问题所属模块，再读取对应参考文档。

## 使用原则

- 先诊断，再改系统。读取日志、服务状态、设备节点、应用实际路径后再执行提权命令。
- 不要删除、移动或覆盖用户已有的可执行文件、配置文件、订阅文件、代理核心或 systemd 单元，除非用户明确要求。
- 每次问题确认修复完成后，必须主动把新增的可复用诊断步骤、修复步骤、风险点或系统特性记录到本 skill 的 `SKILL.md` 或对应 `references/` 文档，并在最终回复中说明记录位置。
- 涉及 `/usr`、`/etc`、`/opt`、`/dev`、systemd、设备节点、网络路由、代理核心复制、系统包安装/卸载等系统级修复时，必须先检查维护模式并说明影响。
- 系统级修复前运行 `mm-cli -s`；只有确认当前是 maintain mode 才能继续实际修改系统路径、系统服务或系统包。
- 如果当前不是维护模式，不要继续系统级修复。应让用户执行 `sudo mm-cli -o`，或经用户授权后执行 `pkexec mm-cli -o`；随后提醒用户重启系统，重启后重新打开 Codex 再继续问题修复。
- 通过命令行安装程序前，注意 Kylin Desktop V11 可能会被系统保护/磐石架构阻拦。需要先切换到维护模式，安装完成后再退出维护模式。
- KARE 打包应用可能存在宿主机路径和 KARE 内部路径不一致的问题；不要假设应用内看到的路径就是宿主机真实路径。
- 优先保留用户已经恢复正常的工作状态。遇到“之前手动修复过”的路径，要先验证再决定是否改动。
- UKUI 设置界面显示的开机自启动项不完全等同于 `~/.config/autostart` 文件列表；它还可能受 `org.ukui.control-center` 的 `autoapp-list` 和 `sort-app-list` 过滤。
- UKUI 开机自启动项不在设置界面显示时，优先检查 `.desktop` 的 `Icon=` 是否能在宿主系统解析到；KARE/Kaiming 应用常见问题是图标路径指向应用内部路径。

## 参考文档

- Clash Verge Rev TUN 模式、`clash-verge-service`、`/dev/net/tun`、`verge-mihomo` 路径问题：读取 [references/clash-verge-tun.md](references/clash-verge-tun.md)。
- UKUI 开机自启动不生效、设置界面不显示新增启动项：读取 [references/ukui-autostart.md](references/ukui-autostart.md)。
- 任务栏/托盘 AI 助手或 AI 子系统卸载、`kylin-ai-runtime`、Kaiming AI 助手残留：读取 [references/kylin-ai-subsystem.md](references/kylin-ai-subsystem.md)。
- 根分区、DATA 分区、`/home` 挂载、磐石架构/ostree/overlay/KARE 合并视图、空间占用判断：读取 [references/storage-layout.md](references/storage-layout.md)。
- Codex 用户级配置、默认 full access、权限显示和与维护模式/root 权限的边界：读取 [references/codex-config.md](references/codex-config.md)。
- Codex、Claude Code、opencode 等多工具全局提示词入口、用户问题 skill 路由和渐进式披露加载模板：读取 [references/agent-global-prompts.md](references/agent-global-prompts.md)。

## 通用检查

处理 Kylin/KARE 应用问题时，先收集这些信息：

```bash
uname -a
id
command -v <应用命令> || true
ps -ef | rg -i '<应用关键字>' | rg -v rg || true
systemctl status <服务名> --no-pager
```

KARE 应用常见路径：

```bash
/opt/kare/usr/bin
/opt/kare-applications/shadow/upper/usr/bin
/opt/kare-applications/shadow/merge/usr/bin
```

如果问题涉及应用日志，优先从用户目录下的 `~/.local/share`、`~/.config`、`~/.cache` 查找实际应用数据目录。

## 命令行安装与维护模式

在 Kylin Desktop V11 上通过命令行安装程序、写入系统路径或修改系统服务前，先检查当前模式：

```bash
mm-cli -s
```

只有输出表明当前是 maintain mode 时，才继续实际系统级修改。如果当前是 normal mode，不要继续修复；先进入维护模式：

```bash
sudo mm-cli -o
```

或由 Codex 在用户授权后执行：

```bash
pkexec mm-cli -o
```

执行后需要重启系统，重启后重新打开 Codex 再继续系统级修复。

完成安装或系统修改后，退出维护模式并保存修改：

```bash
mm-cli -c -a
```

退出维护模式后通常也需要重启系统。不要把系统长期留在维护模式。执行安装前后都应向用户说明当前模式切换、重启要求和影响。
