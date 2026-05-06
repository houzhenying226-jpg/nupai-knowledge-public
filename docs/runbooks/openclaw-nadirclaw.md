# OpenClaw 三模型自动升级路由接入 NadirClaw

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-06T02:41:58Z

## 摘要

OpenClaw 已切到 nadirclaw/auto，普通开发任务会自动走 DeepSeek，中文文档任务走 Qwen，告警链路已实测；高风险任务会命中 GPT 车道规则，但 NadirClaw 直连 openai-codex/gpt-5.5 仍无法真正落到 OpenClaw 原生 GPT 通道，会自动降级到 DeepSeek。

## 正文

> 记录日期：2026-05-06  风险等级：L2

## 背景
用户希望在 OpenClaw 内实现三模型自动分配，不再手动切换 Qwen / DeepSeek / GPT-5.5。
核心原则是“更好的模型放在最好的位置”，并要求部署后必须测试闭环，再整理成知识卡。

## 内容
本次采用 NadirClaw 作为 OpenClaw 前置路由层，配置为 simple=Qwen、mid=DeepSeek、complex=GPT-5.5。
补丁点有三类：
1. 路由判定只看用户消息，不再把 OpenClaw 系统提示里的 deploy / rollback / production 等词误判成高风险。
2. 对 OpenRouter 兼容参数做裁剪，reasoning_effort 不再发给 Qwen / DeepSeek，避免 UnsupportedParamsError。
3. 对带 tools 的 streaming 请求改成“上游缓冲、下游仍返回 SSE”，解决 OpenClaw 将成功响应误判为 incomplete terminal response 的问题。
部署后已将 OpenClaw 默认模型切为 nadirclaw/auto，NadirClaw 常驻在 ~/.nadirclaw 下，由 launchd 托管；并新增 router-alerts 脚本与 LaunchAgent，用于全链路失败、频繁降级、GPT 高风险车道降级的飞书告警。

## 结论
主链路已闭环。
验证结果：
1. OpenClaw 端到端 smoke（“只回复 OK”）已成功由 NadirClaw 路由到 openrouter/deepseek/deepseek-v4-pro，且 OpenClaw 不再记录 fallbackAttempts。
2. 高风险专项测试已命中 high_risk_override，优先尝试 openai-codex/gpt-5.5；第一次失败是 public OpenAI Responses API 的 scope 缺口（缺少 api.responses.write）。
3. 后续将 openai-codex 改成直打 `https://chatgpt.com/backend-api/codex` 后，scope 错误消失，但先后命中过 Cloudflare challenge 和 NotFoundError，说明问题已经从“token scope”推进到了“受信任客户端 / OpenClaw 原生传输链路”层面。
4. 飞书测试告警已触发，alerts.log 留痕为“[OpenClaw][NadirClaw][P1] TEST degraded-request alert”。
因此当前可用状态是：普通任务自动路由可正式使用；高风险任务的“优先 GPT”规则已生效，但执行面仍会降级为 DeepSeek，不能宣称 GPT 车道已打通。

## 影响
OpenClaw 现在具备了可用的自动模型分发层，普通开发任务不再需要手动切模型。
同时要认识到，NadirClaw 直接消费 ChatGPT Pro / openai-codex 的能力并不等于 OpenClaw 原生能力；即使绕过了 public Responses API 的 scope 问题，`chatgpt.com/backend-api/codex` 仍要求更接近 OpenClaw 原生的受信任请求形态。
如果后续必须让高风险任务真正落到 GPT-5.5，方向应是“让 OpenClaw 自己发 openai-codex 请求，再把结果回送给路由层”，而不是继续让 NadirClaw 单独模拟 ChatGPT Web/Codex 后端。

## 下次注意
1. launchd 常驻运行的 Python/venv 不要放在 Documents 目录，放到 ~/.nadirclaw 这类稳定路径更可靠。
2. OpenClaw 注入的系统提示非常长，任何基于关键词的升级规则都要优先只看用户消息。
3. 带 tools 的流式请求如果做了上游降级，给 OpenClaw 的返回仍要保持 SSE 语义，否则会被判成不完整响应。
4. 看到 openai-codex/gpt-5.5 失败时，不要只盯 scope。当前已确认至少有三层门槛：public Responses API scope、chatgpt.com 的 Cloudflare/browser 校验、以及 OpenClaw 原生 Codex 传输契约。

