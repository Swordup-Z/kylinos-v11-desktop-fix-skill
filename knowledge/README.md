# Knowledge 结构说明

`knowledge/` 保存具体可复用系统修复知识，包括背景、诊断、修复步骤、验证、回滚和清理规则。

功能增强、本地客制化和默认行为调整已拆分到 `$HOME/.os-enhance-skill`。

## 场景分类

- `system/`：维护模式、系统服务、系统体检噪声。
- `applications/`：应用安装、AppImage、KARE/Kaiming 隔离。
- `ukui/`：UKUI 自启动、全局搜索异常、托盘、快捷键、AI 组件、系统服务。
- `network/`：代理、TUN、Clash Verge。
- `hardware/`：指纹、生物识别、图形频率。
- `storage/`：分区、挂载、DATA、overlay。
- `agent-tools/`：系统修复中的 AI 工具权限边界。
- `source-rebuild/`：必须源码重编译才能修复的系统问题。

## 渐进式披露

加载顺序：

```text
SKILL.md
-> references/<scenario>.md
-> knowledge/<scenario>/README.md
-> knowledge/<scenario>/<topic>.md
```

`references/` 只做实际修复场景路由；场景 `README.md` 只做特定分类和具体实例索引；具体 `<topic>.md` 才记录处理过程。新增经验时先判断场景，再放入对应目录；如果没有合适章节，新建一个最小主题文件，并从同场景 README 链接过去。

通过源码修改实现的可复用修复，patch 集放在同场景目录下：

```text
knowledge/<scenario>/patches/<fix-id>/PATCHSET.md
knowledge/<scenario>/patches/<fix-id>/*.patch
```

只有需要在源码树中复用或迁移该功能时才读取 patch 集。

每次只读取最小必要链路。不要因为同一场景下存在多个主题文件，就把整个场景目录全部读入上下文。

如果本索引和场景索引没有命中具体章节，停止继续加载 knowledge。不要通过 `find`、`rg` 或目录遍历把全库经验读入上下文；先基于已收集的系统证据诊断，确认产生可复用经验后再新增最小章节。

## 章节要求

具体知识章节应尽量包含：

- 适用场景。
- 关键系统文件、服务、包名或源码位置。
- 先诊断、再修改、最后验证的流程。
- 持久化策略。
- 回滚或恢复方式。
- 中间产物清理规则。
- 不应继续操作的风险信号。

不要把一次性聊天过程、长篇日志、当前用户名或不可复用临时路径写入 knowledge；需要用户目录时使用 `$HOME`、`<user>`、`<component-or-fix>` 等占位符。
