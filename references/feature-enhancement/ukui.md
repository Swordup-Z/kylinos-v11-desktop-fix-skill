# 系统功能增强：UKUI 桌面

## 适用场景

- 增强 UKUI 全局搜索，例如添加 Bing/Google、自定义命令 provider、命令配置图形界面。
- 通过源码级客制化改变 UKUI 默认行为。
- 托盘、设置页、快捷键等系统功能的非故障类增强，例如托盘输入法状态角标。

## 知识入口

AI 工具进入 UKUI 增强索引后，先从用户问题中识别子场景，再读取对应 knowledge；不要一次性加载全部 UKUI 增强资料。

| 子场景 | 快速判断 | 对应 knowledge |
| --- | --- | --- |
| 全局搜索搜索引擎 | 需要新增、删除或调整全局搜索默认互联网搜索引擎 | `knowledge/feature-enhancement/ukui/search-web-engine.md` |
| 全局搜索自定义命令 | 需要在全局搜索里执行用户可配置命令，或给设置页增加命令配置入口 | `knowledge/feature-enhancement/ukui/search-command-provider.md` |
| 托盘输入法状态角标 | 需要在 UKUI 右侧托盘显示搜狗输入法内部中英状态 | `knowledge/feature-enhancement/ukui/system-tray-sogou-input-badge.md` |
| 源码级 patch 迁移 | 已确定要把某个 UKUI 增强 patch 套到本机源码树 | 对应 `knowledge/feature-enhancement/ukui/patches/<feature-id>/PATCHSET.md` |

- [`../../knowledge/feature-enhancement/ukui/README.md`](../../knowledge/feature-enhancement/ukui/README.md)

## 最小诊断

按用户问题选择对应命令，不要一次性执行所有 UKUI 诊断。

全局搜索增强：

```bash
dpkg-query -W -f='${binary:Package} ${Version} ${Source}\n' ukui-search
gsettings list-recursively org.ukui.search.settings 2>/dev/null || true
```

托盘增强：

```bash
dpkg-query -W -f='${binary:Package} ${Version} ${Source}\n' ukui-widget-system-tray
```
