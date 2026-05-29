# OpenClaw worker 静默停滞 P0 事故复盘

> 类型：故障复盘 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-29T14:00:01Z

## 摘要

28 个待办停滞的根因不是单点 GitHub DNS，而是执行层缺少 worker 控制器：无心跳判死、无自动重派、无死信、无强告警，导致 worker 挂死后系统静默等待。

## 正文

> 记录日期：2026-05-05  风险等级：L1

## 背景
2026-05-05，OpenClaw 门店任务队列出现 28 个待办长期停滞。表面现象包括：没有活跃 worker 进程；TASKBOARD 多数停留在“等执行脑开 PR / 修 gate”；部分任务本地修复完成但 PR 未更新；历史 worker 日志出现 GitHub 写入失败、worktree 锁冲突、repo-only 任务误等待确认等问题。用户明确要求夜间无人值守任务必须能持续推进，不能再出现“挂了还在傻等”。

## 内容
事故定位

## 结论
OpenClaw 执行层必须从“派发脚本”升级为“控制器系统”。无人值守的最低标准是：任务有 desired state，worker 有 current state，控制器周期性比较二者；发现 worker 消失、远端失败、锁冲突、等待确认等可恢复状态时自动重派；超过重试上限进入死信并告警。没有 reconciler 的 OpenClaw 只能算半自动，不算 24 小时无人值守。

## 影响
- 以后“没 worker 但还有待办”不再允许静默，必须触发 P0 告警或自动补派。
- 用户睡前批量派任务时，系统会自动维持最多 3 个活跃 worker，失败后补空位。
- GitHub 抖动、DNS 短断、worktree 锁冲突、短命父进程退出不再直接导致队列永久停滞。
- TASKBOARD/看板/worker log 只能作为观察面，不能作为执行层唯一事实源；runtime heartbeat + PID 才是当前活性真账。

## 下次注意
1. 任何“无人值守执行系统”必须先问：控制器在哪里、心跳在哪里、死信在哪里、告警在哪里、并发闸门在哪里。
2. 任何 worker 状态汇报必须同时给出：active worker 数、pid_alive、heartbeat_age、stage、issue、cmd_id、PR/branch。
3. 任何 cron/heartbeat 说 ok，都必须查业务副作用：state history 是否新增、worker 是否真的派出、runtime 是否推进、日志是否落盘。
4. OpenClaw P0 自愈必须用 launchd/systemd 这类系统级守护，不依赖 LLM 自己执行提醒。
5. 夜间任务前必须确认 com.nupai.openclaw-worker-reconciler loaded、run interval 60s、err log 为空、dry-run active_workers/dispatched 符合预期。

