# TUN 触发麒麟网络发现弹窗

适用于用户反馈“连接 WiFi 后总弹出是否允许其他设备发现本机”，但实际弹窗发生在 Clash TUN 启动后，或 NetworkManager/麒麟网络管理把 TUN 虚拟接口当成新网络连接处理。

## 判断时间线

典型时间线：

1. WiFi 已先连接成功。
2. Clash TUN 后创建 `Meta` 等 TUN 虚拟接口。
3. NetworkManager 将该接口显示为 `tun` 类型连接。
4. `kylin-nm` 或安全中心弹出网络发现策略询问。

先诊断：

```bash
nmcli -t -f NAME,UUID,TYPE,DEVICE connection show --active 2>/dev/null || true
nmcli -t -f DEVICE,TYPE,STATE,CONNECTION device status 2>/dev/null || true
ip -br link show
journalctl -b --since '<time>' --no-pager | rg -i 'Meta|tun|kylin-nm|NetworkManager|discover|firewall|networkmode'
```

如果看到类似下面的状态，说明 `Meta` 是 Clash TUN 创建的本机虚拟接口，不是新的外部 WiFi：

```text
Meta:tun:connected (externally):Meta
```

## 运行时验证

不要通过“让 Clash 早于 WiFi 启动”修复。TUN 自动路由依赖真实网络接口，提前启动可能只会引入启动竞态。更直接的修复是让 NetworkManager 忽略 Clash TUN 虚拟接口，避免它进入麒麟网络发现策略流程，同时保留 TUN 设备本身。

先做运行时验证：

```bash
nmcli device set Meta managed no
nmcli -t -f DEVICE,TYPE,STATE,CONNECTION device status 2>/dev/null || true
ip -br link show Meta
ip addr show Meta
systemctl status clash-verge-service --no-pager
```

预期 `Meta` 在 NetworkManager 中变为：

```text
Meta:tun:unmanaged:
```

但 `ip link` 里仍然存在并保持 `UP`，Clash 服务仍然 active。

## 持久化

确认运行时验证正常后，在维护模式下写入持久化 NetworkManager 配置：

```ini
# /etc/NetworkManager/conf.d/99-clash-meta-unmanaged.conf
[device-clash-meta-unmanaged]
match-device=interface-name:Meta
managed=false
```

然后 reload 配置，不必重启 NetworkManager：

```bash
nmcli general reload
```

可重启 Clash 服务验证 TUN 重建后仍然被忽略：

```bash
pkexec systemctl restart clash-verge-service
nmcli -t -f DEVICE,TYPE,STATE,CONNECTION device status 2>/dev/null || true
ip -br link show Meta
```

预期重建后仍是 `Meta:tun:unmanaged:`，且服务日志包含类似：

```text
[TUN] Tun adapter listening at: Meta(...)
```

如果用户使用的 TUN 接口名不是 `Meta`，不要硬编码该名字；先从 `nmcli device status` 和 `ip -br link show` 找到实际接口名，再替换 `match-device=interface-name:<tun-iface>`。

## 停止条件

如果弹窗并非由 TUN 虚拟接口触发，停止加载本章节，不要继续读取 Clash Verge 服务或迁移章节；应回到实际网络组件或系统日志重新判断场景。
