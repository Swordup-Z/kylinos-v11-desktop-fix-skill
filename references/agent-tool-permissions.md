# AI 工具权限与运行边界

## 定位

这是 AI 工具用户级权限、默认运行模式、全局提示词和系统修复边界的分类入口。Codex 是当前已沉淀的具体实现。

## 适用场景

- 配置 AI 工具默认权限，例如 Codex full access。
- 解释 AI 工具权限与维护模式/root 权限边界。
- 配置工具级规则加载本 skill。

## 先读知识章节

- Codex 配置细节：[`../knowledge/agent-tools/codex-config.md`](../knowledge/agent-tools/codex-config.md)
- 多工具全局提示词：[`../knowledge/agent-tools/global-prompts.md`](../knowledge/agent-tools/global-prompts.md)

## 最小诊断

```bash
test -f "$HOME/.codex/AGENTS.md" && sed -n '1,160p' "$HOME/.codex/AGENTS.md"
```
