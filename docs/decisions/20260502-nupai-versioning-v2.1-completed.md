---
type: decisions
risk: L3
project: 全局
created: 2026-05-02
status: CLOSED
version: 3.8.0
---

# [KNOWLEDGE_CARD] NuPai 版本治理 v2.1 落地完结

## 结论

NuPai 版本治理 v2.1 子系统已于 2026-05-02 全链路验收通过。
飞书"NuPai 版本发布通知"群收到 v3.8.0 升级卡片，链路闭环。

**涉及 Linear Issues**：NUP-349 / NUP-350 / NUP-351 / NUP-352 / NUP-353（全部 Done）
**合并 PR**：#402（19 文件，1316 行）+ #406（docs-update）
**commit**：acf82d7（main）

---

## 系统构成

| 组件 | 路径 | 作用 |
|------|------|------|
| 版本单一真源 | `docs/VERSION` | 纯数字 SemVer，不含 v 前缀 |
| 发布日志 | `docs/CHANGELOG.md` | Keep a Changelog 格式 |
| 系统快照 | `docs/current_state.md` | VERSION / DATE / RELEASE_ID 自动维护 |
| 知识状态机 | `docs/knowledge/status/*.md` | 4 状态 + frontmatter 校验 |
| SemVer 自动 bump | `scripts/release-note-gen.py` | PR body → feat/breaking/patch |
| 知识状态校验 | `scripts/validate-knowledge-status.py` | 180天超期 + 空目录 + 未来日期 |
| post-deploy 链路 | `deploy.yml` post-deploy job | mutex → pull → bump → docs PR |
| 发布通知 | `release-notify.yml` | docs-update PR 合入 → 飞书卡片 |

---

## 关键经验（防回滚）

### 1. 飞书 Webhook 命名差异必须在 Stage 2 明确区分
- `FEISHU_WEBHOOK`：心跳通知（deploy.yml notify job）
- `FEISHU_RELEASE_WEBHOOK`：版本发布卡片（release-notify.yml）
- 两个 secret 名字相似，误用导致通知发错群或静默失败。
- **规则**：接入新飞书 webhook 时，先 `gh secret list` 核对现有名称再决定用哪个。

### 2. PAT 写入 Secret 前必须校验长度
- 本次 PAT 首次写入为空字符串（`read -s` 粘贴失败），导致 post-deploy 首次
  运行报 "Input required and not supplied: token"。
- `gh secret set` 写入时不校验长度，空值也返回成功。
- **规则**：`echo "$NEW_PAT" | wc -c` 确认 ≥ 80，再 pipe 进 `gh secret set`。

### 3. GitHub Pro 个人仓库 bypass_pull_request_allowances 无效
- GraphQL `bypassPullRequestActorIds` mutation 在个人账号 Pro 计划下不报错
  但实际无效（bypass list 永远为空）。
- 实际绕过方式：`enforce_admins: false`（仓库 owner 天然绕过）+
  `requiredApprovingReviewCount: 0`。
- **规则**：个人仓库不依赖 bypass list，依赖 owner 权限 + 0 required approvals。

### 4. `gh pr checks --watch --fail-fast` 在 CI 未启动时立即 exit 1
- PR 刚创建后立刻运行 `--watch --fail-fast`，GitHub Actions 队列未就绪，
  报 "no checks reported" → exit 1。
- **规则**：`gh pr checks --watch` 前加 `sleep 15`，或改用 `gh pr merge --auto`
  让平台自行等待 checks。

### 5. `--notify-only` 中 deprecated 内容必须 splice 而非前缀截断
- 用 `rn_clean[:dep_match.start()]` 会丢弃 `### Deprecated` 之后的所有段落。
- 正确做法：`rn_clean[:start] + rn_clean[end:]`，仅切除匹配 span。

---

## 验收链路（供下次参考）

```
PR body 含 "- feat: xxx"
→ post-deploy: release-note-gen.py → 3.7.0 → 3.8.0
→ docs-update PR (#406) → CI 全绿 → 合并
→ release-notify.yml → 飞书群收到 [NuPai 版本发布] 3.8.0 卡片
```

---

## 后续维护

- PAT 下次轮换：2027-05-02（见 `runbooks/pat-rotation.md`）
- `validate-knowledge-status` 已加入 main required checks（共 12 项）
- 每次 PR 含 Release Note → 自动 bump，无需人工维护 VERSION
