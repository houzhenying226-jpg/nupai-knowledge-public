---
type: decisions
title: Worker 5步真账管道接入决策
date: 2026-05-03
risk_level: L4
project: openclaw
tags:
  - worker
  - dispatcher
  - true-ledger
---

# Worker 5步真账管道接入决策

## 背景

2026-05-02 真补债 §3.1 真根因：OpenClaw worker 7 天来从未真出过 PR。
`dispatcher/llm_router.py:spawn_worker()` 直接 `subprocess.Popen` 跑 codex CLI，stdout 重定向日志不解析，不触发进度事件。

`worker.sh` 已实现完整 5 步管道但被注释为 dead code。

## 被否决方案：Python 实时解析 stdout

重复 `worker.sh` 5 步逻辑，维护两份代码。bash 子 shell、进程组、退出码契约都要 Python 重写，风险高。

## 最终决策：spawn_worker 接入 worker.sh

PR #442：`fix(dispatcher): NUP-369 wire spawn_worker to worker.sh + Phase 2-5 pipeline`

新调用链：

```text
spawn_worker(cli="codex", task_type="fix", task_id, issue, project)
└→ bash dispatcher/worker.sh <task_id> <issue> <project> <cli> <repo> <prompt>
   ├ Phase 1: codex exec --full-auto -C <repo> <prompt>
   │  ├ Node 2: 检测 PR URL → POST /worker-event pr-created
   │  ├ Node 3: 检测 @codex review → POST /worker-event review-requested
   │  └ Node 4: 检测 TASK-DONE → POST /worker-event auto-merge
   ├ Phase 2: 校验 HEAD 变动 + 工作树脏检测
   ├ Phase 3: git push origin <branch>
   ├ Phase 4: gh pr create fallback
   └ Phase 5: Linear comment via curl（可选，|| true）
```

## 退出码契约

```text
0  = 全部完成或 codex 正确判定无需改动
11 = codex 遗留未 commit 变更
12 = commit 存在但 push 失败
13 = push 成功但 PR 创建失败
其他 = codex 本身失败
```

## 关键约束

- Linear comment Phase 5 永远不阻塞主流程。
- Phase 2-5 只在 `EXIT_CODE=0` 时执行。
- codex 正确判定无需改动（HEAD 不变 + 工作树干净）→ rc=0，不触发 Phase 2-5。
- `worker.sh` 必须用 process substitution `while ... < <(...)`，避免 bash pipeline 子 shell 变量回传 bug。

## 结论

`worker.sh` 是 OpenClaw worker 真账唯一执行路径，`llm_router.spawn_worker` 必须调 bash `worker.sh`，不许直接 `subprocess.Popen codex`。

验证用 PR #87 STO-202 真账闭环：`commits[0].authors[0] = "OpenClaw"`。
