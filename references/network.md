# 系统问题修复：网络与代理

## 适用场景

- Clash Verge Rev TUN 模式失败。
- `/dev/net/tun` 缺失、代理服务安装失败、代理核心路径异常。
- 网络模式弹窗、TUN 虚拟连接触发网络发现提示等代理相关系统集成问题。

## 知识入口

进入网络与代理索引后，先从下面子场景中选择一个；没有命中时停止继续加载网络 knowledge。

- [`../knowledge/network/README.md`](../knowledge/network/README.md)

| 子场景 | 快速判断 | 对应 knowledge |
| --- | --- | --- |
| TUN 设备节点 | `/dev/net/tun` 缺失、TUN 字符设备异常，尚未指向 Clash Verge 服务 | `knowledge/network/tun-device.md` |
| Clash Verge 服务与启用 | `clash-verge-service` 安装失败、服务 inactive、IPC socket 缺失、界面 TUN 未启用 | `knowledge/network/clash-verge-service.md` |
| KARE 迁移残留 | 原 KARE 版迁移到宿主机 `.deb` 后仍有 KARE ExecStart、desktop 入口或 core 路径残留 | `knowledge/network/clash-verge-kare-migration.md` |
| TUN 网络发现弹窗 | TUN 接口被 NetworkManager/kylin-nm 当成新网络，弹出是否允许发现本机 | `knowledge/network/tun-network-discovery.md` |

## 最小诊断

```bash
mm-cli -s
ls -l /dev/net/tun 2>/dev/null || true
systemctl status clash-verge-service --no-pager 2>/dev/null || true
ip link
```
