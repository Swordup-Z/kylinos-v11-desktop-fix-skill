# 系统问题修复：存储与挂载

## 适用场景

- 根分区空间不足、DATA 分区未按预期使用。
- `/home` 挂载位置、backup 分区、overlay/KARE 合并视图判断。
- 离线分区调整前的风险边界。

## 知识入口

进入存储与挂载索引后，先从下面子场景中选择一个；没有命中时停止继续加载存储 knowledge。

- [`../knowledge/storage/README.md`](../knowledge/storage/README.md)

| 子场景 | 快速判断 | 对应 knowledge |
| --- | --- | --- |
| 挂载布局 | 判断 `/home`、DATA、backup、overlay 真实关系，只做只读诊断 | `knowledge/storage/mount-layout.md` |
| 空间增长 | Kaiming/KARE/ostree/overlay upperdir 空间增长或根分区占用分析 | `knowledge/storage/space-growth.md` |
| 分区扩容 | 缩小 DATA、扩大根分区、backup 分区调整、分区回滚资料保存 | `knowledge/storage/partition-resize.md` |

## 最小诊断

```bash
findmnt -T "$HOME"
findmnt
lsblk -f
df -hT
```
