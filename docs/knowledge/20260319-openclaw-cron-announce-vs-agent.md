# OpenClaw cron 两种执行模式：announce vs agent

> 类型：知识条目 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw cron 的 jobs.json 支持两种执行模式，行为截然不同，配置错误会导致任务「静默失效」——飞书收到消息，但 agent 从未被执行过。

## 正文

## 背景

OpenClaw cron 的 jobs.json 支持两种执行模式，行为截然不同，配置错误会导致任务「静默失效」——飞书收到消息，但 agent 从未被执行过。

## 两种模式对比

| 模式 | jobs.json 配置 | 行为 | 飞书收到 |
|---|---|---|---|
| **announce（广播）** | `"delivery": {"mode": "announce"}` | payload 直接发到飞书 | 任务文本原文 |
| **agent（执行）** | 无 delivery 字段 | payload 交给 agent 执行，结果发飞书 | agent 执行结果 |

## 诊断方法

看飞书收到的内容是「任务文本原文」还是「执行结果」：
- 收到原文（如「【自动巡检】请检查...」）= announce 模式或 agent 未执行
- 收到结果 = agent 正常执行

进一步确认：
```bash
grep "crm-patrol" ~/.openclaw/logs/gateway.err.log
```
空白 = agent 从未执行（announce 模式）
有记录但有 400 = agent 执行了但发送飞书失败（消息权限问题）

## 配置规范

**日常巡检 / AI 分析 cron**：不加 `delivery` 字段（使用 agent 模式）

**仅广播通知 cron**（如定时提醒文字消息）：加 `delivery` 字段

```json
{
  "jobs": {
    "crm-patrol": {
      "schedule": "0 * * * *",
      "payload": { "message": "检查 ~/nupai-crm 错误日志..." }
    }
  }
}
```

## 经验教训

- announce 模式本身不是 bug，是合法的配置项（用于广播纯文本通知），但误用在需要 AI 执行的任务上会完全静默失效且难以察觉。
- 检查新 cron 是否正确执行时，直接看 gateway.err.log 是否有 agent 执行记录，比等飞书消息更可靠。
