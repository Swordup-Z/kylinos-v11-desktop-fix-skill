# TUN 设备节点

适用于 Clash Verge、代理服务或其他网络工具报 `/dev/net/tun` 缺失、TUN 无法创建，但当前问题尚未指向 Clash Verge 服务安装或 NetworkManager 弹窗。

## 诊断

先做只读检查：

```bash
mm-cli -s
modinfo tun 2>/dev/null || true
ls -l /dev/net/tun 2>/dev/null || true
ip link
```

在当前内核上，`tun` 可能是内建模块，因此 `modinfo tun` 可能显示 `filename: (builtin)`。这表示内核支持 TUN，缺失的可能只是 `/dev/net/tun` 设备节点。

## 修复

如果 `/dev/net/tun` 不存在，在宿主机上创建设备节点：

```bash
pkexec sh -c 'mkdir -p /dev/net; if [ ! -e /dev/net/tun ]; then mknod /dev/net/tun c 10 200; fi; chmod 0666 /dev/net/tun'
```

在宿主机上下文验证：

```bash
ls -l /dev/net/tun
```

预期形态：

```text
crw-rw-rw- 1 root root 10, 200 ... /dev/net/tun
```

如果沙箱终端仍然看不到 `/dev/net/tun`，不要直接判断宿主机修复失败。应使用宿主机/提权命令重新检查。

## 分流

- 如果 `/dev/net/tun` 已正常，但 Clash Verge 服务安装失败，读取 [clash-verge-service.md](clash-verge-service.md)。
- 如果 TUN 启动后触发麒麟网络发现弹窗，读取 [tun-network-discovery.md](tun-network-discovery.md)。
- 如果问题只停留在 TUN 节点，完成验证后停止，不要继续加载 Clash Verge 迁移或代理组章节。
