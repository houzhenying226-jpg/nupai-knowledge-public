# Codex 审查触发机制：自动触发，不是评论触发

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-15T11:57:47Z

## 摘要

chatgpt-codex-connector 在 PR 创建/push 时自动审查，Codex 不来时要推空 commit 重触发，不是留评论

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_codex_review_trigger.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

PR #572 的 ai-review-gate 报 "Codex未审查"。我留了 `/codex review` 评论，等了很久 Codex 还是没来。

## 结论

`chatgpt-codex-connector` GitHub App 的触发方式：
- ✅ **自动触发**：PR 创建（opened）或 push 新 commit（synchronized）事件
- ❌ **不触发**：`/codex review` 评论（这个命令对 chatgpt-codex-connector 无效）

## Codex 不来时的正确处理

```bash
# 推一个空 commit，触发 PR synchronized 事件，让 Codex 重新扫描
git commit --allow-empty -m "chore: retrigger CI for Codex review"
git push
```

## Codex quota 超限时

Codex 会自动回复：
> "You have reached your Codex usage limits for code reviews."

这条回复**算作已审查**（gate 脚本 grep 包含 `usage limits|reached your Codex`）：
```bash
CODEX_PRESENT=$(echo "$REVIEWS $REVIEW_COMMENTS $ISSUE_COMMENTS" | grep -c "chatgpt-codex-connector" || true)
# > 0 即视为已审查，只有 CHANGES_REQUESTED 才会 FAIL
```

所以 quota 超限不会卡 gate，quota 恢复后正常审查也不会卡。

## 等待顺序

1. PR 创建 → 等 Codex 自动来（通常 2-5 分钟）
2. 超 5 分钟没来 → 推空 commit 重触发
3. 仍无响应 → 检查 Codex App 是否在 repo settings 中安装

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

