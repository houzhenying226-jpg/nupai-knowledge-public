# OpenClaw「烧 token」根治复核（2026-09-29 09:30 实测）

> 结论：**已根治，且此刻没有任何后台空转在烧 token。**
> 修复落地时间 = 2026-09-24 22:48–23:15；本次为 2026-09-29 09:30 复核（隔 5 天）。

---

## 一、根治动作（2026-09-24 当晚落地，均有痕迹）

| # | 动作 | 证据 |
|---|---|---|
| 1 | 关掉默认心跳：`agents.defaults.heartbeat.every = "0m"` | `openclaw.json` mtime **Sep 24 22:48:11**；备份 `openclaw.json.bak-token-burn-20260924-224801` |
| 2 | 5 个 per-agent 心跳 cron 全部停用 | `state/openclaw.sqlite → cron_jobs`：`heartbeat-main / -crm-dev / -manager / -pos-dev / -security` 均 `enabled=0`（原 `everyMs=300000` = 每 5 分钟） |
| 3 | 新增预算哨兵 `scripts/openclaw-token-budget-guard.py`（Sep 24 22:49） | LaunchAgent `com.nupai.openclaw-token-budget-guard.plist`，`StartInterval=3600`，`runs=107`，`last exit code = 0` |
| 4 | 一次性 24h 复查脚本 `openclaw-token-recheck-20260925.py`（Sep 24 23:15） | 9/25 23:15 自动跑完并自卸载；日志见下 |
| 5 | 默认模型切 `deepseek/deepseek-flash`（primary），fallback=`deepseek-v4-pro` | `openclaw.json → agents.defaults.model` |

一次性复查日志原文（`~/.openclaw/logs/token-recheck-20260925.log`）：

```
2026-09-25 23:15:23 OpenClaw token 复查（修复后 24.4 小时）
✅ 彻底解决：心跳已停，后台无异常用量
合计 0.00M token（修复前约 4300 万/天）；心跳回合 0 次
无任何模型调用
push: ✅ 已推送
```

修复前口径（哨兵脚本文档头记载）：心跳每 5 分钟对 5 个 agent 全员跑、**不隔离会话**，每次重读约 17 万 token 历史，**7 天烧掉 5.89 亿 token（99.9% 非人发起）**。

---

## 二、2026-09-29 09:30 实测（复核证据）

### 1) 配置面
- `agents.defaults.heartbeat` = `{"every": "0m", "target": "none", "directPolicy": "block", "lightContext": true, "isolatedSession": false, ...}` → 心跳已关。
- 哨兵离线自检：`python3 scripts/openclaw-token-budget-guard.py --check-config` → **rc=0（0 违规）**。
- 哨兵最近 8 次运行（每小时一次）全部 `config_violations=0 over_budget=0`；`token-budget-guard.state.json` 仍是 9/25 的指纹，说明此后**再没触发过告警**。

### 2) 用量面（近 24h，全 5 个 agent 库）
```
LAST 24h total=22.489M tokens, sessions=1
    22.4889M / 298calls  main agent:main:feishu:default:direct:ou_177cf294199865ce1ec0888693cedb05
```
- 这 22.49M **全部发生在一个会话**：振英的飞书私聊会话（真实开发活），模型 = `deepseek-flash`。
  该会话 9/28 13:28–22:35 的 694 条事件里 298 条 assistant + 372 条 toolResult，内容是 nupai-crm 分支改动、rebase、CI 重跑、force-push 等真实工作；最后 `status=killed` / `Aborted`（是人停的）。
- **今天 9/29 00:00–09:30 网关日志里只有 1 次模型调用**（`/private/tmp/openclaw/openclaw-2026-09-29.log`，254 行）。
- 对比：9/28 全天 305 次调用（全部 flash），9/24 修复当天光 v4-pro 就 1,142 次 + 每 5 分钟心跳。
- 近 3000 行日志：`failed to dispatch` 0、`candidate_failed` 0、`compact_only` 0、`stuck session` 0、`inbound debounce` 0 → 故障重试/回退空转已消失。

### 3) 其他循环排查（确认无第二处漏点）
- `crontab -l` 里 9 个 autopilot/心跳类脚本（harness-autopilot、xsy-pipeline、fsn726-autopilot、fesun-mos-heartbeat、hermes-supervisor-keepalive、capcut-refine-watchdog、harness-15min、nupai-dispatcher/heartbeat_watchdog）→ 对模型 CLI（`/Users/james/bin/claude`、`codex`、`openclaw agent`）**命中数全部为 0**。
- OpenClaw cron 现存启用项只剩：`Memory Dreaming Promotion`（每天 03:00，systemEvent→跑本地脚本，今天 03:00 跑过一次，0 token）+ 5 个 `skill-collection-review-*`（每 7 天一次）。均为分钟级以下负担。

---

## 三、残留项（不影响 token，供决策）

1. **会话库偏大**：5 个 agent 的 `openclaw-agent.sqlite` 合计约 **4.0 GB**
   （main 1.5G / security 708M / crm-dev 689M / manager 681M / pos-dev 443M；事件数 35k/25k/25k/24k/18k）。
   只影响磁盘与首次读盘速度（磁盘尚余 72G）。可选用 `openclaw sessions cleanup` + `VACUUM` 瘦身，属可回退的维护动作。
2. **三个休眠会话仍钉在贵模型**：`agent:main:main = deepseek-v4-pro`、`agent:security:main = gpt-5.5`、两个 9/3 的 `gpt-5.5` 飞书会话。
   因心跳已关，它们不会再自发跑；**只有被显式唤醒时才会用贵模型**。要彻底切成 flash 需要单独动作（无现成 CLI 一键改）。
3. `crontab` 有一行 `* * * * * /bin/bash /tmp/cron-probe.sh`，而该脚本已不存在 → 每分钟空跑（无 token 成本，仅噪音）。

---

## 四、可复跑复核命令

```bash
# 1) 心跳是否仍关（应为 0m）
python3 -c "import json;print(json.load(open('/Users/james/.openclaw/openclaw.json'))['agents']['defaults']['heartbeat']['every'])"

# 2) 哨兵离线自检（应 rc=0）
python3 /Users/james/.openclaw/scripts/openclaw-token-budget-guard.py --check-config; echo rc=$?

# 3) 近 24h 各会话用量（哨兵日志即可）
tail -5 /Users/james/.openclaw/logs/token-budget-guard.log

# 4) 心跳 cron 是否仍停用（enabled 应全为 0）
sqlite3 /Users/james/.openclaw/state/openclaw.sqlite "select enabled,name from cron_jobs where name like 'heartbeat-%';"
```

> 复核人：Hermes（值守）｜复核时间：2026-09-29 09:30 +08
