---
type: runbooks
title: OpenClaw K11 — 任务级心跳与主动汇报
card_type: ops_runbook
source: openclaw_session_2026-05-03
confidence: 0.95
date: 2026-05-03
created_at: 2026-05-03
risk_level: L4
project: openclaw
is_contradicted: false
tags:
  - openclaw
  - dispatcher
  - heartbeat
  - github-webhook
  - feishu
  - K11
  - runbook
---

# OpenClaw K11 — 任务级心跳与主动汇报

## 一、问题（Problem）

OpenClaw dispatcher 不主动汇报任务进展：心跳只看 dispatcher 进程 `last_run_at`，进程在跑就一直“新鲜”——但 worker 卡死、PR gate 红了一夜都不会响（假活）。

通知是一次性 push，没有任务级状态机做对账（如 #450 `ai-review-gate fail` 靠人眼发现）。

已订阅 GitHub webhook（`workflow_run` / `issue_comment` / `pull_request` / `issues`），但没把 webhook 当事实源——失败状态靠 dispatcher 自己 poll。

## 二、根因（Root Cause）

1. 心跳粒度错位：进程级 ≠ 任务级。
2. 状态机推进不会被外部主动扫描 → 卡住的任务不会被告警。
3. webhook 接收到失败事件后只转发给 dispatcher，未直接触发 IM 告警。

## 三、方案（Solution，对标 GitHub 业界做法）

| 能力 | 实现 |
|---|---|
| 任务级心跳 | `state_machine.py` 加 `last_progress_at` 字段，每次 `transition()` / `increment_round()` 都更新 |
| 状态级 SLA | `STATE_SLA = {NEW:5, SPEC:30, SPEC_REVIEW:15, DEV:60, CODE_REVIEW:20, TEST:20, PR:10, PR_CREATED:10, CI_RUNNING:20, AI_REVIEWING:15, ...}`（分钟） |
| 卡死扫描 | `scan_stalled_tasks()` 返回所有超 SLA 未动的任务 |
| 失败秒推 | `webhook_server` 新增 `/github-webhook` 端点：HMAC-SHA256 验签 → `check_run failure/timed_out/cancelled` 立即推飞书 DM |
| 整点看板 | `dispatcher_pulse.py` 每小时扫在飞任务，🟢 正常 / 🔴 超 SLA；无任务时静默 |

## 四、上线路径（Deployment）

对应 PR：[#453](https://github.com/houzhenying226-jpg/nupai-crm/pull/453)（已合并 2026-05-03T15:37:49Z，CI 全绿后 auto-merge）。

5 步上线 checklist：

```bash
# 1. 安装 launchd pulse
bash dispatcher/launchd/install.sh && launchctl load ~/Library/LaunchAgents/com.nupai.k11-pulse.plist

# 2. 生成 GITHUB_WEBHOOK_SECRET → 写入 ~/.openclaw/.env → 重启 webhook_server

# 3. 注册 GitHub webhook（事件：check_run + workflow_run）
gh api -X POST repos/houzhenying226-jpg/nupai-crm/hooks

# 4. 启动 smee relay → localhost:18790/github-webhook

# 5. 端到端验证：正确签名返 200，错误签名返 401
```

实际落地状态（2026-05-03 23:47 CST）：

```text
✅ launchd 加载成功（com.nupai.k11-pulse）
✅ webhook 注册成功（hook_id=616557107）
✅ smee relay 在跑，2 条 ping delivery 均 200
✅ 端到端签名验证通过
```

## 五、未结尾巴（Open Items）

| 优先级 | 问题 | 影响 |
|---|---|---|
| P1 | `TERMINAL_STATES` 只含 `{"DONE"}`，漏 `MERGED/CLOSED/FAILED/CANCELLED` | 已结束的老任务每小时被 pulse 误报 🔴 |
| P1 | `workflow_run` 已订阅但 handler 是否接上未验证 | 部分聚合 gate 只发 `workflow_run`，可能漏报 |
| P1 | `/github-webhook` 路径未接 `_is_duplicate` 幂等去重 | 重跑 PR 时同一失败被推 N 条飞书，会刷屏 |
| P2 | pulse 对长尾卡死任务每小时重复 🔴 | 同一条噪声收敛差；建议第二次起降级为 🟡 |
| P2 | smee channel 复用 sentry 的可能冲突 | 建议为 K11 单开一条 channel |

## 六、健康指标（7 天后必看）

```bash
# 1. webhook 成功率（红线 <95%）
gh api repos/houzhenying226-jpg/nupai-crm/hooks/616557107/deliveries --paginate \
  | jq '[.[] | .status_code] | group_by(.) | map({code: .[0], n: length})'

# 2. pulse 实际触发次数（红线 <140 次/7天）
log show --predicate 'eventMessage contains "k11-pulse"' --last 7d --info \
  | grep -c "Service exited"

# 3. 主动告警频次涨幅（上线前 vs 上线后）
grep -c "feishu_send_command_dm" ~/nupai-dispatcher/dispatcher.log
```

## 七、关联资源（Related）

- PR：<https://github.com/houzhenying226-jpg/nupai-crm/pull/453>
- 旧版心跳脚本：`heartbeat_watchdog.py`（进程级，K11 后保留作 fallback）
- 状态机源文件：`dispatcher/state_machine.py`
- Webhook：`dispatcher/webhook_server.py`
- 整点看板：`dispatcher/dispatcher_pulse.py`
- 关联前序 PR：#450（ai-review-gate fail 案例）、#451（CI 配置修复）

## 八、经验沉淀（Lesson Learned）

1. “进程活着” ≠ “任务活着”——心跳必须做到业务对象粒度，不能停在进程级。
2. 不要轮询 GitHub，让 GitHub 推给你——webhook + HMAC 是事实源，比 dispatcher poll 更快更省。
3. 反馈通道本身也要监控——K11 上线后必须看 webhook 成功率、pulse 触发次数，否则反馈通道挂了你照样瞎。
4. 合 PR 前 review 出的尾巴必须当场记下——这次 #453 合并时 3 个 P1 问题（`TERMINAL_STATES`、`workflow_run handler`、去重）被跳过，靠这张卡片跟踪。
