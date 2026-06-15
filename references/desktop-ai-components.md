# 桌面 AI 组件

## 定位

这是桌面 AI 助手、托盘 AI 入口、AI 子系统和相关残留清理的分类入口。Kylin AI/Kaiming AI 助手是当前已沉淀的具体实现。

## 适用场景

- 桌面右下角 AI 助手需要卸载或隐藏。
- `kylin-ai-runtime`、Kaiming AI 助手残留。
- 设置项里 AI 子系统残留但已不可用。

## 先读知识章节

- AI 子系统卸载边界和残留清理：[`../knowledge/ukui/kylin-ai-subsystem.md`](../knowledge/ukui/kylin-ai-subsystem.md)

## 最小诊断

```bash
ps -ef | rg -i 'ai|kylin-ai|kaiming' | rg -v rg || true
dpkg -l | rg -i 'kylin-ai|ai-runtime' || true
```
