# 宿主机命令行安装应用

## 适用场景

- 用户要通过 `apt`、`apt-get install ./<package>.deb` 或 `dpkg -i` 安装宿主机应用。
- 需要避免应用被安装到 KARE rootfs/overlay。
- 需要确认当前终端是否在宿主机 namespace。

不适用于 AppImage 用户级安装、第三方 apt 源清理或 KARE 应用桌面入口修复；这些场景读取本目录其他章节。

## 基础原则

1. 涉及 `/usr`、`/etc`、`/opt`、系统包或系统服务前，先检查维护模式。
2. 优先在宿主机 namespace 的终端中安装桌面应用。
3. 安装前确认架构、包来源、命令入口和桌面入口；安装后验证进程路径和 namespace。

## 安装前诊断

```bash
mm-cli -s
readlink /proc/$$/ns/mnt
readlink /proc/$$/ns/uts
hostname
dpkg --print-architecture
```

只有当前是 maintain mode，且当前 shell 位于宿主机 namespace，才继续执行实际安装命令。若 `hostname` 显示 `kare`，或 namespace 与 `ukui-session` 不一致，当前很可能在 KARE 环境内，不应在该终端中安装宿主机应用。

## 安装

`.deb` 包优先使用 apt 处理依赖：

```bash
sudo apt-get install ./<package>.deb
```

只有确认依赖已经满足或需要先解包排错时，才使用：

```bash
sudo dpkg -i <package>.deb
sudo apt-get -f install
```

## 验证

```bash
command -v <app-command> || true
dpkg -l | rg -i '<package-or-app>' || true
rg -n '<app-name>|<desktop-id>' /usr/share/applications "$HOME/.local/share/applications" 2>/dev/null || true
```

如果桌面入口包含 `Exec=/usr/bin/kare run ...`，或路径位于 `/opt/kare`、`/opt/kare-applications`，说明它仍然是 KARE 应用。终端、开发工具、代理工具等需要访问宿主系统状态的应用，应优先使用原生宿主机安装。

完成系统级安装后，按维护模式流程执行：

```bash
sudo mm-cli -c -a
```

随后重启系统验证持久性。
