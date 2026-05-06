# tools.allow与alsoAllow不能同scope共存

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:00Z

## 摘要

分层策略：根级 tools.allow（全局白名单），agent 级 tools.profile \+ alsoAllow（profile 基础上扩展），sandbox 级 tools.sandbox.tools.alsoAllow（继续扩展）。任一层只用一种关键字，绝不同 scope 共存。

## 正文

> 记录日期：2026-05-05  风险等级：L3

## 背景
nupai-oc-router 部署时 gateway startup failed，稳定性日志报 tools cannot set both allow and alsoAllow in the same scope。

## 内容
OpenClaw schema 不允许在同一个 tools 对象里同时设置 allow 和 alsoAllow。3 层 scope 划分（每层独立，不互相继承）：   scope JSON 路径 用途    根级 tools tools 全局白名单  agent 级 tools agents.list[0].tools 单个 agent 的 policy  sandbox 级 tools agents.list[0].tools.sandbox.tools sandbox 子环境 policyallow vs alsoAllow 区别：allow — 完整白名单，只允许列出的工具alsoAllow — 在 profile 默认白名单基础上额外追加（增量）错误写法（同 scope 共存）：```PLAIN_TEXT
"agents.list[0].tools": {
  "allow": ["bash"],
  "alsoAllow": ["dispatch_worker"]   // ❌ 同 scope 报错
}
```正确写法（分层策略，来自实际生效配置）：根级用 allow（全局白名单）：```PLAIN_TEXT
"tools": {
  "allow": ["bash", "dispatch_worker", "query_task_status", "confirm_task_plan", ...]
}
```agent 级用 profile \+ alsoAllow（基于预设 profile 增量扩展）：```PLAIN_TEXT
"agents.list[0].tools": {
  "profile": "coding",
  "alsoAllow": ["dispatch_worker", "query_task_status", "confirm_task_plan"]
}
```sandbox 级只用 alsoAllow（基于父层增量）：```PLAIN_TEXT
"agents.list[0].tools.sandbox.tools": {
  "alsoAllow": ["dispatch_worker", "query_task_status", "confirm_task_plan"]
}
```判断标准：想"全集 \+ 限制" → 用 allow想"预设 \+ 追加" → 用 profile \+ alsoAllow同一对象内只能选一种

## 结论
分层策略：根级 tools.allow（全局白名单），agent 级 tools.profile \+ alsoAllow（profile 基础上扩展），sandbox 级 tools.sandbox.tools.alsoAllow（继续扩展）。任一层只用一种关键字，绝不同 scope 共存。

## 影响
所有 OpenClaw 配置 tools 段落；新增 agent 时 tools 配置；sandbox.mode = "all" 触发的 sandbox tools policy；多 agent 配置时每个 agent 各自的 tools 段。

## 下次注意
配置 tools 前先想"这层是全集白名单还是增量扩展"，二选一；报 cannot set both allow and alsoAllow 立即查同 scope 是否混用；3 层 scope 是独立的，子层不继承父层 policy（要单独配）；profile 是 OpenClaw 预设的工具集合（如 coding），不是自定义命名。

