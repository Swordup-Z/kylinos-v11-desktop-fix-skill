# Clash Verge 服务与 TUN 启用

适用于 Clash Verge Rev TUN 模式失败、`clash-verge-service` 安装失败、服务未启用、IPC socket 缺失，或服务正常后需要确认 TUN 是否由应用启用。

应用数据目录通常是：

```bash
~/.local/share/io.github.clash-verge-rev.clash-verge-rev
```

## 安全规则

- 不要删除、移动或覆盖任何已有的 `mihomo`、`verge-mihomo`、`clash`、`clash-verge` 可执行文件。
- 修改服务或核心路径前，先检查当前路径，并保留用户已经恢复正常的工作状态。
- 在 KARE 环境中进行图形/系统修复时，优先使用 `pkexec`，不要优先使用 `sudo`。Clash Verge 运行在 KARE 环境内，应用内 `sudo` 可能因为主机名 `kare` 和无终端密码输入而失败。

## 诊断

先查看日志：

```bash
tail -200 ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/logs/latest.log
tail -200 ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/logs/sidecar/sidecar_latest.log
```

已知的服务安装失败特征：

```text
pkexec failed with code 127, falling back to sudo
sudo: 无法解析主机：kare
sudo: a terminal is required to read the password
```

检查 TUN 和服务状态：

```bash
mm-cli -s
ls -l /dev/net/tun 2>/dev/null || true
systemctl status clash-verge-service --no-pager
systemctl is-enabled clash-verge-service
systemctl is-active clash-verge-service
```

如果 `/dev/net/tun` 缺失，先读取 [tun-device.md](tun-device.md)，不要继续安装服务。

## 安装服务

先确认 KARE 打包的二进制存在：

```bash
ls -l /opt/kare-applications/shadow/upper/usr/bin/clash-verge-service-install
ls -l /opt/kare-applications/shadow/upper/usr/bin/clash-verge-service
ls -l /opt/kare-applications/shadow/upper/usr/bin/verge-mihomo
```

用当前用户主组安装 systemd 服务。不要硬编码 UID/GID；先动态读取当前用户的主组 GID：

```bash
SERVICE_GID="$(id -g)"
pkexec /usr/bin/env CLASH_VERGE_SERVICE_GID="$SERVICE_GID" /opt/kare-applications/shadow/upper/usr/bin/clash-verge-service-install
```

然后验证：

```bash
systemctl status clash-verge-service --no-pager
systemctl is-enabled clash-verge-service
systemctl is-active clash-verge-service
sed -n '1,120p' /etc/systemd/system/clash-verge-service.service
ls -la /tmp/verge
```

预期服务属性：

```text
Loaded: loaded (/etc/systemd/system/clash-verge-service.service; enabled)
Active: active (running)
Group=<当前用户名或当前用户主组名>
ExecStart=/var/opt/kare-applications/shadow/upper/usr/bin/clash-verge-service
```

预期 IPC socket：

```text
/tmp/verge/clash-verge-service.sock
```

## 启用 TUN

服务和 TUN 设备正常后，让用户从托盘完全退出 Clash Verge，再重新启动。然后在 Clash Verge 界面中启用 TUN 模式。

在界面启用前，配置文件可能仍显示 TUN 关闭：

```bash
rg -n 'enable_tun_mode|^tun:|enable:|stack:|auto-route|dns-hijack' \
  ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/verge.yaml \
  ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/config.yaml \
  ~/.local/share/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml
```

界面切换前的常见状态：

```text
enable_tun_mode: false
tun:
  enable: false
```

除非用户明确要求，不要手动编辑这些 YAML 文件。应让 Clash Verge 自己写入运行时配置。

## 最终验证

向用户报告以下事实：

- `/dev/net/tun` 存在，并且是字符设备 `10,200`。
- `clash-verge-service` 是 `enabled` 且 `active`。
- `/tmp/verge/clash-verge-service.sock` 存在。
- TUN 是否已经在应用配置中启用，或者仍等待用户在界面中切换。
- 如果已经启用 TUN，`ip -br link show` 中应出现 `Meta` 等 TUN 网卡，服务日志应出现类似 `[TUN] Tun adapter listening`。

如果服务安装和 TUN 启用均正常，停止加载网络知识；只有出现 KARE 迁移残留、代理组消失或网络发现弹窗时，才读取对应章节。
