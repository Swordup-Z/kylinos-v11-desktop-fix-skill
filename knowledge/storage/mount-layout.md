# 存储挂载布局

适用于判断 Kylin Desktop V11 上根分区、DATA 分区、`/home`、backup 分区和 overlay 挂载的真实关系。只做只读诊断，不处理空间清理或分区扩容。

## 诊断命令

先做只读诊断：

```bash
df -hT / /home "$HOME" /data /data/home 2>/dev/null || true
findmnt -o TARGET,SOURCE,FSTYPE,SIZE,USED,AVAIL,USE%,OPTIONS / /home "$HOME" /data 2>/dev/null || true
findmnt -t overlay -o TARGET,SOURCE,FSTYPE,SIZE,USED,AVAIL,USE%,OPTIONS
lsblk -o NAME,PATH,SIZE,FSTYPE,LABEL,UUID,FSUSED,FSAVAIL,FSUSE%,MOUNTPOINTS
```

查看分区起止位置需要 root，但属于只读检查：

```bash
pkexec parted -s /dev/<disk> unit GiB print free
```

## `/home` 和 DATA 分区

DATA 分区可能同时挂载到 `/data`，并把其中的 `/home` 子目录挂载为系统 `/home`。例如：

```text
/data  /dev/<disk-part>
/home  /dev/<disk-part>[/home]
```

此时用户目录实际在 DATA 分区上，`$HOME` 等价于 `/data/home/<user>`。文件管理器中的“数据盘”快捷方式可能指向 `/data/usershare`，它是共享目录，不等于用户主目录；看到 `/data` 顶层或 `/data/usershare` 为空，不代表 `$HOME` 没有使用 DATA 分区。

## `noauto` backup 分区

`/etc/fstab` 中 backup 分区使用 `noauto`，只表示它不会在开机时自动挂载，不等于这个分区一定无用。恢复、快照、系统备份或厂商工具可能在需要时按需挂载它。

判断 backup 分区是否实际承载数据时，先只读挂载检查内容和占用：

```bash
mount -o ro /dev/<backup-partition> <mountpoint>
find <mountpoint> -maxdepth 2 -mindepth 1 -print
du -sh <mountpoint>
umount <mountpoint>
```

如果只看到 `snapshots`、`current`、`lost+found` 等目录且占用极小，可以认为当前没有实际备份数据；但仍不应直接删除该分区，除非已经确认系统恢复链路不依赖它，或用户明确接受删除独立备份分区的风险。

## overlay 挂载判断

Kylin Desktop V11 上常见 overlay 分两类：

- 磐石架构/ostree 系统 overlay：如 `/usr`、`/etc`、`/var/lib`，用于把只读系统基线和可写层合并成当前系统视图。
- KARE 应用 overlay：如 `/var/opt/kare-applications/*/merge/usr`、`merge/var`、`merge/etc`、`merge/opt`、`merge/root`，用于 KARE 应用运行环境。

这些 overlay 是实际使用中的挂载视图，不能随意卸载或删除。`df` 中多个 overlay 显示相同容量和相同已用空间时，不表示每个 overlay 都额外占用了同样大小；它们通常映射到底层同一个根分区。

判断真实额外占用时，看各 overlay 的 `upperdir`，不要看 `merge` 目录的 `du` 结果。`merge` 是合并视图，可能把 lowerdir 的内容重复算入，看起来很大但不代表新增占用。

示例：

```bash
findmnt -t overlay -o TARGET,OPTIONS
du -sh /sysroot/ostree/pkgs/*/*-ovl/*-upper /sysroot/ostree/pkgs/*/*-ovl/*-tmpupper 2>/dev/null || true
du -sh /opt/kare-applications/*/upper /opt/kare-applications/*/work 2>/dev/null || true
du -xhd2 /opt/kare-applications/*/upper 2>/dev/null | sort -h | tail -80
```

不要用下面这类结果判断真实占用：

```bash
du -sh /opt/kare-applications/*/merge
```

因为 `merge` 是 overlay 合并后的视图。

## 分流

- 如果问题是 Kaiming/KARE/ostree 空间增长，读取 [space-growth.md](space-growth.md)。
- 如果问题是根分区扩容、缩小 DATA 或 backup 分区调整，读取 [partition-resize.md](partition-resize.md)。
- 如果只需要确认挂载关系，完成诊断后停止，不要继续加载空间治理或分区扩容章节。
