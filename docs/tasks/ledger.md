# 任务台账

任务台账由飞书多维表 B 同步（C2 task-pulse cron 回写）。

更新时间：2026-04-29（批 1 骨架初始化）

| 任务名 | 类型 | 机器 | 调度 | 管理方式 |
|--------|------|------|------|---------|
| crm-patrol | cron | 新加坡 Mac | */60 * * * * | OpenClaw cron |
| pos-patrol | cron | 新加坡 Mac | */60 * * * * | OpenClaw cron |
| daily-health | cron | 新加坡 Mac | 0 8 * * * | OpenClaw cron |
| weekly-memory | cron | 新加坡 Mac | 0 9 * * 1 | OpenClaw cron |
| weekly-report | cron | 新加坡 Mac | 0 9 * * 1 | OpenClaw cron |
| feishu-wiki-sync | cron | 新加坡 Mac | */30 * * * * | OpenClaw cron（v1.1 新建，批 4 启用） |
| task-pulse | cron | 新加坡 Mac | */5 * * * * | OpenClaw cron（v1.1 新建，批 4 启用） |
| status-snapshot | cron | 新加坡 Mac | 0 22 * * * | OpenClaw cron（v1.1 新建，批 4 启用） |
| wiki-lint | cron | 新加坡 Mac | 0 3 * * 0 | OpenClaw cron（v1.1 新建，批 4 启用） |
| openclaw-gateway | 常驻服务 | 新加坡 Mac | 常驻 | launchd |
| webhook-server | 常驻服务 | 新加坡 Mac | 常驻 | OpenClaw 内嵌 |
| watchdog | 常驻服务 | 新加坡 Mac | 常驻 | launchd |
| feishu-ws | 常驻服务 | 新加坡 Mac | 常驻 | OpenClaw 内嵌 |
