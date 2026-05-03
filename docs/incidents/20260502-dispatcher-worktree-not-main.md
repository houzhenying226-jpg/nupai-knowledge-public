---
type: incidents
title: dispatcher跑worktree不跑main真隐患
date: 2026-05-02
risk_level: L4
project: openclaw
tags:
  - dispatcher
  - worktree
  - deployment
---

# dispatcher跑worktree不跑main真隐患

## 背景

2026-05-02 真补债 §3.2 高危真隐患：dispatcher 进程 `cwd=/Users/james/nupai-crm`，HEAD=`feat/nup-272-wf2-split` worktree，不在 main checkout。

所以 PR 合 main 后，生产 dispatcher 真没读新代码。隐藏 6 天。

NUP-321 真测翻车的 worker 也跑在脏 worktree → rc=11 退出无 PR。

## 症状

- PR 已合并到 main，功能仍不生效。
- `/oc 列表` 显示 worker “已退出”但无 PR 创建。

## 真根因诊断

```bash
ps -o command -p $(pgrep -f webhook_server)
# 路径含老 worktree 名

git -C <cwd> rev-parse --abbrev-ref HEAD
# 不是 main

git status
# 有其他任务遗留脏文件
```

worker.sh Phase 2 误判：

```text
HEAD 不变 + 工作树脏（其他任务遗留）
→ worker 误判 codex 没真 commit
→ rc=11 退出没 git push / gh pr create
```

## 修复

PR #443：`fix(dispatcher): NUP-369 _PROJECT_REPO_MAP crm 指向 nupai-crm-main`

永久配置：

```text
ProgramArguments: /Users/james/nupai-crm-main/dispatcher/webhook_server.py
WorkingDirectory: /Users/james/nupai-crm-main/dispatcher
PYTHONPATH: /Users/james/nupai-crm-main/dispatcher
KeepAlive: true
```

`llm_router.py _PROJECT_REPO_MAP`：

```python
{
  "crm": "/Users/james/nupai-crm-main",
  "store": "/Users/james/-nupai-store",
}
```

新建 main 锁定 worktree：

```bash
git worktree add ~/nupai-crm-main main
```

## PR 合并后更新 dispatcher SOP

```bash
git -C ~/nupai-crm-main pull --ff-only origin main
launchctl kickstart -k gui/$(id -u)/com.nupai.k2-webhook
ps -o command -p $(pgrep -f webhook_server) | grep nupai-crm-main && echo "✅ OK" || echo "❌ STALE"
git -C ~/nupai-crm-main branch --show-current  # 期望 main
```

## 结论

任何 service 跑 worktree 都必须用 `git worktree add` 锁定 main 分支，不能用 `git checkout` 漂移。

## 下次注意

门店真测前 SOP：

```bash
cd /Users/james/-nupai-store
git status
git checkout main
git pull --rebase origin main
git status  # working tree clean
```

工作树不 clean，不许真发 `/oc 修 STO-XXX`。
