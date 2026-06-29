# 系统问题修复：UKUI 桌面

## 适用场景

- 开机自启动不生效或设置界面不显示。
- 全局搜索异常、软件商店结果异常、快捷键无法唤起。
- 右侧托盘隐藏/显示状态不持久、任务栏/面板异常。
- 锁屏/屏保/电源超时设置不生效，明明设置较长时间却很快进入锁屏。
- 输入法托盘显示异常、Fcitx5 主程序缺失、输入法默认项不符合预期。
- “打开方式 -> 选择其他应用”文件选择器中应用列表或图标异常。
- 系统截图进入选区模式后冻结画面变暗、变糊、清晰度下降，或截图工具在 Wayland 下捕获路径异常。
- 桌面 AI 组件残留、AI 子系统卸载。

## 知识入口

AI 工具进入 UKUI 修复索引后，先从用户问题中识别子场景，再读取对应 knowledge；不要一次性读取整个 UKUI 目录。

| 子场景 | 快速判断 | 对应 knowledge |
| --- | --- | --- |
| 开机自启动 | 开机后应用未启动，或设置界面不显示/不保存自启动项 | `knowledge/ukui/autostart.md` |
| 快捷键 | 全局搜索快捷键冲突、无法唤起或按键被占用 | `knowledge/ukui/keybindings.md` |
| 全局搜索异常 | 搜索结果来源异常、软件商店结果、搜索框状态或配置项异常 | `knowledge/ukui/search.md` |
| 系统托盘配置 | 右侧托盘显示/隐藏、顺序、折叠区、持久化异常 | `knowledge/ukui/system-tray.md` |
| 托盘源码级修复 | 配置级修复不能解决托盘行为，需要修改 `ukui-sni` | `knowledge/ukui/system-tray-source.md` |
| 锁屏超时 | 设置界面中的锁屏/关屏时间不生效，或电源设置与实际锁屏时间不一致 | `knowledge/ukui/screen-lock-delay.md` |
| 输入法 | Fcitx5、搜狗/讯飞输入法、输入法托盘显示或默认输入法异常 | `knowledge/ukui/input-method.md` |
| 打开方式文件选择器 | 选择其他应用时 `/usr/share/applications` 只显示少数 desktop 文件，或应用条目/图标异常 | `knowledge/ukui/file-dialog-open-with.md` |
| 桌面服务 | UKUI 服务启动、D-Bus activation、system-service-manager 异常 | `knowledge/ukui/system-service-manager.md` |
| 系统截图 | 点击截图后冻结画面变暗/变糊，或最终截图清晰度异常 | `knowledge/ukui/screenshot.md` |
| AI 组件 | 桌面 AI 组件、AI 子系统和残留清理 | `knowledge/ukui/ai-subsystem.md` |

- [`../knowledge/ukui/README.md`](../knowledge/ukui/README.md)

## 最小诊断

```bash
ps -ef | rg -i 'ukui|search|panel|tray|screensaver|power-manager' | rg -v rg
gsettings list-recursively org.ukui.search.settings 2>/dev/null || true
gsettings list-recursively org.ukui.screensaver 2>/dev/null || true
gsettings list-recursively org.ukui.power-manager 2>/dev/null || true
journalctl --user -b --no-pager | rg -i 'ukui|search|panel|tray|screensaver|power-manager|idle' | tail -n 120
```
