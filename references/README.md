# References 路由说明

`references/` 是系统修复场景的策略路由入口，不保存详细修复过程。

功能增强、本地客制化和默认行为调整已拆分到 `$HOME/.os-enhance-skill`。

## 场景入口

- [system.md](system.md)：维护模式、系统保护、服务异常、系统体检噪声。
- [applications.md](applications.md)：应用安装失败、KARE/Kaiming 隔离、AppImage、第三方源。
- [ukui.md](ukui.md)：UKUI 自启动、全局搜索异常、托盘、快捷键、AI 组件、面板/任务栏。
- [network.md](network.md)：代理、TUN、Clash Verge、网络弹窗。
- [hardware.md](hardware.md)：指纹、生物识别、图形频率、硬件稳定性。
- [storage.md](storage.md)：分区、挂载、DATA、`/home`、overlay、空间不足。
- [agent-tools.md](agent-tools.md)：AI 工具权限与系统修复边界异常。
- [source-rebuild.md](source-rebuild.md)：必须通过源码重编译才能修复的系统问题。

## 使用方式

1. 先确认任务属于系统修复，而不是功能增强。
2. 再判断场景：system、applications、ukui、network、hardware、storage、agent-tools 或 source-rebuild。
3. 只读取一个最小匹配 reference。
4. reference 可以列出子场景和对应 knowledge 路径，但不直接展开具体修复内容。
5. 进入 `knowledge/<scenario>/README.md` 后，只读取与用户问题匹配的具体章节；不要一次性读取整个场景目录。
6. 如果 reference 和 knowledge 索引都没有命中具体章节，停止继续加载本 skill。基于当前系统诊断和通用能力处理，或先询问用户补充现象；不要遍历其他 reference 或整个 knowledge。

## 层级链路

固定链路如下：

```text
需求类型
-> 实际场景 reference
-> 场景 knowledge 索引
-> 细分 knowledge 章节
```

对应文件路径：

```text
SKILL.md
-> references/<scenario>.md
-> knowledge/<scenario>/README.md
-> knowledge/<scenario>/<topic>.md
```

只有 `README.md` 明确说明当前场景复用其他场景知识时，才跨场景读取；否则保持同类型、同场景路径。

## Fallback

只有问题明确属于维护模式、系统保护、systemd/D-Bus 基础服务或系统体检噪声时，才读取 [system.md](system.md)。如果只是“不知道该读哪个场景”，不要把 [system.md](system.md) 当成默认全文 fallback。
