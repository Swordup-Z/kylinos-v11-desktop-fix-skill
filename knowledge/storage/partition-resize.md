# 分区扩容与回滚资料

适用于根分区扩容可行性判断、缩小 DATA 分区、backup 分区调整、分区回滚资料保存。此章节只提供评估边界和资料保存规范，不提供普通桌面会话中可直接复制执行的破坏性分区修改流程。

## 根分区扩容判断

缩小 DATA 分区后是否能扩给根分区，取决于 GPT 分区顺序。只有空闲空间紧邻根分区之后，才容易扩容根分区。如果根分区和 DATA 分区之间隔着备份分区、swap 或其他分区，缩小 DATA 通常不能直接扩给根分区，可能需要移动/删除中间分区，风险显著升高。

分区调整前必须先备份重要数据，并在离线环境中操作；不要在未确认分区顺序、文件系统类型、boot/EFI/ostree 引导链、备份分区用途和回滚方案前执行缩容、移动或删除分区。在线会话中只做只读评估和备份，不提供直接执行分区修改的命令流程。

KylinOS Desktop V11 可能同时存在旧 SYSROOT/SYSBOOT 和当前 SYSROOT/SYSBOOT，例如旧分区被桌面自动挂载到 `/run/media/<user>/SYSROOT` 或 `/run/media/<user>/SYSBOOT`。这种状态下，任何 `grub-install`、`update-grub`、loader entry 修复、EFI 文件复制、fstab 修改或分区扩容都必须先确认当前实际启动链路：

```bash
cat /proc/cmdline
findmnt -o TARGET,SOURCE,FSTYPE,UUID,OPTIONS / /boot /boot/efi /sysroot 2>/dev/null || true
lsblk -o NAME,PATH,SIZE,FSTYPE,LABEL,UUID,PARTUUID,MOUNTPOINTS
rg -n 'root=UUID=|search(_fs_uuid|\.fs_uuid)|ostree=' /boot /boot/efi 2>/dev/null || true
```

如果 boot 文件引用了非当前根分区 UUID，或者 EFI/GRUB 指向旧 SYSBOOT，不要继续做空间清理或分区调整；先按引导修复问题处理，并在离线介质或明确回滚方案下操作。

## 分区回滚资料保存位置

分区、挂载、`fstab`、`sfdisk`、`parted`、`blkid`、backup 分区快照等非源码类系统回滚资料，不应长期散放在 `$HOME` 根目录，也不应放入源码客制化目录。统一保存到 DATA 分区共享回滚目录：

```text
/data/usershare/kylinos-system-rollbacks/storage/<scenario>/<timestamp>/
```

例如根分区和 backup 分区调整资料：

```text
/data/usershare/kylinos-system-rollbacks/storage/partition-resize/<timestamp>/
```

至少包含：

```text
<disk>.sfdisk
<disk>.parted.txt
blkid.txt
fstab.before-resize
fstab.after-resize
<partition>/backup-partition.tar
```

若目录原先临时建在 `$HOME` 下，确认内容属于回滚资料后，应移动到上述目录，并在 `/data/usershare/kylinos-system-rollbacks/README.md` 记录索引。源码重编译和本地客制化源码的回滚包仍按源码重编译知识库要求保存到 `/data/usershare/kylinos-local-sources/<component-or-fix>/rollback/<timestamp>/`。

## 根分区后紧邻 backup 分区时的扩容

如果 GPT 顺序类似：

```text
p3  SYSROOT       /        ext4
p4  KYLIN-BACKUP  /backup  ext4,noauto
p5  DATA          /data    ext4
```

且目标是“扩大根分区，同时缩小 backup，不移动 DATA”，可行性只能作为只读评估结论记录，不应在普通桌面会话中直接执行。判断原则：

- 若根分区目标大小小于等于 `p3 + p4可让出的空间`，可以只调整 p3/p4，不移动 DATA，风险相对较低。
- 若还要保留独立 backup，且根分区目标需要超过 p4 可让出的空间，就必须移动或缩小 DATA 的起点，通常应改用离线环境处理。
- 即使文件系统支持在线扩容，分区表、backup 分区重建、boot/EFI/ostree 引导链同步仍属于高风险操作，应改用离线维护方案，不在经验库中提供可直接复制执行的破坏性命令。

注意事项：

- 不要用十进制 GB 和 GiB 混算判断是否能达到“200G”等目标；以 `parted unit s print free` 的精确扇区为准。
- 若 `parted` 报告请求区间与可管理最近区间不一致，停止继续猜测，重新读取扇区级空闲区间后再创建分区。
- 如果 backup 分区包含真实恢复镜像或快照数据，不要直接格式化；先确认恢复机制和备份内容是否可以丢弃或迁移。
- 修改完成后必须重启验证根分区容量、DATA 挂载、boot/EFI 引导项、ostree deployment 和 `/backup` 手动挂载。
