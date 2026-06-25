# UKUI 锁屏超时设置不生效

## 适用场景

- 用户在设置界面把自动锁屏、屏保、关闭屏幕或睡眠时间设置为较长时间，但实际很快进入锁屏。
- `org.ukui.power-manager` 中显示器/睡眠超时已经是期望值，但 `org.ukui.screensaver idle-delay` 仍停留在默认值或旧值。
- 设置界面调整后只影响电源管理，不影响锁屏服务实际读取的屏保超时。

## 背景

KylinOS Desktop V11 的 UKUI 可能同时存在多套超时配置：

- `org.ukui.power-manager sleep-display-ac`
- `org.ukui.power-manager sleep-display-battery`
- `org.ukui.power-manager sleep-computer-ac`
- `org.ukui.power-manager sleep-computer-battery`
- `org.ukui.screensaver idle-delay`
- `org.gnome.desktop.session idle-delay`

其中 `org.ukui.power-manager` 的时间通常以秒为单位，`org.ukui.screensaver idle-delay` 的时间通常以分钟为单位。锁屏服务实际触发时可能读取 `org.ukui.screensaver idle-delay`，因此只改电源管理键会造成“设置界面看起来是 1 小时，但几分钟就锁屏”。

## 诊断

先检查当前值：

```bash
gsettings list-recursively org.ukui.screensaver
gsettings list-recursively org.ukui.power-manager
gsettings list-recursively org.gnome.desktop.session
```

重点比较：

```bash
gsettings get org.ukui.screensaver idle-delay
gsettings get org.ukui.power-manager sleep-display-ac
gsettings get org.ukui.power-manager sleep-display-battery
gsettings get org.gnome.desktop.session idle-delay
```

再看本次会话里是否是 `ukui-screensaver` 提前触发：

```bash
journalctl --user -b --no-pager | rg -i 'screensaver|screen.*lock|lock.*screen|idle|power-manager' | tail -n 200
```

如果日志中出现 `ukui-screensaver-dialog` 被唤起，而 `org.ukui.screensaver idle-delay` 明显短于用户期望值，优先修复设置链路。

## 修复策略

不要把源码级修补作为默认方案。控制中心、电源管理和锁屏组件通常是独立二进制或插件，直接修改源码/替换系统库会提高维护成本，也可能被系统更新覆盖。

优先选择：

- 用户级 GSettings 修复当前值。
- 用户级 systemd 服务监听电源设置变化，把电源超时同步到锁屏实际读取的键。
- 只有确认必须修改系统包、且用户接受维护成本时，才进入源码重编译流程。

## 当前值修复

例如把当前用户的锁屏超时修到 1 小时：

```bash
gsettings set org.ukui.screensaver idle-delay 60
gsettings set org.gnome.desktop.session idle-delay 3600
```

如果电源管理本身也不是 1 小时，同步设置：

```bash
gsettings set org.ukui.power-manager sleep-display-ac 3600
gsettings set org.ukui.power-manager sleep-display-battery 3600
gsettings set org.ukui.power-manager sleep-computer-ac 3600
gsettings set org.ukui.power-manager sleep-computer-battery 3600
```

## 持久同步方案

若设置界面后续仍只写电源管理键，可为当前用户安装轻量同步器。它启动时同步一次，并监听 `org.ukui.power-manager` 变化；当用户在设置界面调整关闭屏幕时间后，自动把秒数换算成分钟写入 `org.ukui.screensaver idle-delay`，同时同步 `org.gnome.desktop.session idle-delay`。

脚本位置建议使用：

```text
$HOME/.local/bin/ukui-lock-delay-sync
```

用户级 systemd 单元建议使用：

```text
$HOME/.config/systemd/user/ukui-lock-delay-sync.service
```

服务启用方式：

```bash
chmod 755 "$HOME/.local/bin/ukui-lock-delay-sync"
systemctl --user daemon-reload
systemctl --user enable --now ukui-lock-delay-sync.service
```

同步器应满足：

- 只写用户级 GSettings，不修改 `/usr`、`/etc`、`/opt`。
- 只在目标值不一致时写入，避免 dconf 频繁写入。
- 使用 `gsettings monitor org.ukui.power-manager` 监听设置变化。
- 可选使用 `upower --monitor-detail` 监听 AC/电池切换，按当前电源来源选择 `sleep-display-ac` 或 `sleep-display-battery`。
- 错误输出交给 systemd user journal，不在桌面上刷屏。

## 验证

检查服务状态：

```bash
systemctl --user status ukui-lock-delay-sync.service --no-pager
```

模拟验证时可临时改变电源管理值，再恢复：

```bash
gsettings set org.ukui.screensaver idle-delay 5
gsettings set org.ukui.power-manager sleep-display-battery 3540
sleep 1
gsettings get org.ukui.screensaver idle-delay
gsettings get org.gnome.desktop.session idle-delay
gsettings set org.ukui.power-manager sleep-display-battery 3600
```

期望 `org.ukui.screensaver idle-delay` 按分钟同步，例如 3540 秒同步为 59 分钟，3600 秒同步为 60 分钟。

## 回滚

```bash
systemctl --user disable --now ukui-lock-delay-sync.service
rm -f "$HOME/.config/systemd/user/ukui-lock-delay-sync.service"
rm -f "$HOME/.local/bin/ukui-lock-delay-sync"
systemctl --user daemon-reload
```

回滚后再按用户期望手动设置当前值。
