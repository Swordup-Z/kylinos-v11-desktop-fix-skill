---
name: os-fix-skill
description: 处理 KylinOS Desktop V11 桌面系统上的应用安装与兼容、系统服务、网络代理、KARE 打包应用、UKUI 桌面环境和常见系统集成问题。适用于系统级修复、维护模式、磐石架构、AppImage/libfuse、第三方 apt 源、Clash Verge Rev TUN 模式安装失败、/dev/net/tun 缺失、clash-verge-service 安装失败、KARE 应用路径异常、代理核心 verge-mihomo 丢失或路径不一致、UKUI 开机自启动项不生效或设置界面不显示、UKUI 全局搜索结果来源、需要重新编译系统源码包的修复评估、指纹/生物识别驱动恢复、图形驱动/devfreq 调频异常等麒麟桌面 V11 维护场景。
---

# KylinOS Desktop V11

这是 KylinOS Desktop V11 桌面系统问题修复经验库。只要用户问题属于“系统已有行为异常、失效、报错、不能持久化、安装失败或系统服务损坏”，就先读取本入口；再按实际场景读取 `references/<scenario>.md`。如果没有命中专门 reference，不要遍历整个 skill；只有明确属于维护模式、系统保护、systemd/D-Bus 基础服务或系统体检噪声时才读取 `references/system.md`。

## 使用原则

- 只要用户问题明确属于 KylinOS Desktop V11 桌面系统修复范围，就应使用本 skill；普通代码开发、文档编辑、Git 操作或与桌面系统无关的问题不应加载本 skill。
- 如果用户目标是系统功能增强、本地客制化、默认行为调整、AI 工具配置或源码级功能新增，切换到 `$HOME/.os-enhance-skill/SKILL.md`，不要在本仓库新增增强内容。
- 如果问题未命中某个专门 reference，只有明确属于维护模式、系统保护、systemd/D-Bus 基础服务或系统体检噪声时才读取 [references/system.md](references/system.md)；否则停止继续加载本 skill，先基于当前系统证据诊断或询问用户补充现象。
- 如果当前本地 skill 已读取但没有覆盖用户的系统问题，在自行探索修复前，先尽力同步上游 GitHub 仓库，查看是否已有新增经验。同步前确认 `$HOME/.os-fix-skill` 是 git 仓库且工作树无本地改动；只使用 `git fetch`、`git status -sb`、`git pull --ff-only` 这类非破坏流程。普通使用者的本地经验不得直接写入 skill 工作树；若发现本地改动，先保存为 `$HOME/.os-fix-skill-patches/` 下的 patch 并跳过同步，不要强制覆盖或清理。若网络不可用、分支分叉或 fast-forward 失败，不要把同步失败作为阻塞错误；继续基于当前资料、模型能力和通用系统维护经验诊断。
- 遵循渐进式披露：先读本入口判断修复场景；再只读取命中的 `references/<scenario>.md`；随后读取该 reference 指向的 `knowledge/<scenario>/README.md`；最后只读取与用户问题匹配的具体知识文件。不要把未命中的 reference、场景 README 或 knowledge 预加载进上下文；没有命中具体经验时，不要遍历整个 skill。
- `references/` 定位为系统修复场景分类的路由入口和快速诊断索引，场景文件直接放在该目录下，例如 `ukui.md`、`network.md`、`source-rebuild.md`。命名和路由规则见 [references/README.md](references/README.md)。
- `knowledge/` 定位为可复用的具体修复章节，场景目录直接放在该目录下，并与 references 使用相同场景分类。结构说明见 [knowledge/README.md](knowledge/README.md)。
- 新增经验分两种路径：普通使用者或无仓库写权限环境只生成 `$HOME/.os-fix-skill-patches/` 下的本地 patch，保持 skill 仓库干净以便拉取上游；开发者维护仓库时才直接更新最小匹配场景的 `references/` 或 `knowledge/<scenario>/` 章节，并按当前开发环境规则处理后续仓库维护。维护模式、安全边界、通用诊断闭环属于 `knowledge/system/`。
- 处理问题时遵循“先诊断、再修改、最后验证”。涉及系统路径、系统包、系统服务、设备节点、分区、KSaf 策略等系统级修改时，先按 [references/system.md](references/system.md) 和 `knowledge/system/maintenance.md` 确认维护模式与回滚边界。
- 保护用户现场：不要删除、移动或覆盖用户已有的可执行文件、配置文件、订阅文件、代理核心、systemd 单元或用户数据，除非用户明确要求且已验证影响。
- 每次问题确认修复完成后，必须主动判断是否产生可复用经验。普通使用者侧把建议变更保存为本地 patch 并在最终回复中说明 patch 路径；开发者侧才把经验写入对应 `references/` 路由或 `knowledge/` 章节。只有入口边界、触发范围或路由结构变化时，才更新 `SKILL.md`。
- 如果实际解决的是当前 skill 尚未覆盖的 KylinOS Desktop V11 系统修复问题，普通使用者侧生成包含新增 `references/<scenario>.md`、同场景 `knowledge/` 章节和必要 `SKILL.md` 路由变更的 patch；开发者侧可直接落库。

## 参考文档路由

按实际修复场景读取最小 reference；不要为了“保险”同时读取多个 reference。

用于“系统已有行为异常、失效、报错、不能持久化”的问题：

- 系统基础、维护模式、系统服务、体检噪声：读取 [references/system.md](references/system.md)。
- 应用安装失败、AppImage、KARE/Kaiming 隔离：读取 [references/applications.md](references/applications.md)。
- UKUI 自启动、全局搜索异常、托盘、快捷键、锁屏超时、AI 组件、面板/任务栏：读取 [references/ukui.md](references/ukui.md)。
- 代理、TUN、Clash Verge、网络弹窗：读取 [references/network.md](references/network.md)。
- 指纹、生物识别、图形频率、硬件稳定性：读取 [references/hardware.md](references/hardware.md)。
- 分区、挂载、DATA、`/home`、overlay、空间不足：读取 [references/storage.md](references/storage.md)。
- AI 工具权限与系统修复边界：读取 [references/agent-tools.md](references/agent-tools.md)。
- 必须源码重编译才能修复的系统问题：读取 [references/source-rebuild.md](references/source-rebuild.md)。

## 通用安全边界

本入口只保留全局安全底线；具体诊断命令、维护模式流程和场景策略按 reference 继续读取。

- 系统级修改先读 [references/system.md](references/system.md)，再按 `knowledge/system/maintenance.md` 执行维护模式、退出维护模式和持久化验证。
- 应用安装、AppImage、KARE/Kaiming 隔离和第三方 apt 源问题读 [references/applications.md](references/applications.md)。
- UKUI 自启动、锁屏超时、托盘、全局搜索等桌面链路读 [references/ukui.md](references/ukui.md)。
- 必须源码重编译才能修复的系统问题读 [references/source-rebuild.md](references/source-rebuild.md)；源码级功能增强或本地客制化切换到 `$HOME/.os-enhance-skill/SKILL.md`。
