# Qt/UKUI 应用界面布局

## 适用场景

- 在 KylinOS Desktop V11 / UKUI 上开发 Qt Widgets 桌面维护工具。
- 表格首行或所有行文字显示不全、被固定行高裁剪。
- 语言下拉框、组合框或下拉列表在 UKUI 主题下文字颜色与背景冲突，看不到当前选项。

## 诊断要点

- 检查 `QTableWidget` 是否被父布局挤压到只剩表头高度。卡片式页面里如果所有内容直接放进一个 `QVBoxLayout`，窗口高度不足时表格行会被压到不可见。
- 检查 `QTableWidget` 是否使用了过低的固定 `verticalHeader()->setDefaultSectionSize()`。
- 检查是否使用了 `ResizeToContents + setWordWrap(true) + resizeRowsToContents()`。在 UKUI 主题、高 DPI 或复杂 stylesheet 下，Qt 可能误算行高，表现为巨大空白、只显示少数行或列表像被强行截断。
- 检查是否所有列都设置为 `QHeaderView::Stretch`。长应用 ID、版本、路径和中文说明在窄列中会被压缩，若同时使用固定行高就会裁剪。
- 检查是否没有设置 `setTextElideMode(Qt::ElideNone)`，以及单元格没有 tooltip。
- 检查 `QComboBox` 是否依赖系统主题默认颜色。UKUI 深浅主题或 token 样式可能覆盖组合框前景色、下拉列表背景色和选中态。
- 检查 `QTableWidget::item` 是否显式设置了 `color`。UKUI 深色主题可能把普通单元格文字继承成浅色，导致白底表格中未选中行看起来像没有内容。

## 推荐实现

- 表格统一配置：
  - 对维护工具的列表类表格优先使用 `setWordWrap(false)`，用横向滚动和 tooltip 展示长路径、应用 ID。
  - `setTextElideMode(Qt::ElideNone)`
  - `verticalHeader()->setSectionResizeMode(QHeaderView::Fixed)`
  - `verticalHeader()->setDefaultSectionSize(48 或 50)`
  - `horizontalHeader()->setMinimumSectionSize(...)`
  - 按列设置 `ResizeToContents` 与 `Stretch`，不要所有列都无差别 `Stretch`。
- 页面主体内容较多时，把除自定义标题栏之外的主体放入 `QScrollArea`，给关键表格设置足够的 `setMinimumHeight(...)`，不要让父布局为了适配窗口高度压缩表格行。
- 不要在这种场景里默认调用 `resizeRowsToContents()`；如果真实截图中出现巨大空白或行数异常，改回固定行高。
- 长文本单元格同时写入 `setToolTip(text)`，让用户鼠标悬停可查看完整内容。
- 路径、应用 ID、说明列优先使用 `Stretch` 或允许横向滚动；状态、大小、数量列优先 `ResizeToContents`。
- 在应用级 stylesheet 中显式设置 `QTableWidget` 和 `QTableWidget::item` 的 `background`、`color`、`selection-background-color`、`selection-color`，不要依赖 UKUI 当前主题。
- 如果要做悬停反馈，优先实现整行 hover/selection 背景条；避免每个单元格单独高亮造成“按列分块”的视觉效果。
- `QComboBox` 在应用级 stylesheet 中显式设置：
  - 当前框 `background`、`color`、`border`
  - `QComboBox QAbstractItemView` 的 `background`、`color`、`selection-background-color`、`selection-color`
  - 必要时设置 `setMinimumWidth()` 和 `setSizeAdjustPolicy(QComboBox::AdjustToContents)`

## 验证

```bash
cmake -S . -B build -G Ninja
cmake --build build
timeout 8s env QT_QPA_PLATFORM=offscreen ./build/<app>
```

离屏启动只验证进程和样式不会崩溃；真实裁剪、悬停和语言栏可见性必须在 UKUI 桌面中打开应用并截图确认。Wayland 原生窗口可能不受 `wmctrl` 控制；需要自动置前截图时，可临时用 `QT_QPA_PLATFORM=xcb` 启动测试实例，再用 `wmctrl -ia <window-id>` 激活。
