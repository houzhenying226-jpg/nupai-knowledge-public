# OpenClaw 执行层必须采用控制器模式

> 类型：架构决策 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:30:00Z

## 摘要

OpenClaw 执行层决策采用 controller/reconciler 模式：以 desired/current state 对账，自动恢复可重试 worker，死信告警，不再依赖单次派发脚本和人工追问。

## 正文

> 记录日期：2026-05-05  风险等级：L1

## 背景
用户要求 OpenClaw 能像成熟无人值守系统一样，晚上睡前批量布置任务后自动推进。2026-05-05 的停滞事故说明，旧执行层只做一次性派发，没有持续控制循环；任务挂住后，系统不会主动恢复，也不会及时汇报。

## 内容
架构决策：
1. OpenClaw worker 执行层必须有独立 reconciler。
2. desired state 来自 pending plans、TASKBOARD、未完成 issue、PR/gate 状态。
3. current state 来自 worker runtime heartbeat、PID、worker logs、PR/branch 信号。
4. controller 每 60 秒对账一次，自动补派 repo-only 可恢复任务。
5. D3/生产写操作不自动执行，只生成阻塞告警和 runbook。
6. 并发默认上限 3，避免模型/机器/GitHub 被重试风暴打爆。
7. 每个 issue 最多 3 次自动重试；超过进入死信，必须人工判断。
8. 系统级运行由 LaunchAgent 承载，不依赖 LLM cron prompt。

成熟经验对照：
- GitHub Actions self-hosted runner 的成熟做法是 runner 不接任务会重排队，长时间排队会失败。
- Kubernetes controller 的成熟做法是持续比较 desired state 和 current state，把系统拉回目标状态。
- Temporal 的成熟做法是 activity heartbeat、timeout、retry policy。
- Sidekiq 的成熟做法是 retry/backoff/dead set。

## 结论
OpenClaw 的执行层不能再被设计成“派出去就完事”。凡是 worker/PR/CI/部署类异步任务，都必须进入 controller/reconciler 管辖。没有心跳、超时、重试、死信、告警、并发闸门的异步执行，默认判定为不具备无人值守能力。

## 影响
- 后续所有 worker 派发能力必须写 runtime state。
- 所有状态看板必须优先读 runtime 活性，而不是只读 TASKBOARD 或历史日志。
- 所有恢复逻辑必须能解释为什么可自动重试、为什么需要人工、为什么进入死信。
- OpenClaw cron 事件只适合提醒和报告，不适合作为 P0 执行守护的唯一机制。

## 下次注意
1. 新增任何 worker 类型时，必须同步接入 worker runtime schema 和 reconciler 分类逻辑。
2. 新增任何任务状态时，必须定义 retryable / blocked / terminal / dead-letter 四类语义。
3. 新增任何看板时，必须显示 active worker 和 stale/dead worker 的区别。
4. 自愈机制上线必须用真实故障样本验收：worker_vanished、remote_blocked、worktree_add_failed、repo_only_confirmation。

