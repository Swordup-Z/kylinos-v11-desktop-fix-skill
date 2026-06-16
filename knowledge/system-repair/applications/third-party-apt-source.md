# 第三方 apt 源残留或公钥缺失

## 适用场景

- `apt update` 因第三方源缺失公钥失败。
- 用户已卸载第三方应用，但源配置仍残留。
- 需要删除不再使用的软件源订阅。

不适用于普通 `.deb` 本地安装或 AppImage。

## 典型报错

```text
NO_PUBKEY <key-id>
仓库 “<repo-url>” 没有数字签名
```

## 诊断

先判断用户是否还需要该第三方应用。若用户明确要删除该应用，或系统中已经没有对应包、命令、桌面入口和进程，应清理残留 apt 源，而不是继续修复公钥。

```bash
dpkg -l | rg -i '<app-name>|<package-name>' || true
command -v <app-command> 2>/dev/null || true
find /usr/share/applications "$HOME/.local/share/applications" -maxdepth 1 -iname '*<app-name>*' -print 2>/dev/null
ps -ef | rg -i '<app-name>|<package-name>' | rg -v rg || true
rg -n '<repo-domain>|<app-name>' /etc/apt /etc/apt/sources.list.d 2>/dev/null || true
```

## 清理残留源

如果确认只剩源配置残留，且当前处于维护模式，可以删除对应源文件和已无用 keyring：

```bash
sudo rm /etc/apt/sources.list.d/<repo>.sources
sudo rm /usr/share/keyrings/<repo>-archive-keyring.gpg 2>/dev/null || true
sudo apt-get update
```

验证 `apt-get update` 不再访问该第三方源，也不再出现 `NO_PUBKEY` 或“没有数字签名”错误。

如果用户仍要保留该应用，应改为按厂商官方文档重新安装 keyring，并确认 `Signed-By=` 指向的 keyring 文件存在；不要在未确认来源时导入未知公钥。
