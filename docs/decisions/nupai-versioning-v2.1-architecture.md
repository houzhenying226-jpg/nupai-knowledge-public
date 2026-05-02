---
type: decisions
risk: L3
project: 全局
status: CURRENT
created: 2026-05-02
author: CC（Claude Code）
---

# [KNOWLEDGE_CARD] NuPai 版本治理 v2.1 落地

## 决策摘要

在 nupai-crm 仓库引入全自动版本治理链路，解决「代码版本号靠人工维护、容易忘/不一致」的问题。

## 架构决策

### 决策 1：docs/VERSION 为单一版本号真源（纯数字，不带 v）
- **Why**: git tag 用 `v3.7.0`，Python 计算 SemVer 时只用数字部分。分开存储避免解析混乱。
- **Trade-off**: 与 package.json `version` 字段不同步——未来如果要对齐，需要额外步骤。

### 决策 2：release-note-gen.py 的 --notify-only 为完全独立代码路径
- **Why**: main 流程（bump+写文件）和通知流程（只读+发飞书）不能共享任何 STEP 3-7 代码。
  若共享，`notify_only_mode` 可能意外执行写操作，违反 L2 铁律（L2 事实必须先存在才能宣告）。
- **实现**: 两个独立函数，`--notify-only` 触发 early return，不进入主流程。

### 决策 3：paths-ignore 防无限递归（docs/VERSION, CHANGELOG, current_state）
- **Why**: deploy.yml 触发 → post-deploy → docs-update PR 合入 main → 若没有 paths-ignore，
  CHANGELOG.md 合入会再次触发 deploy → 无限循环。
- **机制**: `paths-ignore` 让 docs-only 推送不触发 deploy job。

### 决策 4：docs-update PR 走普通 required checks，不 skip CI
- **Why**: commit message 不加 `[skip ci]`，让 validate-knowledge-status 等 checks 正常跑。
  这确保即使是自动生成的版本文档，也经过 CI 校验。
- **Trade-off**: CI 时间增加约 10 分钟。可接受，因为版本发布不是高频操作。

### 决策 5：bypass list 必须有 openclaw-bot
- **Why**: `gh pr merge --auto --squash` 要求 opener 在 bypass list 里，否则 auto-merge 因等待
  approval 永久挂起，deploy job 超时，后续 PR 全部堆积。
- **风险**: bypass 权限是高权限。openclaw-bot PAT 泄露 = 任何人可绕过 review 合 PR。
  缓解：使用 Fine-grained PAT，限定仓库范围，定期轮换。

### 决策 6：release-notify.yml 触发条件为 PR closed + merged + paths: docs/CHANGELOG.md
- **Why**: 不用 `push` 触发（会与 deploy.yml 的 paths-ignore 打架），用 PR 事件更精确。
  只有真正 merged 的 PR 才发通知，cancelled/closed PR 不发。

## 已知局限

1. **nupai-store 未覆盖**: 将来给 store 加同样能力需复制全套 PAT/Secrets/workflow，
   不可复用。建议第二次实施时抽象成 reusable workflow。
2. **PAT 安全**: openclaw-bot PAT 是高权限凭据，建议中期改用 GitHub App（更细粒度权限控制）。
3. **scripts/ 不在 infra-only 豁免列表**: ai-review-gate 会要求 3 个 AI 审查 scripts/*.py，
   每次改脚本都需要等审查（约 10-15 分钟）。可考虑把 scripts/ 加入 infra-only 豁免。

## 关联

- Linear: NUP-349, NUP-350, NUP-351, NUP-352, NUP-353
- PR: https://github.com/houzhenying226-jpg/nupai-crm/pull/402
- Blueprint: docs/blueprints/nupai-versioning-v2.1.md
- Branch Protection 快照: infra/branch-protection-snapshot-20260430.json
