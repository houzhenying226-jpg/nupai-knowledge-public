# OpenClaw "停摆"根因取证（2026-09-24）

**一句话结论：OpenClaw 本体未停（网关/飞书通道/调度器/看门狗全部在线），停的是"活源"——三类供料断链 + 若干次生故障。它一直在原地喊，但缺的东西都要"人/授权"给。**

## 一、本体在线证据（2026-09-24 12:19 +08 复核）

- ai.openclaw.gateway PID 3348；com.nupai.dispatcher PID 3455；com.openclaw.dashboard PID 3334。
- watchdog 每 3 分钟一跳（最新 12:19:17）：`OK gateway running (PID=3348)` / `OK Feishu WS healthy (last state: ws client ready)`。
- 数十个组件近 3 小时均有日志活动（nl-hash-guard 12:14、commander-watch 12:13、watchdog 12:16、self-heal 12:16 等）。

## 二、三处"停"（主因）

### 1) 开发任务池空置 ≥8 天（自 9/16）
- `fesun-refill.log`：每 5 分钟一条 `⚠️ 待办池空了 —— 需要值守补充池子（这本身是要报的事）`，自 2026-09-16 09:04:29 起累计 **2345 条**。
- `fesun-autopilot.log`：每小时 `🅿️ 队列见底（在跑 0/3 根，还有余位）—— 需要值守补派单令进 /Users/james/.openclaw/fesun-queue`，自 9/16 09:09 起。
- 现状：`/Users/james/.openclaw/fesun-queue` 内无任何待处理派单令（现存均为 .sent）；v5 五条产线（L1–L5）全部 `⏹收工 status=done`、待命（fleet-pump 每 3 分钟刷新）。
- 含义：产线干完上一批后没有任何新派单补充，整队空转等待。

### 2) 验收账停在 0/22 近 4 周（FSN-1243）
- `fesun-ledger-watch`：`🚨 验收账连续 1230 轮不涨，仍停在 0/22（只读具名按钮）—— 登录通道正常，但本进程无 FESUN_PROD_USR/FESUN_PROD_PWD ⇒ 匿名会话，后端 AUTH_REQUIRED（FSN-1243）`（每约 4 小时升级一次，仍在持续）。
- 已知唯一前置（自 2026-08-28 挂起）：需要一个**只读生产账号**并注入 FESUN_PROD_USR / FESUN_PROD_PWD。系统自判 HUMAN_ONLY、明确不自取凭据。

### 3) 巡检链断供约 1 个月
- v5：fleet-watchdog 每 3 分钟 `ISSUES PATROL_STALE status=FAIL age=592660s`（≈6.9 天；自 9/15 15:20 首现，累计 3798 条）。
- Commander：`[巡检提醒] 距上次巡检已 45179 分钟（阈值 90）——该派一轮巡检了`（自 8/24 起持续提醒；实际巡检停约 1 个月）。
- 含义：两条"定期巡检"链都没有再被派发，提醒无人接手。

## 三、四处次生故障（顺带查到）

| 项 | 症状 | 证据 |
|---|---|---|
| MOS 文案链 | 1 条 job 卡住（"说在跑但 0 镜超过 30 分钟"） | fesun-mos-heartbeat.cache（12:21 仍报）；inbox 自 9/21 起多次 |
| Prime Agent daemon | 8/27 起掉线（socket ENOENT），stall-watchdog 每分钟报错 | ~/.prime/agent/logs 最后写入 Aug 27；launchctl exit=1 |
| OpenClaw 自检 | 每小时 `OUTCOME NOT_READY hard_fails=1-config-validate 3-gateway-health 4-plugin-runtime`（最近 11:36） | launchd-selfcheck.log（与 wiki-lint 同根：主配置含未识别字段） |
| 告警静音 | 一类告警默认不推送：`[feishu_dm] alert webhook suppressed kind=github_gate_alert` | dispatcher.log / selfcheck stdout（需 OPENCLAW_ALERT_WEBHOOK_PUSH=1 才推） |
| 网关内存高压 | `memory pressure: level=critical`（rss 2.67GiB、30 秒增 1.18GiB、超阈值 118%）；系统自带建议"不稳则重启网关"；当前仍在正常工作 | Library/Logs/openclaw/gateway.log 与 /private/tmp/openclaw/openclaw-2026-09-24.log（12:22） |
| 模型降级回退 | `openai` 配置"temporarily unavailable" → 自动回退 deepseek-v4-pro 成功（不断流，但持续以回退链运行） | 同上（12:22–12:24，cron-nested 通道） |

## 四、恢复路径（按优先级；本次未改动 OpenClaw 任何组件）

1. **补派单令**放进 `/Users/james/.openclaw/fesun-queue` → v5 产线即可复工。（任务指针=明细表总纲；派单令为 .json 文件。）
2. **FSN-1243**：提供或授权创建"只读生产账号"，注入 FESUN_PROD_USR/PWD → 验收账可开始从 0 动。
3. **巡检重派**：v5 与 Commander 各派一轮。
4. 次生：MOS 卡住 job 排查；Prime 守护进程清退或复活；自检配置字段修复；评估是否打开告警推送。

## 五、边界说明

- 本报告仅为诊断，未改动 OpenClaw 任何组件。
- 早前已修的相关项见同目录 `openclaw-ledger-audit-20260924.md`（门店/Dify 误报、feishu-wiki-sync、知识库推送恢复）。
