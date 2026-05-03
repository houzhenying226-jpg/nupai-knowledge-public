---
type: runbooks
title: worker 4节点飞书战报真规则
date: 2026-05-02
risk_level: L3
project: openclaw
tags:
  - worker
  - feishu
  - progress
---

# worker 4节点飞书战报真规则

## 背景

2026-05-02 真测翻车：`/oc fix` 派发后 38+ 分钟心跳告警群无任何进度推送，`feishu-failed.log` 全是 `all retries exhausted`。

PR #422 + #446 后建立 worker 4 节点真账战报规则。

## 4 节点真规则

| 节点 | 触发条件 | 飞书消息 |
|---|---|---|
| spawn | `route_and_spawn()` 成功后 | `🚀 [OpenClaw] crm-NUP-XXX worker 启动 (cmd_id=XXXXXXXX LLM=gpt-5.5)` |
| pr-created | worker log 检测 GitHub PR URL | `📝 [OpenClaw] crm-NUP-XXX PR #420 已开 URL=...` |
| review-requested | worker log 检测 `@codex review` | `🔍 [OpenClaw] crm-NUP-XXX @codex review 已提交(等待结果)` |
| auto-merge | worker log 检测 `TASK-DONE` | `✅ [OpenClaw] crm-NUP-XXX PR #420 auto-merge 完成 deploy 触发` |

## 架构：双源

A. `_monitor_worker_log` 后台线程：轮询 `~/.openclaw/worker-logs/<cmd_id>.log`，正则匹配触发。

B. `/worker-event` 端点：`worker.sh` 主动 curl 回调。

## 去重真规则（PR #446）

`(cmd_id, stage)` 联合主键去重，任一来源先到的算，后到的丢弃。
否则双源同时上报 → 飞书重复 2 次。

## PR 空号规则

`worker.sh _post_event "auto-merge"` 调用时，如果 `pr_number` 提取失败为空：

- 中央出口 `broadcast_progress` 必须校验 `pr_number` 非空才发 ✅。
- 空号改发 ❌ 诊断。

禁止：`PR #?` / `PR #null` / 空 PR 号。

## webhook URL 来源优先级

1. `~/.openclaw/openclaw.json` → `channels.feishu.heartbeatWebhook`
2. `~/.openclaw/feishu-webhook.txt` fallback

飞书 19024 关键词拦截自愈：`feishu_send_webhook()` 检测 `code=19024` → 自动消息前加 `[OpenClaw]` 前缀重试。

## 结论

worker.sh `_post_event` 模板真规则：

- 必须 `[OpenClaw]` 关键词前缀。
- 必须含 `cmd_id / project / issue` 三字段。
- `pr_number` 为空时不发 ✅，改发 ❌ 诊断。
- 19024 重试机制真在。
- 双源 `(cmd_id, stage)` 去重必须。

## 下次注意

worker 启动后 5-15 分钟内应收 4 战报。任一缺失 → 不是真闭环：

- 缺 spawn → dispatcher 没 spawn worker。
- 缺 pr-created → worker log 无 PR URL。
- 缺 review-requested → worker.sh 没真 `gh pr comment "@codex review"`。
- 缺 auto-merge → worker.sh 没真合 PR。

战报破不影响 GitHub PR 真账事实源，但破战报会让用户瞎等 30+ 分钟。
