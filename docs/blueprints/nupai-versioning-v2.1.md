# NuPai 版本治理系统 · 最终执行稿 v2.1

> 存档日期：2026-05-02  
> 来源：振英在 claude.ai 对话中贴入，CC 执行  
> 范围：仅 nupai-crm 仓库，nupai-store 不动  
> 执行体：Claude Code（CC），非 Codex

---

## ⚡ 前置手动操作（振英执行，约 12 分钟）

全部完成后才开始 NUP-VERSIONING-01~05。

```
步骤 1：开启 GitHub Actions 创建 PR 权限
步骤 2：生成 Fine-grained PAT（openclaw-bot token）
步骤 3：把 PAT 加进 GitHub Secrets（名称：OPENCLAW_BOT_PAT）
步骤 4：新建飞书发布通知群 + 加 Webhook Secret（名称：FEISHU_RELEASE_WEBHOOK）
步骤 5：确认心跳 Webhook 仍有效（FEISHU_HEARTBEAT_WEBHOOK secret 存在）
步骤 6：Branch Protection bypass list 加 openclaw-bot
```

---

## NUP-VERSIONING-01：建文件结构

文件列表：
- docs/VERSION（内容：3.7.0）
- docs/CHANGELOG.md
- docs/current_state.md
- docs/knowledge/status/_template.md
- docs/knowledge/status/wf-icebreak.md
- docs/knowledge/status/wf-morning-report.md
- docs/knowledge/status/wf-lead-script.md
- docs/knowledge/status/wf-followup-strategy.md

额外：`gh label create auto-merge --color 0E8A16 --description "auto merge eligible" || true`

---

## NUP-VERSIONING-02：建 PR 模板

文件：`.github/pull_request_template.md`

PR 模板包含以下章节：
- 变更说明（必填）
- Release Note（格式：feat:/fix:/deprecated:/breaking: 前缀行；留空=内部变更，不发飞书，不 bump）
- 废弃公告（如有）
- 测试验证 checklist

---

## NUP-VERSIONING-03：release-note-gen.py

文件：`scripts/release-note-gen.py`

功能：
- `--repo` 参数指定仓库
- `--notify-only` 模式：只读 VERSION/CHANGELOG 发飞书，不 bump，不写文件（完全独立代码路径）
- 主流程：GITHUB_SHA → 反查 PR → 提取 Release Note → SemVer bump → 写 VERSION/CHANGELOG/current_state
- 无 LLM 依赖，纯 Python 标准库 + requests
- docs/VERSION 格式：纯数字（3.7.0），不带 v

---

## NUP-VERSIONING-04：validate-knowledge-status.py + CI 集成

文件：`scripts/validate-knowledge-status.py`

功能：
- 校验 docs/knowledge/status/*.md 的 frontmatter
- 必填字段：id, status, version, replaced_by, migration_note, last_reviewed
- status 合法值：CURRENT / DEPRECATED / SUPERSEDED / HISTORICAL
- SUPERSEDED 必须有 replaced_by 且目标文件存在
- last_reviewed > 180 天 = ERROR，> 90 天 = WARNING
- 检测 replaced_by 循环引用（DFS）

CI 集成：在 `.github/workflows/pr-gate.yml` 末尾追加 job `validate-knowledge-status`

---

## NUP-VERSIONING-05：deploy.yml 改造 + release-notify.yml

修改 deploy.yml：
1. on.push 加 `paths-ignore: [docs/VERSION, docs/CHANGELOG.md, docs/current_state.md]`（防无限递归）
2. 追加 `post-deploy` job（依赖 deploy 成功）：
   - checkout（用 OPENCLAW_BOT_PAT）
   - 运行 release-note-gen.py（无飞书，只写文件）
   - 若 docs/ 有变更：新建分支 → commit → push → gh pr create → gh pr merge --auto --squash

新建 release-notify.yml：
- 触发：PR closed + merged + paths: docs/CHANGELOG.md
- 运行 release-note-gen.py --notify-only（用 FEISHU_RELEASE_WEBHOOK 发卡片）

---

## 手动收尾（Step 7）

把 `validate-knowledge-status` 加入 Branch Protection required checks。

---

## 验收方法

1. 新建业务 PR，body 含 `## Release Note\n- feat: 测试版本治理系统自动发布`
2. Merge 到 main
3. 观察：deploy ✅ → post-deploy ✅ → docs-update PR CI 绿 → 自动合入 → 飞书收到升级卡片（版本 3.8.0）

---

## 执行清单

| Issue | 内容 | 文件数 |
|-------|------|--------|
| NUP-VERSIONING-01 | 文件结构 | 8 |
| NUP-VERSIONING-02 | PR 模板 | 1 |
| NUP-VERSIONING-03 | release-note-gen.py | 1 |
| NUP-VERSIONING-04 | validate-knowledge-status.py + CI | 2 |
| NUP-VERSIONING-05 | deploy.yml 改造 + release-notify.yml | 2 |
