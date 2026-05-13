# OpenClaw worker 自愈验收手册

> 类型：操作规程 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:40:15Z

## 摘要

OpenClaw worker 自愈上线后，验收不能只看 cron ok 或接口 200，必须检查 LaunchAgent、reconciler state、runtime heartbeat、active worker、并发闸门和日志副作用。

## 正文

> 记录日期：2026-05-05  风险等级：L1

## 背景
OpenClaw 旧任务推进链路容易出现“看板有任务、cron 显示 ok、但 worker 实际没活着”的假正常。2026-05-05 的 28 个待办停滞事故证明，执行层验收必须以真账为准，不能只看状态接口或调度记账。

## 内容
验收步骤：
1. 查 LaunchAgent 是否加载：launchctl print gui/$(id -u)/com.nupai.openclaw-worker-reconciler，必须看到 state、run interval = 60 seconds、RunAtLoad 生效。
2. 查恢复器日志：tail -80 ~/.openclaw/logs/worker-reconciler.launchd.log，必须看到 JSON summary，且非 dry-run 时 history 有新增。
3. 查错误日志：tail -80 ~/.openclaw/logs/worker-reconciler.launchd.err.log，必须为空或无新 traceback。
4. 查状态文件：tail -120 ~/.openclaw/worker-reconciler-state.json，必须看到每个自动重派 issue 的 attempts、last_reason、last_source_cmd_id、last_recovery_cmd_id。
5. 查 runtime：ls ~/.openclaw/worker-runtime/*.json，每个 running worker 必须有 status=running、stage、pid、worktree、branch、last_heartbeat_at。
6. 查 PID 活性：pgrep -laf '<cmd_id>|dispatcher/worker\.sh|codex exec --full-auto'，runtime 中的 pid 必须实际存在。
7. 查 dry-run：python3 dispatcher/worker_reconciler.py --dry-run --max-dispatch 4。如果 active_workers 已到上限，应显示 dispatched=0；如果有空位且存在可恢复任务，应显示将自动重派。
8. 查 worker 日志：tail -120 ~/.openclaw/worker-logs/<cmd_id>.log，必须看到 worktree ready、existing PR detected 或 Codex exec 启动证据。

停车条件：
- LaunchAgent 没加载或 err log 有 traceback。
- state history 不新增，但 cron/launchd 声称 ok。
- runtime 显示 running 但 pid 不存在。
- active worker 已有同 issue，reconciler 仍要重复派发。
- worker 停在 spawned 超过 2 个 heartbeat 周期。
- dispatched 连续增加但没有 PR/commit/明确失败原因。

## 结论
OpenClaw 自愈验收必须看“系统级守护 + 状态副作用 + worker 真进程 + 业务推进”四层真账。200 OK、cron status=ok、看板显示任务，都不能单独证明无人值守执行链路健康。

## 影响
- 以后 OpenClaw 执行层上线或改动后，必须按本 runbook 验收。
- 用户问“问题解决了吗”时，不准只说“已启动/已启用”，必须贴出 active worker、state history、dry-run dispatched、LaunchAgent 状态。
- P0 事故中恢复器可以自动处理 repo-only 失败，但 D3 生产写操作仍必须停车等待用户审批。

## 下次注意
1. dry-run 只验证决策，不验证真实派发；真实链路必须至少跑一次 --max-dispatch 1 并观察 runtime。
2. 清理失败试运行遗留 worktree/branch 前必须确认无 dirty changes。
3. 控制器自动重派必须有 attempts/backoff/dead-letter，避免无限循环。
4. 任何状态判断都要按新活跃 worker 优先，旧失败记录只能作为历史证据。

