# OpenClaw 三模型自动升级路由接入 NadirClaw

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-15T22:01:02Z

## 摘要

OpenClaw 已切到 nadirclaw/auto，普通开发任务自动走 DeepSeek，中文文档任务走 Qwen；高风险任务现在会通过 OpenClaw 本地 /v1/responses 原生桥真正落到 openai-codex/gpt-5.5，tool-calling 与 SSE 都已闭环。

## 正文

> 记录日期：2026-05-06  风险等级：L2

## 背景
用户希望在 OpenClaw 内实现三模型自动分配，不再手动切换 Qwen / DeepSeek / GPT-5.5。
核心原则是“更好的模型放在最好的位置”，并要求部署后必须测试闭环，再整理成知识卡。

## 内容
本次采用 NadirClaw 作为 OpenClaw 前置路由层，配置为 simple=Qwen、mid=DeepSeek、complex=GPT-5.5。
补丁点有四类：
1. 路由判定只看用户消息，不再把 OpenClaw 系统提示里的 deploy / rollback / production 等词误判成高风险。
2. 对 OpenRouter 兼容参数做裁剪，reasoning_effort 不再发给 Qwen / DeepSeek，避免 UnsupportedParamsError。
3. 对带 tools 的 streaming 请求改成“上游缓冲、下游仍返回 SSE”，解决 OpenClaw 将成功响应误判为 incomplete terminal response 的问题。
4. 对 openai-codex/* 不再走 LiteLLM / ChatGPT Web 模拟链路，而是改成 OpenClaw 本地 `POST /v1/responses` 原生桥：
   - 网关开启 `gateway.http.endpoints.responses.enabled=true`
   - NadirClaw 调本机 `http://127.0.0.1:18789/v1/responses`
   - 通过 `x-openclaw-model: openai-codex/gpt-5.5` 指定真实 GPT 车道
   - 将 OpenAI chat-completions 的 messages/tools/tool_choice 转成 OpenResponses 输入，再把 function_call 输出映射回 chat-completions tool_calls
部署后 NadirClaw 常驻在 ~/.nadirclaw 下，由 launchd 托管；并新增 router-alerts 脚本与 LaunchAgent，用于全链路失败、频繁降级、GPT 高风险车道降级的飞书告警。

## 结论
主链路已闭环。
验证结果：
1. OpenClaw 端到端 smoke（“只回复 OK”）已成功由 NadirClaw 路由到 openrouter/deepseek/deepseek-v4-pro，且 OpenClaw 不再记录 fallbackAttempts。
2. OpenClaw 本地 `/v1/responses` 直测成功：`model=openclaw/main + x-openclaw-model=openai-codex/gpt-5.5` 返回 `OK`，证明原生 GPT 入口可被本机服务调用。
3. NadirClaw 直测成功：`model=openai-codex/gpt-5.5` 返回 `OK`，且 requests.jsonl 记录 `fallback_used=null`。
4. Tool-calling 直测成功：NadirClaw 返回 `finish_reason=tool_calls`，并透传 `ping({"value":"42"})`；流式 SSE 版本同样成功返回 tool_calls 与 `[DONE]`。
5. 高风险 auto 路由闭环成功：`“数据库迁移 + 权限改造 + 上线回滚预案”` 命中 `high_risk_override` 后，requests.jsonl 记录 `selected_model=openai-codex/gpt-5.5`、`fallback_used=null`、`modifiers_applied=["high_risk_override"]`。
6. 飞书测试告警已触发，alerts.log 留痕为“[OpenClaw][NadirClaw][P1] TEST degraded-request alert”。
因此当前可用状态是：普通任务自动路由可正式使用；高风险任务已经能真正落到 GPT-5.5 原生车道，不再依赖 DeepSeek 兜底执行。

## 影响
OpenClaw 现在具备了可用的自动模型分发层，普通开发任务不再需要手动切模型。
关键经验是：不要让 NadirClaw 单独模拟 ChatGPT Web/Codex 后端。正确做法是让 OpenClaw 自己发原生 openai-codex 请求，NadirClaw 只负责路由和协议转换。
这样既保住了 GPT-5.5 的真实调用能力，也保住了 tool-calling 与流式返回，不需要再赌 `api.responses.write` scope、Cloudflare 校验、或隐藏请求头细节。

## 下次注意
1. launchd 常驻运行的 Python/venv 不要放在 Documents 目录，放到 ~/.nadirclaw 这类稳定路径更可靠。
2. OpenClaw 注入的系统提示非常长，任何基于关键词的升级规则都要优先只看用户消息。
3. 带 tools 的流式请求如果走 OpenClaw 原生桥，仍建议在 NadirClaw 侧做“上游缓冲、下游 SSE 包装”，这样最稳，不容易碰到客户端对 chunk 语义的兼容问题。
4. 看到 openai-codex/gpt-5.5 失败时，不要继续补 ChatGPT Web 模拟链路。优先检查本机 OpenClaw `/v1/responses` 是否开启、网关 token 是否有效、以及 `x-openclaw-model` 是否仍在 allowlist 内。

