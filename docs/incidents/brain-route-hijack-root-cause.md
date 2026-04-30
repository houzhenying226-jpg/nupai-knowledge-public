# Brain 抢路由真根因与 before_dispatch 修法

> 类型：故障复盘 | 项目：crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

OpenClaw brain 插件在 before_dispatch 阶段劫持正常路由，导致所有 GPT-5.5 请求被重定向。根因：brain 插件 hook 优先级高于路由器。修法：route_passthrough: true 或降低 hook 优先级。

## 正文

# Brain 抢路由真根因与 before_dispatch 修法

## 问题
brain 插件注册全局 before_dispatch hook（priority:10），高于主路由器（priority:50），劫持所有入站消息。

## 修法
在 openclaw.json plugins.brain 添加 route_passthrough: true。

## 验证
重启网关后发测试消息，确认路由到 agent-main 而非 brain。

## 下次注意
- 新增插件检查 before_dispatch hook 优先级
- brain 激活时测试正常消息路由
