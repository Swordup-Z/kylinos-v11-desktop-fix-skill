# Clash Verge KARE 迁移残留

适用于用户从 KARE 打包版改为宿主机原生 `.deb` 安装后，出现旧服务单元、旧桌面入口、KARE 路径残留、服务 `203/EXEC`、普通用户 sidecar 和 systemd core 冲突，或代理组消失。

## 迁移状态诊断

不能只看 `clash-verge` 命令是否存在，还要同时确认包归属、桌面入口、服务单元和运行进程都已离开 KARE 路径。

先确认当前 shell 位于宿主机 namespace，且系统处于维护模式：

```bash
mm-cli -s
hostname
readlink /proc/$$/ns/mnt
readlink /proc/$$/ns/uts
```

宿主机原生安装的典型特征：

```bash
command -v clash-verge
command -v verge-mihomo
dpkg -l | rg -i 'clash|verge|mihomo'
dpkg -S /usr/bin/clash-verge /usr/bin/verge-mihomo /usr/bin/clash-verge-service /usr/bin/clash-verge-service-install 2>/dev/null || true
sed -n '1,80p' /usr/share/applications/*Clash* 2>/dev/null
```

预期结果是 `clash-verge` 包提供 `/usr/bin/clash-verge`、`/usr/bin/verge-mihomo`、`/usr/bin/clash-verge-service` 和 `/usr/bin/clash-verge-service-install`，桌面入口使用：

```text
Exec=clash-verge %u
```

如果仍看到 `Exec=/usr/bin/kare run ...`，或路径位于 `/opt/kare`、`/opt/kare-applications`，说明还没有完全切换到宿主机原生入口。

## 旧服务单元残留

迁移后要特别检查旧服务单元是否残留 KARE 路径：

```bash
systemctl status clash-verge-service --no-pager
sed -n '1,120p' /etc/systemd/system/clash-verge-service.service
```

如果服务报 `status=203/EXEC`，且 `ExecStart=` 仍指向：

```text
/var/opt/kare-applications/shadow/upper/usr/bin/clash-verge-service
```

说明旧 KARE 服务单元覆盖了新包。应在维护模式下把 `ExecStart=` 修正为宿主机路径：

```ini
ExecStart=/usr/bin/clash-verge-service
```

然后重新加载并启用服务：

```bash
pkexec systemctl daemon-reload
pkexec systemctl enable --now clash-verge-service
```

## 端口冲突

如果服务 core 和图形界面 sidecar 同时运行，可能出现端口冲突：

```text
Start Mixed(http+socks) server error: listen tcp 127.0.0.1:<port>: bind: address already in use
```

此时检查进程父子关系：

```bash
pgrep -a 'clash-verge|verge-mihomo' || true
ps -o pid,ppid,user,group,comm,args -p <pid-list>
ss -ltnup 2>/dev/null | rg ':53|:<mixed-port>|verge-mihomo' || true
```

若普通用户 `verge-mihomo` 是由图形进程拉起，而 systemd 服务也在运行，应完整退出 Clash Verge 图形进程，再重启 `clash-verge-service`，让图形进程在服务已就绪的状态下重新连接服务。不要删除或覆盖 `verge-mihomo`。

## 代理组消失

立即检查核心二进制路径：

```bash
ls -l /usr/bin/verge-mihomo
/usr/bin/verge-mihomo -v
ls -l /opt/kare-applications/shadow/upper/usr/bin/verge-mihomo
```

如果 `/usr/bin/verge-mihomo` 缺失或不可用，说明代理组可能因为 Clash Verge 找不到预期核心而消失。已知的一种恢复方式是补回该核心二进制：

```bash
sudo cp /opt/kare-applications/shadow/upper/usr/bin/verge-mihomo /usr/bin/verge-mihomo
```

该命令会写入 `/usr/bin`。除非 `/usr/bin/verge-mihomo` 缺失或损坏，并且用户明确同意，否则不要执行。确实需要复制时，只能从上面的 KARE 打包二进制复制，并用 `/usr/bin/verge-mihomo -v` 验证。

## 停止条件

如果桌面入口、服务单元、运行进程和核心路径均已转到宿主机原生路径，停止加载迁移知识；后续 TUN 启用问题再回到 [clash-verge-service.md](clash-verge-service.md)。
