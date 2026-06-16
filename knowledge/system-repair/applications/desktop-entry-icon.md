# 用户级桌面入口与开始菜单图标

## 适用场景

- 手工安装应用或 AppImage 的桌面图标异常。
- 桌面文件管理器图标正常，但 UKUI 开始菜单中图标异常。
- 需要删除或修复无效开始菜单入口。

不适用于应用二进制安装失败或第三方 apt 源错误。

## 诊断

```bash
sed -n '1,120p' "$HOME/.local/share/applications/<app>.desktop"
find "$HOME/.local/share/icons" -iname '*<app>*' -print
desktop-file-validate "$HOME/.local/share/applications/<app>.desktop"
rg -n '<app>|<desktop-id>' "$HOME/.local/share/applications" /usr/share/applications 2>/dev/null || true
```

如果应用已固定到开始菜单收藏，还要检查收藏项：

```bash
rg -n '<app-name>|<desktop-id>' "$HOME/.config/ukui-menu/favorite.json" 2>/dev/null || true
```

## 图标修复

让 `.desktop` 使用标准图标名：

```ini
Icon=<app>
```

把图标放到用户级 `hicolor` 主题下：

```bash
mkdir -p "$HOME/.local/share/icons/hicolor/512x512/apps"
install -m 0644 "<source-icon>.png" "$HOME/.local/share/icons/hicolor/512x512/apps/<app>.png"
```

若 `$HOME/.local/share/icons/hicolor` 缺少 `index.theme`，开始菜单可能无法解析用户级图标。可以复制系统主题索引后刷新缓存：

```bash
mkdir -p "$HOME/.local/share/icons/hicolor"
install -m 0644 /usr/share/icons/hicolor/index.theme "$HOME/.local/share/icons/hicolor/index.theme"
gtk-update-icon-cache -f -q "$HOME/.local/share/icons/hicolor"
update-desktop-database "$HOME/.local/share/applications"
```

## 刷新 UKUI 菜单

```bash
pkill -x ukui-menu || true
systemd-run --user --collect --unit=ukui-menu-refresh \
  env DISPLAY=:0 WAYLAND_DISPLAY=wayland-0 XDG_SESSION_TYPE=wayland XDG_CURRENT_DESKTOP=UKUI \
  /usr/bin/ukui-menu
```

这类修复属于用户级配置，一般不需要维护模式；只有要写入 `/usr/share/icons`、`/usr/share/applications` 或安装系统包时，才按维护模式流程处理。
