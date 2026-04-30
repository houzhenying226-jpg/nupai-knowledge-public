# cron delivery.mode=announce 导致任务不执行

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

振英的自动巡检 cron（crmpatrol、pospatrol）在飞书收到的是任务文本原文（「【自动巡检】请检查...」），而不是执行结果。排查发现 jobs.json 中存在 delivery.mode: announce 字段。

## 正文

## 背景

振英的自动巡检 cron（crm-patrol、pos-patrol）在飞书收到的是任务文本原文（「【自动巡检】请检查...」），而不是执行结果。排查发现 jobs.json 中存在 `delivery.mode: announce` 字段。

## 根因分析

`delivery.mode: announce` 使 OpenClaw 将 cron payload 直接广播到飞书频道，完全跳过 agent 执行阶段。这不是 OpenClaw 缺陷，是配置错误。

验证方法：`grep "crm-patrol" ~/.openclaw/logs/gateway.err.log` 日志完全空白 → 证明 agent 从未被执行。

## 处理方式

删除 jobs.json 中所有 job 的 `delivery` 字段，执行 `openclaw reload`。

```python
import json
with open('/Users/james/.openclaw/cron/jobs.json') as f:
    cfg = json.load(f)
for job in cfg['jobs'].values():
    if 'delivery' in job:
        del job['delivery']
with open('/Users/james/.openclaw/cron/jobs.json', 'w') as f:
    json.dump(cfg, f, ensure_ascii=False, indent=2)
```

删除后：cron 触发 → agent 执行 → 结果发回飞书。

## 经验教训

- OpenClaw cron jobs.json 中的 `delivery` 字段是覆盖行为，设置后 payload 直接发送，不经过 agent。此字段只应在「需要广播原始消息」场景使用，日常巡检任务不加。
- 诊断顺序：先查 gateway.err.log 有没有 agent 执行记录，空白 = 问题在配置层，不是代码层。
- 「改之前先看实际 jobs.json 结构」比直接给修复命令更稳妥，避免基于错误假设的修复。
