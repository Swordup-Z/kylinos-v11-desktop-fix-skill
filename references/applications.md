# 系统问题修复：应用与隔离

## 适用场景

- 宿主机命令行安装 `.deb` 或系统包，尤其是需要避免误装到 KARE 环境时。
- AppImage 架构、FUSE 兼容库或用户级集成问题。
- aTrust、VPN/安全客户端等第三方安全软件安装后组件失败、虚拟网络组件异常或内核模块残留。
- 用户级 `.desktop`、开始菜单图标或无效入口问题。
- 第三方 apt 源残留、公钥错误或源订阅清理。
- KARE/Kaiming 宿主机路径、hostname、namespace、开始菜单入口或误装到隔离环境。

## 知识入口

AI 工具进入应用与隔离索引后，先从用户问题中识别一个子场景，再读取对应 knowledge；不要一次性读取全部安装知识。

| 子场景 | 快速判断 | 对应 knowledge |
| --- | --- | --- |
| 宿主机命令行安装 | 用户要安装 `.deb`、用 `apt`/`dpkg` 装系统包，或担心装进 KARE | `knowledge/applications/host-package-install.md` |
| AppImage | `.AppImage` 无法启动、架构不匹配、缺 `libfuse.so.2`、需要用户级集成 | `knowledge/applications/appimage.md` |
| aTrust/UEM 安全客户端 | aTrust 安装成功但虚拟网络、UEM、剪切板隔离或安全组件启动失败，或第三方内核模块导致启动异常 | `knowledge/applications/atrust-uem.md` |
| 桌面入口和图标 | 开始菜单图标异常、无效图标残留、用户级 `.desktop` 需要修复 | `knowledge/applications/desktop-entry-icon.md` |
| 第三方 apt 源 | `apt update` 报 `NO_PUBKEY`、源无签名、用户要删除源订阅 | `knowledge/applications/third-party-apt-source.md` |
| KARE/Kaiming 隔离 | 应用内 hostname 是 `kare`、路径来自 `/opt/kare`、namespace 或面板入口异常 | `knowledge/applications/isolation.md` |

- [`../knowledge/applications/README.md`](../knowledge/applications/README.md)

## 最小诊断

按子场景选择命令。

宿主机包安装：

```bash
mm-cli -s
command -v <app-command> || true
dpkg -l | rg -i '<package-or-app>' || true
readlink /proc/$$/ns/mnt
hostname
```

AppImage：

```bash
file "$HOME/下载/<app>.AppImage"
ldconfig -p 2>/dev/null | rg 'libfuse\.so\.2|libfuse3' || true
```

桌面入口：

```bash
rg -n '<app-name>|<desktop-id>' "$HOME/.local/share/applications" /usr/share/applications 2>/dev/null || true
desktop-file-validate "$HOME/.local/share/applications/<desktop-id>.desktop"
```

第三方 apt 源：

```bash
rg -n '<repo-domain>|<app-name>' /etc/apt /etc/apt/sources.list.d 2>/dev/null || true
sudo apt-get update
```

KARE/Kaiming 隔离：

```bash
readlink /proc/$$/ns/mnt
readlink /proc/$$/ns/uts
hostname
ps -ef | rg -i '<app-keyword>' | rg -v rg || true
```
