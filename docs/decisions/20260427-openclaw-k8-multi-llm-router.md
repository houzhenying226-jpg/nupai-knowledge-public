# OpenClaw K8 多 LLM 路由器设计

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 不依赖任何单一 LLM 厂商。K8 路由器按任务类型自动选最合适的 LLM，Codex 挂可自动切 Qwen，Qwen 挂自动切 DeepSeek。这是抗单点风险的核心机制，批次 2 落地。

## 正文

## 背景

OpenClaw 不依赖任何单一 LLM 厂商。K8 路由器按任务类型自动选最合适的 LLM，Codex 挂可自动切 Qwen，Qwen 挂自动切 DeepSeek。这是抗单点风险的核心机制，批次 2 落地。

## 路由表设计

| 任务类型 | 首选 LLM | 备选 LLM | 选择理由 |
|---|---|---|---|
| Spec 写作 | Qwen 千问 | DeepSeek | 中文好 + 便宜 |
| Spec 审查 | DeepSeek | Qwen | 双 AI 互审，异质对抗 |
| Dev 简单 bug | Codex (OpenAI) | Qwen | 推理强 |
| Dev 中文需求 | Qwen | Codex | 中文理解准 |
| Dev 架构改动 | GPT-5.5 | Codex | 复杂推理 |
| Test 写作 | DeepSeek | Qwen | 便宜够用 |
| PR Review #1 | Codex (OpenAI) | — | 主 reviewer |
| PR Review #2 | Gemini (Google) | — | 异质互审 |
| AI Gate JSON Schema | Qwen + DeepSeek | — | JSON 硬判双重保证 |
| 紧急部署回滚分析 | GPT-5.5 | Codex | 推理强 |

健康检查（K10）：dispatcher 每 5 分钟跑 each_executor --version，记录到 .agent/executor-health.json。spawn 前查 health，选第一个 healthy executor。任一 LLM 30 分钟不健康 → 飞书心跳告警 @ 振英。

## 已规避的单点风险

- Codex (OpenAI) 挂 → 自动切 Qwen → 不停摆
- Anthropic 涨价 / 服务中断 → 不影响（Claude 不在流水线）
- 阿里 Qwen 限流 → 自动切 Codex 或 DeepSeek
- 某 LLM 输出错误 → review 阶段被另一个 LLM 拦截

## 经验教训

- Spec 写+审必须用两个不同 LLM（异质对抗），同一个 LLM 自审必然存在盲区。
- AI Gate 必须要求 JSON Schema 输出，不能让 LLM 自由描述"通过与否"，这是铁律 2 的直接应用。
- GPT-5.5 切换后 7 项主力任务迁移，验证了路由表的可热更新性。
