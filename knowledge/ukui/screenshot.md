# UKUI 系统截图清晰度与 Wayland 捕获路径

## 适用场景

- 点击系统截图按钮后，当前界面进入固定/冻结状态，同时显示变暗、变糊或清晰度下降。
- 手动选区截图时预览层清晰度下降，但保存后的图片可能正常。
- 保存后的截图也低清晰度、尺寸异常或内容缩放异常。

## 背景判断

KylinOS Desktop V11 的 UKUI Wayland 会话通常由 `kylin-wlcom` 提供合成器，截图工具为 `kylin-screenshot`。手动截图进入选区模式时，工具会基于当前画面生成冻结帧，并叠加遮罩、边框和选区交互层。这个覆盖层可能让用户看到的桌面临时变暗或变糊；如果保存后的图片分辨率正常，这通常是截图交互效果，不代表屏幕分辨率或显示设置被改低。

如果日志出现 `Screencopy is disabled by environment, set env QT_WAYLAND_USE_WLR_SCREENCOPY=1 to enable`，说明截图工具在 Wayland 下没有启用原生 screencopy 路径，可能退回兼容捕获、冻结帧或缩放路径。此时选区界面的显示质量更容易下降；若最终输出图片也模糊，再按异常处理。

## 最小诊断

确认会话、截图工具和屏幕输出：

```bash
printf 'XDG_SESSION_TYPE=%s\nDESKTOP_SESSION=%s\nXDG_CURRENT_DESKTOP=%s\n' \
  "$XDG_SESSION_TYPE" "$DESKTOP_SESSION" "$XDG_CURRENT_DESKTOP"
command -v kylin-screenshot || true
apt-cache policy kylin-screenshot 2>/dev/null || true
xrandr --verbose 2>/dev/null | sed -n '1,80p' || true
```

查看截图配置和日志：

```bash
gsettings list-recursively org.ukui.screenshot 2>/dev/null || true
gsettings list-recursively org.ukui.screencap 2>/dev/null || true
tail -n 160 "$HOME/.log/kylin-screenshot.log" 2>/dev/null || true
journalctl --user -b --no-pager \
  | rg -i 'kylin-screenshot|screenshot|portal|wlcom|screencopy|scale' \
  | tail -n 120
```

区分正常交互效果和真实异常：

- 只在选区模式变暗/变糊，保存后的 PNG 尺寸和内容清晰：通常是遮罩和冻结帧效果，不需要系统级修复。
- 保存后的图片也低清晰度、尺寸小于屏幕或文字明显糊：优先检查 Wayland screencopy 环境、缩放、截图工具版本和 portal 后端。
- 日志反复出现 `QImage::setPixel: coordinate ... out of range`：记录为截图工具绘制覆盖层的异常线索；只有最终截图异常或程序卡死时才继续修复。

## 处理建议

先不要修改 `/usr/share/applications/kylin-screenshot.desktop`。若只需验证 screencopy 路径，可临时从终端启动：

```bash
QT_WAYLAND_USE_WLR_SCREENCOPY=1 kylin-screenshot gui
```

如果临时启动后最终截图清晰度恢复，再考虑用用户级 `.desktop` 覆盖或启动脚本持久化环境变量。持久化优先放在 `$HOME/.local/share/applications/` 或用户级 wrapper 中；只有确认必须改系统 desktop 文件时，才进入维护模式后修改系统路径。

当 `/usr/share/applications/kylin-screenshot.desktop` 中的 `Exec=` 使用未带绝对路径的 `kylin-screenshot`，且当前桌面会话 `PATH` 中 `$HOME/.local/bin` 位于 `/usr/bin` 之前时，可以使用用户级 wrapper 持久化：

```bash
mkdir -p "$HOME/.local/bin"
cat > "$HOME/.local/bin/kylin-screenshot" <<'EOF'
#!/bin/sh
export QT_WAYLAND_USE_WLR_SCREENCOPY=1
exec /usr/bin/kylin-screenshot "$@"
EOF
chmod 755 "$HOME/.local/bin/kylin-screenshot"
```

应用该方案后，关闭旧的截图进程，再重新从开始菜单、快捷键或按钮启动截图工具：

```bash
ps -ef | rg -i 'kylin-screenshot' | rg -v rg || true
kill <old-kylin-screenshot-pid>
command -v kylin-screenshot
```

如果最终截图清晰、只是选区阶段看起来变糊，不建议做修复。该现象属于截图工具为了选区交互叠加遮罩/冻结帧后的视觉效果。

## 验证

- 截图选区模式能正常进入和退出。
- 保存后的图片分辨率等于选区实际尺寸，文字和边缘清晰。
- 截图日志不出现新的崩溃、D-Bus 失败或 portal 失败。
