# 本地源码客制化索引

本文档用于记录所有“通过本地修改系统组件源码并替换系统产物解决问题”的索引规则。它不记录具体补丁细节；具体补丁仍放到对应组件或场景知识章节中，例如 `ukui-search-web-engine.md`。

## 适用场景

- 用户要求保留本地修改源码，方便后续继续迭代。
- 修复需要替换系统共享库、控制中心插件、系统服务或 schema。
- 同一个组件可能存在多次本地试装、回滚、正式保留和后续继续修改。
- 需要把源码、构建目录、回滚包和验证结果从聊天记录中抽离成可复用索引。

## 固定工作区

除必须由系统包管理器安装到默认系统路径的依赖组件外，源码、构建目录、staging、回滚包、符号对比文件和其他中间产物都应优先放到 DATA 分区共享目录：

```text
/data/usershare/kylinos-local-sources/
```

每个问题使用独立目录：

```text
/data/usershare/kylinos-local-sources/<component-or-fix>/
```

目录命名应体现组件和修复目标，例如：

```text
ukui-search-store-toggle
ukui-search-web-engine
ukui-panel-tray-persistence
```

不要把这类本地开发工作区放到 `$HOME`、桌面、下载目录或根分区临时目录；这样会挤占根分区，也不利于后续查找。

## 必备索引文件

每个本地源码客制化目录必须创建：

```text
/data/usershare/kylinos-local-sources/<component-or-fix>/CUSTOMIZATION.md
```

`CUSTOMIZATION.md` 是后续继续修改、回滚和判断系统差异的主索引。至少记录：

- 目标问题、期望行为和是否已经在系统中试装。
- 涉及的二进制包、源码包、关键系统文件和用户级配置文件。
- 源码来源、候选 git 节点、当前工作分支、精确匹配或近似匹配依据。
- 是否为本机客制化工作树；如果用户没有要求，不需要提交本地源码 git commit。
- 修改过的源码文件、新增资源文件、安装到系统的目标文件。
- 构建命令、构建目录、构建依赖和构建失败后的处理记录。
- 安装前 ABI、SONAME、NEEDED、RPATH/RUNPATH、导出符号、关键字符串验证结果。
- 试装时间、staged 文件、系统备份、用户级覆盖备份和回滚脚本路径。
- 安装后验证结果、仍需用户重启或手工验证的项目。
- 当前状态：`draft`、`tested`、`installed`、`rolled-back`、`kept-for-future`。

示例结构：

```text
/data/usershare/kylinos-local-sources/<component-or-fix>/
├── CUSTOMIZATION.md
├── <source-tree>/
├── build/
├── rollback/
│   └── <timestamp>/
│       ├── system-backup/
│       ├── user-backup/
│       ├── SHA256SUMS
│       └── restore.sh
└── notes/
```

`restore.sh` 必须和对应回滚包放在同一目录。重新生成 staged 文件、调整回滚脚本或补充备份后，必须重新生成并验证 `SHA256SUMS`。

## 全局索引

如果 `/data/usershare/kylinos-local-sources/README.md` 不存在，应创建它作为全局索引。该文件只记录每个本地客制化项目的导航信息，不承载长篇补丁过程：

```text
# KylinOS Local Source Customizations

| Project | Component | Status | Installed files | Rollback | Notes |
| --- | --- | --- | --- | --- | --- |
| <component-or-fix> | <package/source> | installed | /usr/lib/... | rollback/<timestamp>/restore.sh | CUSTOMIZATION.md |
```

每次新增、试装、回滚或迁移本地源码客制化项目时，都要同步更新全局索引和对应 `CUSTOMIZATION.md`。

## 保留与清理

- 用户明确要求后续继续修改时，保留源码树和必要构建目录；只清理重复构建缓存、下载缓存和不再需要的临时包。
- 如果问题已经通过官方包或配置级方案解决，且用户不需要保留源码，清理源码树和构建目录，只保留必要回滚记录或在 `CUSTOMIZATION.md` 标记为 `rolled-back`。
- 如果本地客制化修改已经正式安装并持续使用，保留最小回滚包；可继续留在项目目录 `rollback/<timestamp>/`，也可迁移到：

```text
/data/usershare/kylinos-local-sources/persistent-rollbacks/<component>/<timestamp>/
```

迁移后必须更新全局索引和 `CUSTOMIZATION.md` 中的回滚路径。

## 与 skill 知识库的关系

本地工作区保存“当前机器上实际源码、构建和回滚状态”；skill 仓库只保存通用规则和可复用方法。不要把以下内容写入 skill：

- 当前用户专属绝对路径、用户名、一次性日志。
- 未泛化的聊天过程。
- 只对某一次构建有效的临时文件名。
- 未验证的本地补丁细节。

如果从本地客制化中抽象出通用经验，应写到对应 `knowledge/source-rebuild/*.md` 或组件场景章节，并使用 `$HOME`、`<component-or-fix>`、`<timestamp>` 等占位符。
