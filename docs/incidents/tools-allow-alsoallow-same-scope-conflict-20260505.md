# tools.allow与alsoAllow不能同scope共存

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

OpenClaw schema 不允许同一个 tools 对象里同时设置 allow 和 alsoAllow；根级、agent 级、sandbox 级是独立 scope。

## 正文

> 记录日期：2026-05-05  风险等级：L3

## 背景
nupai-oc-router 部署时 gateway startup failed，稳定性日志报 tools cannot set both allow and alsoAllow in the same scope。

## 内容
OpenClaw schema 不允许在同一个 tools 对象里同时设置 allow 和 alsoAllow。

3 层 scope：根级 tools 是全局白名单；agent 级 agents.list[0].tools 是单个 agent 的 policy；sandbox 级 agents.list[0].tools.sandbox.tools 是 sandbox 子环境 policy。

allow 表示完整白名单，只允许列出的工具；alsoAllow 表示在 profile 默认白名单基础上额外追加。错误写法是在同一个对象里同时配置 allow 和 alsoAllow。正确分层策略：根级 tools.allow 作为全局白名单；agent 级 tools.profile + alsoAllow 做 profile 基础扩展；sandbox 级 tools.sandbox.tools.alsoAllow 继续扩展。

判断标准：想“全集 + 限制”用 allow；想“预设 + 追加”用 profile + alsoAllow；同一对象内只能选一种。

## 结论
分层策略固定为：根级 tools.allow，agent 级 tools.profile + alsoAllow，sandbox 级 tools.sandbox.tools.alsoAllow。任一层只用一种关键字，绝不同 scope 共存。

## 影响
影响所有 OpenClaw 配置 tools 段落、新增 agent 配置、sandbox.mode = all 触发的 sandbox tools policy 和多 agent 配置。

## 下次注意
配置 tools 前先判断这层是全集白名单还是增量扩展；报 cannot set both allow and alsoAllow 立即查同 scope 是否混用；3 层 scope 独立，子层不继承父层 policy；profile 是 OpenClaw 预设工具集合。

