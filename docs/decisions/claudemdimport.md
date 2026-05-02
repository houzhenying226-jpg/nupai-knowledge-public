# CLAUDE.md@import三行兜底

> 类型：架构决策 | 项目：_global | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

项目级 CLAUDE.md（如 nupai-crm/CLAUDE.md、nupai-store/CLAUDE.md）末尾加 @import 引用之外，再加 3 行兜底注释，让 AI 在 import 不生效时也能强制执行起手式：① cat docs/STATUS.md ② cat docs/index.md ③ memory MCP search_nodes 当前任务实体。

## 正文

> 记录日期：2026-04-29  风险等级：L2

## 背景
v1.1 方案 4.4 节让项目级 CLAUDE.md 用 @import 引用 nupai-knowledge/CLAUDE.md。Claude Code 社区有反馈 import 偶发不生效，可能让起手式失效。

## 内容
Claude Code 的 CLAUDE.md 加载机制是按 cwd 向上查找。@import 语法支持引入其他文件。但社区报告 import 偶发被忽略（具体原因未明）。如果完全依赖 import，会出现"AI 没起手式但你以为有"的隐性故障。

## 结论
项目级 CLAUDE.md（如 nupai-crm/CLAUDE.md、nupai-store/CLAUDE.md）末尾加 @import 引用之外，再加 3 行兜底注释，让 AI 在 import 不生效时也能强制执行起手式：① cat docs/STATUS.md ② cat docs/index.md ③ memory MCP search_nodes 当前任务实体。

## 影响
所有用 @import 引用其他 CLAUDE.md 的项目，都该加 3 行兜底。NuPai 项目级 CLAUDE.md 标准模板含此结构。

## 下次注意
任何关键自动化逻辑都要有"兜底"，不要 100% 依赖单一机制。@import 是好用，但要假设它可能失效，给出降级路径。

