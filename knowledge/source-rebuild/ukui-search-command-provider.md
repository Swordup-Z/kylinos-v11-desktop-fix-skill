# UKUI 全局搜索自定义命令 Provider

## 适用场景

用户希望在 UKUI 全局搜索中执行可配置的系统动作或自定义命令，例如“清空回收站”。原生设置界面通常没有“添加自定义命令”的入口；全局搜索内部虽然支持插件和结果动作，但需要通过 `libukui-search` 的搜索插件体系扩展。

如果只是屏蔽软件商店结果或调整已有设置，先读 [`../../knowledge/ukui/search.md`](../../knowledge/ukui/search.md)。如果要新增通用 provider、替换系统库或重新构建 `ukui-search`，必须先读 [`../../references/source-rebuild.md`](../../references/source-rebuild.md)。

## 设计原则

- 优先做通用 provider，不要为每个动作都硬编码 C++ 分支。
- provider 从用户级配置读取命令项，新增命令时优先通过设置界面的图形入口修改配置；没有图形入口时再手工编辑配置文件。
- 配置使用 `command` + `args` 数组，由 `QProcess` 直接执行，不经过 shell。
- 默认不支持 shell 字符串拼接；如确需复杂 shell，应由用户明确配置 `command=/bin/sh`、`args=["-c", "..."]` 并承担注入风险。
- 为了避免在配置中硬编码用户名，可以支持 `~`、`$HOME`、`${HOME}` 在 `command`、`args`、`workingDirectory` 单个 token 内展开为当前用户目录；该展开不是 shell，不处理任意环境变量、通配符、管道或重定向。
- 破坏性命令应支持 `confirm=true` 和 `confirmMessage`，执行前弹确认。
- provider 应作为全局搜索普通搜索插件注册，保留插件启停、排序和 best match 行为。
- 如果用户觉得手写 JSON 麻烦，推荐在 `search-ukcc-plugin` 的全局搜索设置页增加“自定义命令”入口，打开图形化配置对话框，直接读写同一个 `custom-commands.json`。

## 用户级配置格式

推荐配置路径：

```text
$HOME/.config/org.ukui/ukui-search/custom-commands.json
```

示例：

```json
{
  "version": 1,
  "commands": [
    {
      "id": "empty-trash",
      "name": "清空回收站",
      "description": "永久删除回收站中的所有项目",
      "keywords": ["清空回收站", "回收站", "垃圾桶", "empty trash", "trash"],
      "command": "gio",
      "args": ["trash", "--empty"],
      "icon": "user-trash",
      "confirm": true,
      "confirmMessage": "是否永久删除回收站中的所有项目？",
      "detached": false,
      "showInBestMatch": true,
      "timeout": 30000
    }
  ]
}
```

字段建议：

- `id`：稳定唯一 ID，用作 `actionKey`。
- `name`：结果显示名称。
- `description`：详情页描述和 tooltip。
- `keywords`：匹配关键词，建议包含中文名、别名和英文别名。
- `command`：可执行文件名或绝对路径。
- `args`：参数数组。
- `icon`：XDG 图标名。
- `confirm` / `confirmMessage`：破坏性命令确认。
- `detached`：是否后台启动。
- `workingDirectory`：可选工作目录。
- `timeout`：非 detached 命令等待超时。
- `showInBestMatch`：是否进入最佳匹配。

可读性建议：

- 路径参数优先写成 `$HOME/下载`、`~/Documents` 这类形式，不要硬编码 `/home/<user>/...`。
- 每个参数仍然是数组中的一个元素；不要写成一整段 shell 字符串。
- 需要打开目录这类动作时，可配置为 `command=xdg-open`、`args=["$HOME/下载"]`、`detached=true`。
- 如果已经实现设置页图形化入口，该入口保存时应使用缩进后的 JSON，保留未知顶层字段，并只更新 `commands` 数组；这样既方便人工审阅，也避免破坏未来扩展字段。

## 最小源码修改位置

常见源码目录：

```text
libsearch/commandsearch/command-search-plugin.cpp
libsearch/commandsearch/command-search-plugin.h
libsearch/CMakeLists.txt
libsearch/pluginmanage/search-plugin-manager.cpp
libsearch/pluginmanage/search-plugin-manager.h
translations/libukui-search/libukui-search_zh_CN.ts
search-ukcc-plugin/search.cpp
search-ukcc-plugin/search.h
search-ukcc-plugin/translations/zh_CN.ts
```

实现要点：

- `CommandSearchPlugin` 实现 `SearchPluginIface`。
- `KeywordSearch()` 读取 JSON 配置并匹配 `keywords`。
- `openAction()` 根据 `id` 找到命令项并执行。
- 详情页显示名称、描述和“运行”动作。
- 首次配置不存在时可创建默认配置，但不要覆盖用户已有配置。
- 在 `SearchPluginManager` 默认插件顺序中加入 `Command Search`，通常放在 `Web Page` 前。
- 对已有用户配置做增量迁移：旧配置存在但缺少 `Command Search` 时，应追加该插件，而不是因为版本号相同就跳过。
- 如需图形化配置，在 `search-ukcc-plugin/search.cpp` 中增加“自定义命令”设置项和配置对话框：
  - 左侧显示命令列表。
  - 右侧编辑 `id`、`name`、`description`、`keywords`、`command`、`args`、`icon`、`workingDirectory`、`timeout`、`confirm`、`confirmMessage`、`detached`、`showInBestMatch`。
  - `keywords` 和 `args` 可用“每行一个值”的文本框，比直接编辑 JSON 数组更适合普通用户。
  - 保存前验证 `id`、`name`、`command` 非空且 `id` 唯一。
  - 使用 `QSaveFile` 或等价方式原子写入，避免配置文件写一半导致全局搜索 provider 无法解析。
  - 图形入口仍然读写用户级配置，不需要提权，不应写入 `/usr` 或系统级配置。

## 构建与安装验证

按照源码重编译通用流程构建，安装前至少验证：

```bash
readelf -d <build>/libsearch/libukui-search.so.2.3.0 | rg 'NEEDED|SONAME|RUNPATH|RPATH|FLAGS'
readelf -d /usr/lib/<arch>/libukui-search.so.2.3.0 | rg 'NEEDED|SONAME|RUNPATH|RPATH|FLAGS'

nm -D --defined-only /usr/lib/<arch>/libukui-search.so.2.3.0 | awk '{print $3}' | sort > /tmp/ukui-search.system.syms
nm -D --defined-only <build>/libsearch/libukui-search.so.2.3.0 | awk '{print $3}' | sort > /tmp/ukui-search.new.syms
comm -23 /tmp/ukui-search.system.syms /tmp/ukui-search.new.syms
comm -13 /tmp/ukui-search.system.syms /tmp/ukui-search.new.syms | c++filt

strings <build>/libsearch/libukui-search.so.2.3.0 | rg 'Command Search|custom-commands.json'
strings <build>/search-ukcc-plugin/libsearch-ukcc-plugin.so | rg 'Custom commands|One keyword per line|Require confirmation'
```

可接受结果：

- 无 `RPATH`/`RUNPATH`。
- `SONAME` 与系统库一致。
- `NEEDED` 集合无非预期变化。
- 系统库已有导出符号无缺失。
- 新增符号仅来自新增 provider 或预期辅助函数。

安装前准备回滚包：

```text
/data/usershare/kylinos-local-sources/<component-or-fix>/rollback/<timestamp>/
├── install.sh
├── restore.sh
├── system-backup/
├── user-backup/
├── staged/
└── SHA256SUMS
```

系统级安装必须在维护模式中执行：

```bash
mm-cli -s
pkexec <rollback>/<timestamp>/install.sh
```

## 安装后验证

验证系统库已经替换：

```bash
sha256sum /usr/lib/<arch>/libukui-search.so.2.3.0 <rollback>/<timestamp>/staged/usr-lib/libukui-search.so.2.3.0
strings /usr/lib/<arch>/libukui-search.so.2.3.0 | rg 'Command Search|custom-commands.json'
```

验证动态链接：

```bash
timeout 8s /usr/bin/ukui-search --quit
```

触发一次全局搜索前端加载，再检查日志：

```bash
timeout 10s /usr/bin/ukui-search
tail -n 120 "$HOME/.log/ukui-search/ukui-search-0.log" | rg 'register search plugin|Command Search|undefined symbol|symbol lookup'
```

预期出现：

```text
register search plugin:  "Command Search"
```

验证用户级配置：

```bash
rg -n 'Command%20Search|Web%20Page' "$HOME/.config/org.ukui/ukui-search/ukui-search-plugin-order.conf"
rg -n 'empty-trash|清空回收站|gio|trash' "$HOME/.config/org.ukui/ukui-search/custom-commands.json"
jq . "$HOME/.config/org.ukui/ukui-search/custom-commands.json" >/dev/null
```

功能最终验证需要在图形界面中打开全局搜索，输入命令关键词，例如：

```text
清空回收站
```

应出现 `Command Search` 结果，点击后弹确认框；确认后执行配置中的命令。

如果实现了设置页图形化入口，继续验证：

- 打开“设置 -> 全局搜索”，应能看到“自定义命令”入口。
- 点击“配置”后能看到命令列表和编辑表单。
- 保存后 `custom-commands.json` 是格式化 JSON。
- 使用 `$HOME/下载`、`~/Downloads` 这类参数时，全局搜索执行动作前应展开到当前用户目录。

## 回滚

回滚脚本应恢复系统库和翻译文件，并恢复或删除本次修改过的用户级配置。

如果 `custom-commands.json` 是本次新增的文件，回滚时应删除它；不要把它错误地当作原始用户配置恢复。若它原本存在，则应在 `user-backup/` 保存原文件并恢复。

回滚示例：

```bash
pkexec <rollback>/<timestamp>/restore.sh
```

回滚后验证：

```bash
sha256sum -c <rollback>/<timestamp>/SHA256SUMS
strings /usr/lib/<arch>/libukui-search.so.2.3.0 | rg 'Command Search|custom-commands.json' || true
```

## 风险点

- 这是系统库替换，必须在维护模式中安装，并保留回滚脚本。
- 如果已有用户级插件顺序配置缺少新插件，provider 可能编译进库但不注册；需要做旧配置增量迁移或显式追加 `Command Search`。
- `ukui-search --quit` 只验证动态链接，不一定完整加载搜索插件；要验证 provider 注册，需要启动一次前端并检查日志。
- 不要直接执行 shell 拼接命令；优先使用 `QProcess(command, args)`。
- 破坏性命令必须默认弹确认。
