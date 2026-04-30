# GPT-5.5 全栈激活决策

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenAI 于 20260423 正式发布 GPT5.5（ChatGPT + Codex 全开放）。振英通过 ChatGPT Pro 订阅，经 Codex CLI deviceauth 完成认证，将 K8 路由表 7 项主力任务切换到 GPT5.5。

## 正文

## 背景

OpenAI 于 2026-04-23 正式发布 GPT-5.5（ChatGPT + Codex 全开放）。振英通过 ChatGPT Pro 订阅，经 Codex CLI device-auth 完成认证，将 K8 路由表 7 项主力任务切换到 GPT-5.5。

## 决策内容

**GPT-5.5 切换为 K8 路由表 7 项主力：** spec_writing / dev_simple / dev_chinese / dev_arch / pr_review_1 / emergency_rollback / natural_language_translation。

接入路径：
- 订阅：ChatGPT Pro（Plus/Pro/Business/Enterprise 均支持）
- 认证方式：Codex CLI device-auth（ChatGPT 设置 → 启用「为 Codex 启用设备代码授权」→ 生成设备码 → 手机扫码）
- 扫码位置：北京手机（北京 IP 不触发风控），新加坡 Mac 拿 token
- 调用路径：`subprocess.Popen([codex, ...])` 纯 subprocess，无需 Python openai 库

真测响应：`Hello, glad to meet you.` Codex CLI 0.125.0 版本。

## 经验教训

- Claude AI 曾错判「ChatGPT 会员违规」和「GPT-5.5 还没发布」——两次都被振英用事实纠正。教训：涉及外部服务状态，必须 web_search 实查，不能靠训练数据记忆。
- ChatGPT Pro OAuth 路径（JWT 真官方端点）是合规接入方式，OpenAI 官方明确支持 Codex CLI + ChatGPT 登录组合。
- device-auth 登录即使多次杀进程重试，OpenAI 服务器记住授权，任一次扫码成功均可获取 token。
