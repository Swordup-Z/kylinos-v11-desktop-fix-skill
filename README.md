# kylinos-v11-desktop-fix-skill

[English](README.en.md)

这是一个面向 KylinOS Desktop V11 的系统修复经验库，用来沉淀桌面系统已有能力异常、失效、报错、不能持久化、安装失败或系统服务损坏时的可复用修复流程。它不是可执行程序，也不绑定某个工具的内置目录；入口文件为 `$HOME/.os-fix-skill/SKILL.md`，详细内容按 `references/` 和 `knowledge/` 渐进展开。

内容覆盖 UKUI、KARE/Kaiming、Clash Verge TUN、应用安装、aTrust/UEM 安全客户端、开机自启动、全局搜索、托盘、锁屏超时、输入法、系统服务、维护模式、磐石架构、指纹/图形硬件、分区挂载和桌面 AI 子系统残留等场景。

系统功能增强、本地客制化、默认行为调整和源码级功能新增由功能增强经验库维护。

## 先安装一个 AI 编程工具

如果还没有可用的 AI 编程工具，先安装 Codex、Claude Code 或 opencode 之一。下面命令面向 KylinOS Desktop V11 这类 Linux 桌面终端；更多安装方式见对应官方文档。

### Codex

官方文档：

- Codex CLI：https://developers.openai.com/codex/cli
- Codex 快速开始：https://developers.openai.com/codex/quickstart

推荐安装命令：

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

安装后启动：

```bash
codex
```

### Claude Code

官方文档：

- Claude Code 快速开始：https://docs.anthropic.com/en/docs/claude-code/quickstart
- Claude Code 设置：https://docs.anthropic.com/en/docs/claude-code/setup

推荐安装命令：

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

安装后启动：

```bash
claude
```

### opencode

官方文档：

- opencode 文档：https://opencode.ai/docs/
- opencode 下载：https://opencode.ai/download

推荐安装命令：

```bash
curl -fsSL https://opencode.ai/install | bash
```

安装后启动：

```bash
opencode
```

如果系统策略不允许 `curl | sh` 或 `curl | bash`，打开上面的官方页面，按当前 Linux 桌面环境选择独立安装包、npm 或其他受信任的安装方式。

## 安装本经验库

### 复制给 AI 工具执行安装

把下面整段复制给 Codex、Claude Code 或 opencode，让 AI 工具代为安装并接入用户级规则文件：

```text
请帮我在本机安装 KylinOS Desktop V11 系统修复经验库 kylinos-v11-desktop-fix-skill，并接入当前 AI 工具的用户级规则文件。

执行要求：
1. 先检查 git 是否可用。
2. 如果 $HOME/.os-fix-skill 不存在，执行：
   git clone https://github.com/Swordup-Z/kylinos-v11-desktop-fix-skill.git "$HOME/.os-fix-skill"
3. 如果 $HOME/.os-fix-skill 已存在且是 git 仓库，只允许执行非破坏更新：
   git -C "$HOME/.os-fix-skill" status -sb
   git -C "$HOME/.os-fix-skill" fetch --prune
   git -C "$HOME/.os-fix-skill" pull --ff-only
   如果存在本地改动、分支分叉或 fast-forward 失败，不要覆盖，直接说明原因。
4. 如果 $HOME/.os-fix-skill 已存在但不是 git 仓库，不要删除或覆盖，直接停止并说明原因。
5. 确认 $HOME/.os-fix-skill/SKILL.md 存在。
6. 必要时创建当前 AI 工具用户级规则文件的父目录。
7. 将下面规则追加到当前 AI 工具的用户级规则文件；如果无法判断当前工具，则优先询问我。不要覆盖已有内容，已有同等规则时不要重复追加。

规则文件位置：
- Codex: $HOME/.codex/AGENTS.md
- Claude Code: $HOME/.claude/CLAUDE.md
- opencode: $HOME/.config/opencode/AGENTS.md

需要追加的规则：
当用户处理 KylinOS Desktop V11、UKUI、KARE、Kaiming、Clash Verge、系统服务、开机自启动、TUN、维护模式、磐石架构、系统保护、分区、挂载、overlay、AI 子系统等桌面操作系统问题，且现象属于已有能力异常、失效、报错、不能持久化、安装失败或系统服务损坏时，默认使用 $HOME/.os-fix-skill/SKILL.md 作为经验入口。开始处理系统修复问题前，先读取 $HOME/.os-fix-skill/SKILL.md，再按其中“参考文档路由”选择性读取 references/<scenario>.md；随后只读取该 reference 指向的 knowledge/<scenario>/README.md，以及与当前现象匹配的一个具体 knowledge 章节。如果没有命中具体 reference 或 knowledge 章节，不要遍历整个 skill；只有问题明确属于维护模式、系统保护、systemd/D-Bus 基础服务或系统体检噪声时，才读取 $HOME/.os-fix-skill/references/system.md。处理问题时遵循“先诊断、再修改、最后验证”；涉及 /usr、/etc、/opt、系统包、系统服务、设备节点、分区、KSaf 策略等系统级修复前，必须先运行 mm-cli -s 检查维护模式，只有确认当前是 maintain mode 才允许实际修改系统路径、系统服务或系统包。普通使用者确认产生新增可复用修复经验时，不要直接修改或提交 $HOME/.os-fix-skill；应生成 $HOME/.os-fix-skill-patches/ 下的本地 patch，保持 skill 仓库可继续 git pull --ff-only。只有显式进入开发者维护模式、确认 $HOME/.os-fix-skill 的 osSkill.maintainer=true 且 dry-run push 通过时，才按当前开发环境规则直接维护仓库。

完成后请告诉我仓库路径、已更新的规则文件路径，以及是否因为已有本地改动或分支状态跳过了更新。
```

### 手动安装

先克隆或更新本经验库。已有目录时不要覆盖本地改动；如果 `git pull --ff-only` 失败，先处理分支状态再继续。

```bash
if [ ! -d "$HOME/.os-fix-skill/.git" ]; then
  git clone https://github.com/Swordup-Z/kylinos-v11-desktop-fix-skill.git "$HOME/.os-fix-skill"
else
  git -C "$HOME/.os-fix-skill" status -sb
  git -C "$HOME/.os-fix-skill" fetch --prune
  git -C "$HOME/.os-fix-skill" pull --ff-only
fi
```

入口文件是：

```text
$HOME/.os-fix-skill/SKILL.md
```

选择当前 AI 工具的用户级规则文件，必要时先创建父目录：

```text
Codex:       $HOME/.codex/AGENTS.md
Claude Code: $HOME/.claude/CLAUDE.md
opencode:    $HOME/.config/opencode/AGENTS.md
```

例如 Codex：

```bash
mkdir -p "$HOME/.codex"
${EDITOR:-vi} "$HOME/.codex/AGENTS.md"
```

将下面规则追加到对应规则文件末尾。不要覆盖已有内容；如果已有等价规则，不要重复追加。

```text
当用户处理 KylinOS Desktop V11、UKUI、KARE、Kaiming、Clash Verge、系统服务、开机自启动、TUN、维护模式、磐石架构、系统保护、分区、挂载、overlay、AI 子系统等桌面操作系统问题，且现象属于已有能力异常、失效、报错、不能持久化、安装失败或系统服务损坏时，默认使用 $HOME/.os-fix-skill/SKILL.md 作为经验入口。

开始处理系统修复问题前，先读取 $HOME/.os-fix-skill/SKILL.md，再按其中“参考文档路由”选择性读取 references/<scenario>.md；随后只读取该 reference 指向的 knowledge/<scenario>/README.md，以及与当前现象匹配的一个具体 knowledge 章节。如果没有命中具体 reference 或 knowledge 章节，不要遍历整个 skill；只有问题明确属于维护模式、系统保护、systemd/D-Bus 基础服务或系统体检噪声时，才读取 $HOME/.os-fix-skill/references/system.md。

处理问题时遵循“先诊断、再修改、最后验证”；涉及 /usr、/etc、/opt、系统包、系统服务、设备节点、分区、KSaf 策略等系统级修复前，必须先运行 mm-cli -s 检查维护模式，只有确认当前是 maintain mode 才允许实际修改系统路径、系统服务或系统包。

普通使用者确认产生新增可复用修复经验时，不要直接修改或提交 $HOME/.os-fix-skill；应生成 $HOME/.os-fix-skill-patches/ 下的本地 patch，保持 skill 仓库可继续 git pull --ff-only。只有显式进入开发者维护模式、确认 $HOME/.os-fix-skill 的 osSkill.maintainer=true 且 dry-run push 通过时，才按当前开发环境规则直接维护仓库。
```

配置完成后，KylinOS Desktop V11 桌面系统修复问题可从 `$HOME/.os-fix-skill/SKILL.md` 进入，再按其中的 `references/` 路由继续查阅。功能增强、本地客制化和默认行为调整由 `$HOME/.os-enhance-skill` 维护。

普通使用者的本地新增经验建议保存到 `$HOME/.os-fix-skill-patches/`，而不是直接提交到 `$HOME/.os-fix-skill`。这样 skill 仓库可以持续 `git pull --ff-only` 拉取上游更新；开发者维护仓库前可运行下面的可选检查，只有检查通过并写入本地维护者标记后，才把 patch 内容整理进 `references/` 或 `knowledge/` 并按本机全局规则维护仓库。

```bash
skill_dir="$HOME/.os-fix-skill"; if git -C "$skill_dir" push --dry-run --porcelain origin HEAD:refs/heads/main >/dev/null 2>&1; then git -C "$skill_dir" config --local osSkill.maintainer true; else git -C "$skill_dir" config --local --unset osSkill.maintainer 2>/dev/null || true; fi
```

系统维护使用固定会话名，例如 `os-fix`。之后遇到系统问题时，恢复同一个会话继续处理：

```bash
codex resume os-fix
claude resume os-fix
opencode resume os-fix
```

## 整体架构

本仓库面向系统修复场景：系统已有能力异常、失效、报错、不能持久化。例如 TUN 模式失败、开机自启动不生效、托盘隐藏状态丢失、指纹设备断开、系统服务异常。

修复内容按场景分类：

```text
system
applications
ukui
network
hardware
storage
agent-tools
source-rebuild
```

## 目录结构

```text
$HOME/.os-fix-skill/
├── SKILL.md
├── references/
│   ├── README.md
│   ├── system.md
│   ├── applications.md
│   ├── ukui.md
│   ├── network.md
│   ├── hardware.md
│   ├── storage.md
│   ├── agent-tools.md
│   └── source-rebuild.md
├── knowledge/
│   ├── README.md
│   ├── system/
│   ├── applications/
│   ├── ukui/
│   ├── network/
│   ├── hardware/
│   ├── storage/
│   ├── agent-tools/
│   └── source-rebuild/
├── scripts/
│   └── cleanup-kylin-ai.sh
├── README.md
└── README.en.md
```

`references/` 是场景入口和策略路由层，包含适用场景、简短说明、知识入口和最小诊断。`knowledge/<scenario>/README.md` 是场景内索引，负责把问题继续路由到具体章节。具体 `<topic>.md` 保存背景、诊断、修复步骤、验证、回滚和清理规则。通过源码修改实现的可复用修复还会在同场景 `patches/<fix-id>/` 下保存 patch 集和 `PATCHSET.md` 元数据。

固定加载链路：

```text
修复需求
-> 实际场景 reference
-> 场景 knowledge README
-> 具体细分 knowledge
```

## 路由示例

Clash Verge TUN 模式失败：

```text
SKILL.md
-> references/network.md
-> knowledge/network/README.md
-> knowledge/network/tun-device.md 或 knowledge/network/clash-verge-service.md
```

UKUI 全局搜索显示软件商店未安装应用：

```text
SKILL.md
-> references/ukui.md
-> knowledge/ukui/README.md
-> knowledge/ukui/search.md
```

给 UKUI 全局搜索增加自定义命令面板、接入共享经验库等增强任务，可从 `$HOME/.os-enhance-skill/SKILL.md` 进入。

## 当前覆盖范围

- 维护模式、磐石架构、系统级修改边界。
- 应用安装、AppImage、第三方 apt 源、KARE/Kaiming 隔离。
- aTrust/UEM 安全客户端、虚拟网络组件和第三方内核模块残留。
- Clash Verge TUN、`/dev/net/tun`、代理服务和代理核心路径。
- UKUI 开机自启动、全局搜索异常、快捷键、系统托盘、锁屏超时、输入法、面板/任务栏、打开方式文件选择器。
- 桌面 AI 组件、AI 子系统和残留清理。
- 指纹/生物识别、图形频率、硬件稳定性。
- 根分区、DATA 分区、`/home` 挂载、overlay、Kaiming/KARE 和 ostree 空间占用判断。

## 安全边界

KylinOS Desktop V11 上的系统级修改通常需要维护模式。涉及 `/usr`、`/etc`、`/opt`、系统包、systemd、设备节点、分区、KSaf 或系统服务时，应先确认维护模式：

```bash
mm-cli -s
```

进入维护模式：

```bash
sudo mm-cli -o
```

退出维护模式并保存：

```bash
sudo mm-cli -c -a
```

切换维护模式后通常需要重启。具体操作流程以 `SKILL.md` 和对应 reference/knowledge 章节为准。

## 独立工具

空间清理、Kaiming/KARE 应用层治理、ostree 占用审计这类任务适合由独立应用承载。若本机存在独立开发工作区，可从对应项目的规则文件进入：

```text
$HOME/desktop-develop/kylin-space-guard/AGENTS.md
```

本仓库记录系统诊断、安全边界和可复用修复经验；具体 UI、构建、验证、依赖选择和项目实现规则应维护在独立项目内。工具类项目仍必须遵守系统安全边界：不得自动删除 ostree deployment、EFI、GRUB、loader entries、`/etc/fstab` 或分区表。

如果使用本机的开发工作区，入口通常是：

```text
$HOME/desktop-develop/AGENTS.md
```

该文件只做项目路由；具体项目继续读取各自目录下的 `AGENTS.md`。

## License

当前仓库使用 MIT License，见 [LICENSE](LICENSE)。
