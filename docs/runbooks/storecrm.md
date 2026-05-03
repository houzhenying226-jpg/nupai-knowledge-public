# Store/CRM定时任务清单

> 类型：操作规程 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-03T12:05:57Z

## 摘要

Store 任务靠系统 cron + scheduler 双轨；CRM 任务全在 cortex 容器内（重启 cortex 会中断所有 scanner）。排查任务漏跑：Store 看 cron 日志 + scheduler 容器日志；CRM 看 cortex 容器日志。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
两台服务器定时任务来源不同：Store 是系统 cron + scheduler 容器；CRM 是 cortex 容器内的 12 个 scanner。排障“为什么没跑/跑错了”前需要清楚任务来源。

## 内容
Store：系统 cron：每天 3:00 跑 deploy/backup.sh（数据库备份）。scheduler 容器：nupai-store-scheduler-1 内部 APScheduler，具体 job 需查容器日志 docker logs nupai-store-scheduler-1（job 列表未在基础设施清单展开）。健康告警 cron：每 5 分钟跑 deploy/health-check.sh（飞书 webhook 告警容器 NOT RUNNING/RESTARTING），日志 backups/health-check.log。 CRM：cortex 容器内：每分钟跑 Pending 事件兜底扫描；12 个 scanner 各自周期不同（信任衰减/承诺到期等，1 小时 ~ 4 小时不等，具体周期需查 cortex 代码）。 开发机：蓝图漂移报告 cron：每天 9:00 推送（振英记忆）。

## 结论
Store 任务靠系统 cron + scheduler 双轨；CRM 任务全在 cortex 容器内（重启 cortex 会中断所有 scanner）。排查任务漏跑：Store 看 cron 日志 + scheduler 容器日志；CRM 看 cortex 容器日志。

## 影响
备份链路、健康告警链路、AI 感知引擎运行、Pending 事件处理时延、scanner 触发的业务逻辑（信任衰减/承诺到期等）。

## 下次注意
重启 cortex 前预估对 12 个 scanner 中断的影响；新增定时任务先决定放 cron 还是放 scheduler/cortex；scheduler 容器 APScheduler job 变更要写文档（当前未在基础设施清单展开是缺口）；蓝图漂移报告 cron 不要漏掉。

