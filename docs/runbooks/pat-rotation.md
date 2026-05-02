---
status: CURRENT
created: 2026-05-02
next_rotation: 2027-05-02
risk: L3
---

# PAT 轮换 Runbook — openclaw-bot

## 背景

`OPENCLAW_BOT_PAT` 是 nupai-crm 版本治理 v2.1 系统使用的 Fine-grained PAT，
用于 post-deploy job 自动创建 docs-update PR。

## 轮换时间

- **Expiration：365 days**（铁律：不得用 No expiration）
- 下次轮换：2027-05-02（从首次创建日期 + 365 天）

## 轮换步骤

### 1. 生成新 PAT

```
GitHub Settings → Developer Settings → Fine-grained tokens
→ Generate new token

Token name: openclaw-bot-v2（加版本号区分）
Expiration: 365 days
Resource owner: 你的账号（个人）
Repository access: Only select repositories → nupai-crm

Repository permissions:
  Contents:      Read and write
  Pull requests: Read and write
  Metadata:      Read-only（强制必选）
```

### 2. 更新 GitHub Secret

在 Claude Code 中（不暴露在对话/git/飞书）：
```bash
! read -s NEW_PAT
# 输入新 PAT，Enter
! echo "$NEW_PAT" | gh secret set OPENCLAW_BOT_PAT --repo houzhenying226-jpg/nupai-crm
```

### 3. 验证新 PAT 有效

触发一个测试 PR，观察 post-deploy job 能否正常执行 `gh pr create`。

### 4. 删除旧 PAT

GitHub Settings → Developer Settings → Fine-grained tokens → openclaw-bot（旧的）→ Delete

### 5. 更新本文件

把 `next_rotation` 字段更新为新的 + 365 天日期。

## 安全规则

- PAT 永不出现在 git commit / PR 描述 / Feishu 消息 / 对话内容
- 使用 `read -s` 方式输入，不留在 shell history
- Secret 只通过 `gh secret set` 写入，不通过其他途径

## 关联

- 使用方：`.github/workflows/deploy.yml` post-deploy job
- Secret 名：`OPENCLAW_BOT_PAT`
- 仓库：houzhenying226-jpg/nupai-crm
