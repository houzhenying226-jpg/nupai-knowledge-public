# OpenClaw jobs.json delivery 字段清理 SOP

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

飞书收到的 cron 消息是任务文本原文而不是执行结果，或 gateway.err.log 中无 agent 执行记录。

## 正文

## 适用场景

飞书收到的 cron 消息是任务文本原文而不是执行结果，或 `gateway.err.log` 中无 agent 执行记录。

## 诊断步骤

**Step 1**：确认问题类型
```bash
grep "crm-patrol\|pos-patrol" ~/.openclaw/logs/gateway.err.log | tail -20
```
日志空白 = delivery 配置问题；有记录但有 400 = 飞书消息权限问题（见飞书卡片 400 runbook）。

**Step 2**：查看当前 jobs.json 结构
```bash
python3 -c "
import json
with open('/Users/james/.openclaw/cron/jobs.json') as f:
    cfg = json.load(f)
# 打印每个 job 是否有 delivery 字段
for k, v in cfg['jobs'].items():
    print(k, '→ delivery字段:', 'delivery' in v)
"
```

## 修复步骤

**Step 3**：删除所有 delivery 字段
```python
import json
with open('/Users/james/.openclaw/cron/jobs.json') as f:
    cfg = json.load(f)
for job in cfg['jobs'].values():
    if 'delivery' in job:
        del job['delivery']
with open('/Users/james/.openclaw/cron/jobs.json', 'w') as f:
    json.dump(cfg, f, ensure_ascii=False, indent=2)
print('Done')
```

**Step 4**：重载配置
```bash
openclaw reload
```

**Step 5**：验证（等下一个整点，或手动触发）
```bash
openclaw cron run --id crm-patrol
# 等 10 秒后查日志
tail -20 ~/.openclaw/logs/gateway.err.log
```

期望：看到 agent 执行记录，飞书收到执行结果而非原始文本。
