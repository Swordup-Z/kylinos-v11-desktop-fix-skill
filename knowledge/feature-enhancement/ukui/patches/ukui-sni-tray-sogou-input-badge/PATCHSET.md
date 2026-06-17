# UKUI 托盘搜狗中英状态角标 Patch 集

## 功能目标

在 UKUI 右侧托盘的搜狗输入法图标上显示“中/英”角标，用于表达搜狗输入法内部中文/英文状态。

## 类型与场景

- 需求类型：系统功能增强。
- 实际场景：UKUI 托盘。
- 知识入口：`knowledge/feature-enhancement/ukui/system-tray-sogou-input-badge.md`。

## 上游与基线

- 上游仓库：`https://gitee.com/openkylin/ukui-sni.git`
- 源码包：`ukui-sni`
- 相关二进制包：`ukui-widget-system-tray`
- 已验证系统包版本：`4.21.0.2-ok0.1k0.10`
- 已验证架构：`arm64`
- 基线节点：tag `build/4.21.0.2-ok0.1`，commit `3b2b1a6abaf2f1d5b083060b56bbfe69c04bad15`
- patch 文件：`0001-Add-Sogou-input-mode-tray-badge.patch`
- 验证状态：`runtime-verified`

## 套用策略

现场增强前先确认当前系统包版本：

```bash
dpkg-query -W -f='${binary:Package} ${Version} ${Source}\n' ukui-widget-system-tray
apt-cache policy ukui-widget-system-tray
```

获取并切换到匹配源码节点：

```bash
git clone https://gitee.com/openkylin/ukui-sni.git
cd ukui-sni
git fetch --all --tags
git checkout <current-system-matched-node>
```

套用：

```bash
git am <path-to>/0001-Add-Sogou-input-mode-tray-badge.patch
```

如果冲突，优先判断新版本是否已经提供输入法状态角标或等价接口。若没有，按语义迁移：

- 模型侧保留独立角色 `SogouInputBadge` 或等价只读角色。
- 状态来源仍以搜狗 CPIS `current_mode` 为准，不用键盘布局替代。
- 响应速度优先依赖 CPIS/Fcitx 事件信号，低频轮询只做兜底。
- QML 只负责显示角标，不承担状态判断。

## 安装目标

常见安装目标：

```text
/usr/lib/aarch64-linux-gnu/qt5/qml/org/ukui/systemTray/libukui-system-tray.so
/usr/share/ukui/widgets/org.ukui.systemTray/ui/TrayIcon.qml
/usr/share/ukui/widgets/org.ukui.systemTray/ui/TrayView.qml
/usr/share/ukui/widgets/org.ukui.systemTray/ui/FoldArea.qml
```

不同架构下插件库路径中的 multiarch 目录可能不同，应以 `dpkg -L ukui-widget-system-tray` 为准。

如果现场使用手动替换而不是重新打包安装 `.deb`，且要求包重装/升级后继续保留该功能，应为上述安装目标建立 `dpkg-divert` 本地转移：

```bash
dpkg-divert --local --rename --add --divert <target>.distrib <target>
install -m 0644 <custom-file> <target>
```

建立 diversion 前先确保 `<target>` 是官方文件，这样 `.distrib` 保存的是包管理器应维护的官方版本；随后再把定制文件安装回 `<target>`。撤销时先恢复官方文件，再 `dpkg-divert --rename --remove --divert <target>.distrib <target>`。

## 构建与验证

安装前至少验证：

```bash
readelf -d <build>/ukui-system-tray/libukui-system-tray.so | rg 'NEEDED|SONAME|RUNPATH|RPATH'
ldd <build>/ukui-system-tray/libukui-system-tray.so | rg 'not found' || true
```

运行时验证：

- 搜狗中文状态显示“中”。
- 搜狗英文状态显示“英”。
- 切换状态后无明显人为延迟；允许存在 CPIS/Fcitx 信号、D-Bus 异步查询和 QML 重绘造成的轻微延迟。
- `ukui-panel` 无 QML 加载错误、D-Bus 连接错误或崩溃。

## DATA 工作区映射

当前机器现场工作区示例：

```text
/data/usershare/kylinos-local-sources/<component-or-fix>/
```

现场工作区保存本机源码树、构建目录、回滚包和 `CUSTOMIZATION.md`；本目录只保存可复用 patch 和迁移策略。
