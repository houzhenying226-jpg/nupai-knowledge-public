# OpenClaw 执行层停滞与自愈升级复盘

> 类型：故障复盘 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T18:30:01Z

## 摘要

多个待办长期停滞的根因是执行层缺少统一可验证状态机；本次升级以 OpenClaw Gate、worker reconciler、runtime heartbeat、PR ledger 和 Feishu alert 建立自愈闭环。

## 正文

> 记录日期：2026-05-06  风险等级：L1

## 背景
多个待办长时间停滞，TASKBOARD 保留“已派执行脑 / 等修 gate / 等 PR”的历史状态，但运行面没有足够活跃 worker 推进。部分 worker 在 GitHub push、PR 更新失败、worktree 锁冲突、网络/DNS 失败后，没有形成稳定的自动重试、死信、告警闭环。用户侧只能看到“任务派出过”，看不到“执行层已断 / 正在自愈 / 需要人工配置”的真实状态。

## 内容
根因是旧规则只有半闭环：能派工、能记录部分结果，但没有把读评论、处理评论、双 AI 审查、CI、push、PR 更新、告警、重试上限收敛成唯一机器可验证状态机。worker 判活不能依赖 stdout 静默，必须使用显式 heartbeat 和 runtime ledger。PR 上多个 required checks 并列展示会造成 UX 混乱，应由单一 aggregator check 作为真账。Store 仓库缺少 OPENROUTER_API_KEY secret，导致 Qwen 子审失败，这是仓库配置缺口，不能用假通过绕过。

本次修复包括：
1. CRM main 已合入 OpenClaw Gate 聚合器：只把 OpenClaw Gate 作为 required check，Qwen、DeepSeek、旧 CI 作为 Gate 内部子检查。
2. CRM branch protection 已切换为 required checks = OpenClaw Gate，conversation resolution = enabled。
3. CRM 已合入运行时基础：worker heartbeat/stale 分类、PR 状态账本、worker 日志分类、Feishu 告警格式。
4. CRM 已合入 GitHub dispatch：读取 GitHub webhook/check/review 事件，幂等触发 fix_pr_ci / fix_pr_review。
5. CRM 已合入 worker watchdog：reconciler 支持 retry、backoff、死信、告警、自动重派。
6. 本机 launchd 已开启 OPENCLAW_RECONCILER_ALERTS=1、OPENCLAW_ALERT_WEBHOOK_PUSH=1、OPENCLAW_RECONCILER_MAX_RETRIES=6。

验收标准：
- OpenClaw 运行面：gateway reachable，worker-reconciler last exit code = 0，且 active workers > 0 或有明确死信/告警。
- GitHub 真账：PR 只需要 OpenClaw Gate 一个 required check；Gate 内部必须强制 Qwen、DeepSeek、旧 CI、review threads。
- review comments：Gate 必须读取 unresolved review threads；修复后要回复并 resolve。
- merge queue：Gate 在 merge_group 下不能等待 PR-only 子检查；conversation resolution 要覆盖 merge_group 内所有 PR。

## 结论
OpenClaw 执行层必须升级为“控制器 + Gate 聚合器 + runtime ledger”的闭环系统。TASKBOARD 只能作为观察面，不能作为执行真账；真正真账应来自 worker heartbeat、runtime 状态、PR ledger、Gate 聚合结果和 Feishu 告警。没有这些闭环时，“已派发”不等于“正在推进”。

## 影响
以后所有 worker/PR/CI/review 类异步任务都必须进入统一状态机：可判活、可重试、可死信、可告警、可验收。GitHub branch protection 只应依赖 OpenClaw Gate，Qwen/DeepSeek/旧 CI 作为内部子检查，避免 required checks UX 混乱和误判。Store 仓库切 Gate 前必须先补 OPENROUTER_API_KEY secret，否则 Qwen child review 会失败并锁住 Gate。

## 下次注意
1. 不要用“无日志增长”判断 worker 死活，统一用 heartbeat / runtime ledger。
2. 不要把 Qwen/DeepSeek 直接设为 required checks，只设 OpenClaw Gate，子检查由 Gate 聚合。
3. Store 切 Gate 前必须先补 OPENROUTER_API_KEY secret。
4. review comments 必须读取 unresolved threads，修完要回复并 resolve。
5. merge_group 场景下 Gate 不能等待 PR-only 子检查。

