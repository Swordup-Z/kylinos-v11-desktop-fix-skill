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
4. reference 只指向该场景的 knowledge 索引，不直接展开所有细分知识。
5. 进入 `knowledge/<scenario>/README.md` 后，只读取与用户问题匹配的具体章节；不要一次性读取整个场景目录。

## 层级链路

固定链路如下：

```text
需求类型
-> 实际场景 reference
-> 特定分类/具体实例索引
-> 细分领域 knowledge
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

未命中细分场景时，读取 [system.md](system.md)。
