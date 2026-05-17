# MemoryOps v1.1 deployment complete

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-17T08:11:44Z

## 摘要

OpenClaw v1.1 self-evolving memory system — git-backed learnings, 3 capture hooks, shared pool, weekly skill-evolve engine

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `project_memoryops_v11.md`，本次按侯爷要求同步到飞书知识库。

## 内容
MemoryOps v1.1 fully deployed and verified 2026-05-04.

**Components live:**
- `~/.openclaw/.learnings/` — git repo synced to `houzhenying226-jpg/nupai-learnings` (private)
- H14 v3: transcript-scanning PostToolUse hook that captures bash failures from JSONL transcript (UUID-based dedup per session)
- H15 v2: captures cross-check FAIL/REJECT when Write/Edit tool writes to `~/.openclaw/cross-check-log.md`
- H16: 4 insertion points in `~/.openclaw/watchdog.sh` writing to ERRORS.md
- `~/.openclaw/workspace-shared/MEMORY.md` — shared pool for all 5 agents
- `~/.openclaw/scripts/skill-evolve.py` — GPT-4.1 proposal → Qwen + DeepSeek dual review → Feishu → apply/rollback
- `~/Library/LaunchAgents/com.nupai.skill-evolve.plist` — C5 cron Sunday 04:00
- `/oc evolve approve|reject|trigger` — dispatcher command parsing in `command_parser.py` + `webhook_server.py`

**Key fixes discovered during deployment:**
- `learnings-autocommit.sh` credential scan: exclude `<REDACTED>` lines to avoid `WEBHOOK_SECRET=<REDACTED>` false positives
- `skill-evolve.py` API key: read from `~/.openclaw/agents/main/agent/auth-profiles.json` → `profiles["openrouter:default"]["key"]`
- `skill-evolve-rollback.sh`: remove new skills by scanning staging `new-*-SKILL.md` files, not just `proposal.json.new_skills`

**Why:** Closes 3 gaps in OpenClaw v1.0: unreliable manual ERRORS capture, static skill library, isolated agent experience.

**How to apply:** When debugging skill-evolve or hooks, check transcript JSONL format and auth-profiles structure first.

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

