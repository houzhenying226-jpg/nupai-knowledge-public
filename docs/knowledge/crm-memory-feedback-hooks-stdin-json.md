# Claude Code hooks use stdin JSON not env vars

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T11:12:19Z

## 摘要

PostToolUse/PreToolUse hooks receive data via stdin JSON, not environment variables like $CLAUDE_TOOL_EXIT_CODE

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_hooks_stdin_json.md`，本次按侯爷要求同步到飞书知识库。

## 内容
Claude Code hooks (PostToolUse, PreToolUse) receive their data via **stdin JSON**, NOT environment variables.

The JSON format for PostToolUse Bash:
```json
{
  "tool_name": "Bash",
  "tool_input": {"command": "..."},
  "tool_response": {"stdout": "...", "stderr": "...", "interrupted": false},
  "session_id": "...",
  "transcript_path": "/Users/james/.claude/projects/.../session.jsonl"
}
```

**Why:** Discovered when H14 v1 (using `$CLAUDE_TOOL_EXIT_CODE` env vars) silently failed. Debug hook revealed stdin is the actual data source.

**How to apply:**
- All hook scripts must read stdin: `INPUT=$(cat)` then `python3 -c "import sys,json; d=json.load(sys.stdin); ..."`
- There is NO `exit_code` field — detect failures by checking non-empty `tool_response.stderr`
- PostToolUse does NOT fire when Bash command returns non-zero exit code — only fires on success (exit 0)
- Failed commands are stored in JSONL transcript as `toolUseResult: "Error: Exit code N\n..."` (string, not dict)

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

