# OpenClaw 禁止使用 Claude/Anthropic

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-18T06:18:37Z

## 摘要

OpenClaw 运行时严禁依赖任何 Anthropic 服务，违反会导致 Claude 账号被拉黑

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_openclaw_no_anthropic.md`，本次按侯爷要求同步到飞书知识库。

## 内容
OpenClaw 跑起来后不依赖任何 Anthropic 服务。所有执行 LLM 只能是 Codex / Qwen / DeepSeek / Gemini / GPT-5.5。

**Why:** 用户用 Claude 会员，如果 OpenClaw 也走 Claude（即使是通过 Claude Code session），Anthropic 会判定违反 ToS，账号会被拉黑。用户也不使用 Claude API。

**How to apply:**
- OpenClaw 的任何脚本、Skill、配置文件中禁止出现 anthropic、claude-api、claude-code-action 字样
- 自检命令：`grep -r 'anthropic|claude-code-action' ~/.openclaw/` 必须为空
- 新增到 ~/.openclaw/ 的任何 LLM 调用必须走 OpenRouter（GPT/Qwen/DeepSeek），绝不走 Anthropic
- ~/.claude/settings.json 的 hooks 是 Claude Code 用户自己的配置（不属于 OpenClaw），可以存在
- OpenClaw 自身的 skills/scripts/docs/workspace 不能引用 Claude 或 Anthropic

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

