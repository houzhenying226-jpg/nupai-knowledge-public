# 分支管理与 worktree 冲突处理

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-15T19:00:09Z

## 摘要

main 锁在 worktree、提交到错误分支的教训与操作规范

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_git_branch_workflow.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 已知限制

- `main` 分支永久锁在 worktree `/Users/james/.clawteam/workspaces/nupai-fix/nup-383`
- 禁止 `git checkout main`，永远从 `origin/main` 拉新分支
- `gh pr merge <N> --squash --delete-branch --admin` 因 worktree 冲突失败

## 标准创建分支流程

```bash
# ✅ 正确：从 origin/main 创建，不依赖本地 main
git fetch origin
git checkout -b feat/nup-XXX-description origin/main
```

## PR 合并（worktree 冲突时唯一有效方式）

```bash
gh api repos/houzhenying226-jpg/nupai-crm/pulls/<N>/merge \
  -X PUT \
  -f merge_method=squash \
  -f commit_title="<完整标题> (#N)" \
  -f commit_message="<一句话说明"
# 删分支
gh api repos/houzhenying226-jpg/nupai-crm/git/refs/heads/<branch-name> -X DELETE
```

## 提交到错误分支的修复

若发现提交在错误分支（如 feat/NUP-415-browser），补救步骤：
```bash
WRONG_COMMIT=$(git log --oneline -1 | awk '{print $1}')
git checkout -b fix/correct-branch origin/main
git cherry-pick $WRONG_COMMIT
git push -u origin fix/correct-branch
```

## Cherry-pick add/add 冲突处理

```bash
# 冲突文件选用 --ours（保留目标分支版本）
git checkout --ours path/to/conflicted/file
git add path/to/conflicted/file
git cherry-pick --continue --no-edit
```

## 检查 PR 实际包含的文件（防止「已 push」实为空）

```bash
gh pr diff <N> --name-only
git diff main...HEAD --name-only
```
无输出 = 实际为空，需要重新提交。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

