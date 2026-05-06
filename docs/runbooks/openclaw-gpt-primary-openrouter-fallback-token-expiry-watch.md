# OpenClaw GPT 主力链路与 openai-codex token 飞书过期提醒

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

OpenClaw main agent 以 openai-codex/gpt-5.5 为主力，openai-codex/gpt-5.4 为第一 fallback，OpenRouter 的 Qwen 与 DeepSeek 为后备 fallback；openai-codex OAuth 过期检查已做成本机 LaunchAgent，只在剩余 48 小时内通过飞书提醒，不同步 token 本体。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
OpenClaw main agent 需要长期以 ChatGPT Pro 会员通道 GPT 为主力，同时保留 OpenRouter 上的 Qwen 与 DeepSeek 作为后备。此前 openai-codex 旧 profile 已过期，且需要避免 fallback 到 openrouter/auto 或 openrouter/openai/gpt-5.5 这类不可控路径。

## 内容
当前模型链路为：default = openai-codex/gpt-5.5；fallback#1 = openai-codex/gpt-5.4；fallback#2 = openrouter/qwen/qwen3.6-plus；fallback#3 = openrouter/deepseek/deepseek-v4-pro。Qwen 与 DeepSeek 仍通过 OpenRouter key 走 openrouter:default。openai-codex OAuth profile 已统一为 openai-codex:default (houzhenying75@gmail.com)，真实过期时间记录在 ~/.openclaw/agents/main/agent/auth-profiles.json。已安装长期本机提醒脚本 /Users/james/.openclaw/scripts/openclaw-codex-token-watch.py，并由 /Users/james/Library/LaunchAgents/com.nupai.openclaw-codex-token-watch.plist 每天 09:10 运行。脚本只在 token 已过期或剩余不超过 48 小时时发送飞书 DM，提醒内容只包含账号、profile、过期时间、剩余时间和重新登录命令，不包含 token 本体。

## 结论
不需要每次人工重新设置提醒。重新登录后，脚本会自动读取新的 expires 字段。平时验证 token 时间使用 /Users/james/.openclaw/scripts/openclaw-codex-token-watch.py --status；验证模型链路使用 openclaw models list --agent main --status-plain；重新登录命令为 openclaw models auth login --provider openai-codex。

## 影响
GPT 正常时 OpenClaw 优先使用 openai-codex/gpt-5.5；GPT 通道异常时按顺序降级到 openai-codex/gpt-5.4、OpenRouter Qwen、OpenRouter DeepSeek。token 过期风险会提前 48 小时在飞书中显性化，但敏感凭据仍只保存在本机 OpenClaw 配置中。

## 下次注意
修改 default/fallback 后必须重启 ai.openclaw.gateway 并用真实 agent smoke test 确认 winnerProvider/winnerModel/fallbackUsed。不要把 token、OpenRouter key、Feishu appSecret 写入飞书或知识卡。不要加入 openrouter/auto 或 openrouter/openai/gpt-5.5，除非振英明确批准。

