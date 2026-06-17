# UKUI 托盘搜狗中英状态角标

可复用 patch 集保存在 [`patches/ukui-sni-tray-sogou-input-badge/PATCHSET.md`](patches/ukui-sni-tray-sogou-input-badge/PATCHSET.md)。只有需要在 UKUI 托盘上显示搜狗输入法内部中英状态时才读取 patch 集。

## 适用场景

- 用户需要从右侧托盘图标直接判断搜狗输入法当前是中文状态还是英文状态。
- 原始托盘图标已经能显示输入法图标，但不能稳定表达搜狗内部中英模式。
- 需求是“状态展示增强”，不是修复 Fcitx 输入法切换失败，也不是修改托盘隐藏区排序。

## 状态来源

不要用系统键盘布局或 Fcitx 当前输入法名称直接判断搜狗内部中英状态。搜狗内部模式应从 CPIS 面板服务读取：

```bash
busctl --user call com.cpis.panel /com/cpis/panel com.cpis.panel AcquireEngineStat ss "$USER#im.module=fcitx5|im.name=com.sogou.ime.ng.fcitx5.kylin" current_mode
```

常见映射：

- `kb_zh`、`@zh_cn`、`26keyZhong`：显示“中”。
- `kb_en`、`@en_us`、`26keyEnglish`、`invalid`：显示“英”。

`invalid` 可能表示搜狗内部英文直通状态；不要直接当作未知状态丢弃。

## 最小实现

在 `ukui-sni` 的 `ukui-system-tray` 中新增一个只供 QML 使用的模型角色，例如 `SogouInputBadge`：

1. 只对 `Id=Fcitx` 且图标名包含 `sogou` 的托盘项返回角标，避免普通键盘布局图标误显示。
2. 通过用户会话 D-Bus 异步调用 `com.cpis.panel.AcquireEngineStat(<uid>, "current_mode")` 获取状态。
3. 优先监听 `com.cpis.engine.Event` 和 `com.cpis.panel.UpdateUi/Preedit/Commit`，收到搜狗 UID 相关信号后立即刷新。
4. 保留低频 `QTimer` 兜底，避免信号遗漏后状态长期不更新；不要依赖高频轮询解决响应速度。
5. 在 `TrayIcon.qml` 绘制右下角小角标，并由 `TrayView.qml`、`FoldArea.qml` 传入模型角色。

事件驱动后仍可能有轻微延迟，通常来自 CPIS/Fcitx 发信号、D-Bus 异步查询和 QML 重绘链路；不要额外加入人为延迟。

## 构建与验证

构建前先按源码重编译通用流程确认维护模式、源码节点和 ABI 风险。构建示例：

```bash
cmake -S /data/usershare/kylinos-local-sources/<component-or-fix>/ukui-sni \
      -B /data/usershare/kylinos-local-sources/<component-or-fix>/build \
      -DCMAKE_BUILD_TYPE=Release -DCMAKE_SKIP_RPATH=ON
cmake --build /data/usershare/kylinos-local-sources/<component-or-fix>/build --target ukui-system-tray -j$(nproc)
```

安装前至少检查：

```bash
readelf -d <build>/ukui-system-tray/libukui-system-tray.so | rg 'NEEDED|SONAME|RUNPATH|RPATH'
ldd <build>/ukui-system-tray/libukui-system-tray.so | rg 'not found' || true
```

运行时验证：

1. 替换插件库和相关 QML 文件后重启 `ukui-panel`。
2. 确认搜狗中文状态显示“中”。
3. 按搜狗内部中英切换键，确认英文状态显示“英”。
4. 观察 `journalctl --user` 中是否有 `ukui-panel`、QML 或 D-Bus 相关错误。

## 持久化边界

该功能修改的是 `ukui-widget-system-tray` 管理的系统插件和 QML 文件。普通手动替换可以跨重启生效，但不能防止后续包重装、包升级或系统修复把文件覆盖回官方版本。若用户要求“重装/升级后仍保留角标”，应对安装目标建立 `dpkg-divert` 本地转移：官方文件放到 `<target>.distrib`，定制文件保留在 `<target>`。

验证：

```bash
dpkg-divert --list | rg 'ukui-system-tray|org.ukui.systemTray|TrayIcon|TrayView|FoldArea'
dpkg -V ukui-widget-system-tray
```

`dpkg -V` 在 diversion 正确时不应把这些原路径报成包校验异常；因为包管理器视角中的官方文件已经转移到 `.distrib`。后续系统包升级后，应检查 `.distrib` 是否变化，并判断是否需要基于新官方版本重新合并 patch。

## 回滚

手动替换系统插件前必须准备 `system-backup/`、`staged/`、`SHA256SUMS` 和同目录 `restore.sh`。如出现面板崩溃、托盘缺失或 QML 加载错误，执行回滚脚本并重启面板。
