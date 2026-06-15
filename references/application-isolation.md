# 应用隔离与宿主机边界

## 定位

这是应用隔离环境、宿主机路径不一致、桌面入口来源和 namespace 边界的分类入口。KARE/Kaiming 是当前系统中的典型实现。

## 适用场景

- 应用内 hostname 显示隔离环境名称，例如 `kare`。
- 开始菜单、固定项或桌面入口仍指向隔离环境应用。
- 从隔离环境误启动 UKUI 面板或系统工具。

## 先读知识章节

- KARE/Kaiming namespace、hostname、入口覆盖和恢复：[`../knowledge/applications/kare-namespace.md`](../knowledge/applications/kare-namespace.md)

## 最小诊断

```bash
hostname
ps -ef | rg -i '<app>|kare|kaiming' | rg -v rg || true
command -v <app-command> || true
```
