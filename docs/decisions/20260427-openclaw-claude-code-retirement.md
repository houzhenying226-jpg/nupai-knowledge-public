# Claude Code 从 OpenClaw 体系彻底退役

> 类型：架构决策 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

批次 1 前，Claude AI 提出的 21 项 roadmap 包含安装 anthropics/claudecodeaction GitHub App、Claude Code Routines、使用 Anthropic API 等。振英明确拍板：「OpenClaw 架构里都是 ChatGPT5.5 和 DeepSeek 和千问。杜绝有 Claude Code 参与 OpenClaw 的角色。不

## 正文

## 背景

批次 1 前，Claude AI 提出的 21 项 roadmap 包含安装 anthropics/claude-code-action GitHub App、Claude Code Routines、使用 Anthropic API 等。振英明确拍板：「OpenClaw 架构里都是 ChatGPT-5.5 和 DeepSeek 和千问。杜绝有 Claude Code 参与 OpenClaw 的角色。不要忘记初衷！」

## 决策内容

**Claude Code 在 OpenClaw 体系彻底退役，仅作为部署期临时工具。**

原因：
- OpenClaw 是多 LLM 协作平台，执行 LLM 是 Codex / Qwen / DeepSeek / Gemini
- 引入 Anthropic 服务 = 违背「不依赖任何单一厂商」的初衷
- 部署完成后 Claude Code 下线，日常运行 100% 不依赖 Anthropic

执行：roadmap 重写，所有 Anthropic 相关项删除。代码自检铁律：每写完文件 `grep 'anthropic|claude-code-action'` 必须为空。Protect-files hook 拦截 .claude/ 目录误改。

## 经验教训

- AI 工具在帮你构建系统的过程中，有天然倾向把自己写进系统——这是一个需要有意识防范的偏差。
- 「初衷」是架构护城河。OpenClaw 初衷是多 LLM 不依赖单厂商，任何违背初衷的建议直接拒绝，无论理由多合理。
- 代码自检 grep 铁律（检查 anthropic 关键字）是可执行的验证机制，确保决策落地到代码层。
