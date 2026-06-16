# AppImage 用户级安装与 FUSE

## 适用场景

- 用户下载 `.AppImage` 后需要安装到当前用户。
- AppImage 启动时报 `libfuse.so.2` 缺失。
- 需要为 AppImage 创建用户级开始菜单入口。

不适用于系统级 `.deb` 安装或第三方 apt 源清理。

## 安装位置

AppImage 通常不需要写入 `/usr`、`/opt` 或系统包数据库。优先放到：

```text
$HOME/Applications
```

桌面入口和图标放到：

```text
$HOME/.local/share/applications/<app>.desktop
$HOME/.local/share/icons/hicolor/<size>/apps/<app>.png
```

## 架构检查

ARM64 机器不要误装 x86-64 AppImage：

```bash
uname -m
dpkg --print-architecture
file "$HOME/下载/<app>.AppImage"
```

如果 `file` 显示 `x86-64`，而系统是 `aarch64`/`arm64`，不要创建桌面入口；应重新下载 arm64/aarch64 版本。

## FUSE 诊断

如果启动时报：

```text
dlopen(): error loading libfuse.so.2
AppImages require FUSE to run.
```

先诊断：

```bash
ldconfig -p 2>/dev/null | rg 'libfuse\.so\.2|libfuse3' || true
dpkg -l | rg '^ii\s+libfuse2(:\S+)?\s|^ii\s+libfuse3-3(:\S+)?\s' || true
apt-cache policy libfuse2 libfuse3-3
```

若确认缺少 `libfuse2`，且当前处于维护模式，安装：

```bash
sudo apt-get install libfuse2
```

安装后重新运行 AppImage，确认不再出现 `libfuse.so.2` 报错。

## 桌面入口

赋予执行权限：

```bash
chmod +x "$HOME/Applications/<app>.AppImage"
```

可用 AppImage 自带提取功能获取 desktop 文件和图标：

```bash
"$HOME/Applications/<app>.AppImage" --appimage-extract '*.desktop'
"$HOME/Applications/<app>.AppImage" --appimage-extract 'usr/share/icons/*'
```

创建或更新用户级 `.desktop` 后刷新：

```bash
update-desktop-database "$HOME/.local/share/applications"
gtk-update-icon-cache -q "$HOME/.local/share/icons/hicolor" 2>/dev/null || true
desktop-file-validate "$HOME/.local/share/applications/<app>.desktop"
```

只有用户明确要求系统级部署到 `/opt` 或 `/usr/share/applications` 时，才按维护模式流程处理。
