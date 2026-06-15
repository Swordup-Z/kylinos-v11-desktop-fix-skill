# References 路由说明

`references/` 是场景分类和路由入口，不是具体包名或单次故障记录。每个文件负责回答三个问题：

- 这个问题属于哪个功能场景。
- 先读取哪些 `knowledge/` 章节。
- 做哪些最小诊断来确认方向。

## 命名规则

- 用功能或属性命名，例如 `application-installation.md`、`proxy-tun.md`、`desktop-service-activation.md`。
- 不用具体包名作为 reference 文件名；具体包名、服务名、命令名作为触发线索写在适用场景或 knowledge 章节中。
- 如果一个问题同时命中多个场景，先读最接近用户目标的 reference，再按该 reference 指向的 knowledge 继续。
- 如果没有命中专门场景，读取 `system-maintenance.md`。

## 当前场景入口

### 基础与安全

- `system-maintenance.md`：维护模式、磐石架构、系统级修复边界、未覆盖问题闭环。
- `system-health-noise.md`：系统体检后的低风险日志噪声和残留配置清理。
- `source-rebuild.md`：源码重编译、ABI 风险、本地客制化源码和回滚包索引。

### 应用与运行环境

- `application-installation.md`：应用安装、卸载、AppImage、第三方 apt 源和宿主机安装边界。
- `application-isolation.md`：KARE/Kaiming namespace、宿主机路径、桌面入口和误启动边界。

### UKUI 桌面功能

- `ukui-autostart.md`：开机自启动和设置界面显示。
- `ukui-keybindings.md`：全局快捷键、系统占用提示和全局搜索快捷键。
- `ukui-search.md`：全局搜索结果来源、软件商店结果和搜索引擎设置。
- `ukui-system-tray.md`：右侧系统托盘顺序、隐藏区和持久化。
- `desktop-service-activation.md`：桌面服务启动顺序、D-Bus activation、孤儿进程和面板/任务栏异常。
- `desktop-ai-components.md`：桌面 AI 助手、AI 子系统和相关残留清理。

### 网络、硬件与存储

- `proxy-tun.md`：代理客户端 TUN 模式、TUN 设备、代理服务和代理核心路径。
- `biometric-authentication.md`：指纹/生物识别驱动、设备识别和认证服务。
- `graphics-frequency.md`：图形、GPU/显示频率、devfreq 和硬件强相关稳定性。
- `storage-layout.md`：分区、挂载、DATA、`/home`、backup、overlay 和空间判断。

### AI 工具配置

- `agent-tool-permissions.md`：AI 工具权限、Codex full access 和维护模式/root 边界。
- `agent-global-prompts.md`：Codex、Claude Code、opencode 等全局提示词和 skill 自动加载规则。
