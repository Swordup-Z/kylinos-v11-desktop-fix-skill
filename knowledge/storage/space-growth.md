# Kaiming/KARE 与 ostree 空间增长

适用于根分区或系统层空间增长，怀疑与 Kaiming、KARE、ostree deployment、overlay upperdir 或容器层有关的场景。此章节不处理分区扩容。

## Kaiming 层诊断

Kaiming 应用会在 `/var/opt/kaiming` 下保存 app、runtime 和 base 层。空间占用增长时，先确认是否存在同一 app/runtime/base 的多版本目录或孤儿层，不要直接删除 `/var/opt/kaiming/layers`：

```bash
/opt/kaiming-tools/bin/kaiming list
/opt/kaiming-tools/bin/kaiming update --upgradable
du -sh /var/opt/kaiming/layers/stable/*/*/*/*/* 2>/dev/null | sort -h
find /var/opt/kaiming/layers/stable -mindepth 5 -maxdepth 5 -type d -printf '%p\n' 2>/dev/null | sort
```

如果每个应用、runtime、base 只有一个版本目录，则大占用通常不是“旧版本不断复制”，而是 Kaiming base/runtime 本身体积较大。

## 安全策略

- 空间清理工具不要把“卸载应用”作为主清理动作；卸载应用属于应用管理，不等同于清理旧版本容器。
- 不需要某个 Kaiming 应用时，才用 `kaiming uninstall <app-id>` 卸载，不手工删除 app 层。
- 只要仍保留任何 Kaiming 应用，base/runtime 通常仍会保留。
- 若怀疑有孤儿版本目录，优先从 `/var/opt/kaiming/info/*.list` 读取当前实际安装层目录，再与 `/var/opt/kaiming/layers/stable/<arch>/<kind>/<id>/<module>/<version>` 对比；不要只解析 `kaiming list` 的对齐表格，因为“名称”列可能包含空格，容易把版本列读错。
- 旧版本容器候选必须确认不属于当前 info 清单、没有被 mount 或进程使用；优先移动到 DATA 分区隔离目录，而不是直接删除。
- 对自动启动或预热造成的持续占用，可优先在用户级 `.config/autostart` 写入 `Hidden=true` 覆盖，而不是删除系统级 `.desktop`。

Kaiming/KARE/ostree 空间治理适合做成独立工具；具体 UI、构建、验证和实现要求应维护在对应项目目录的项目级提示词中，不写入本系统修复知识库。本知识库只规定安全边界：工具必须默认先报告候选项和风险，不应自动清理；任何会改变系统或用户会话行为的动作都必须先展示候选项、执行计划和可回滚路径，不能静默执行。

## ostree 和引导文件边界

ostree 部署、`/boot`、EFI、GRUB、loader entries、`/etc/fstab` 和分区表不属于普通空间清理对象。即使 `/sysroot/ostree/deploy` 看起来很大，也不能据此删除 deployment 或 boot 文件；应先确认当前 `root=UUID`、`ostree=`、loader entry、EFI 启动路径和双 SYSROOT/SYSBOOT 分区状态。

## 停止条件

如果空间增长不来自 Kaiming/KARE/ostree/overlay，停止加载本章节，回到实际占用路径对应的应用或系统场景。不要因为“空间不足”直接读取分区扩容章节。
