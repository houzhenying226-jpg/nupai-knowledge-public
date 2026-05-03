# Store每日备份+异地复制

> 类型：操作规程 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-03T12:35:58Z

## 摘要

备份脚本可复用到 CRM/Dify：复制 deploy/backup.sh，把 pg_dump 换成对应库的 dump 命令，cron 走同时间，存储路径分目录避免混淆。.env 改了必须 recreate 才重载。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
Store 是当前 NuPai 唯一有完整备份的项目，备份策略是新项目的参考模板。基础设施清单 §8 单列。

## 内容
方式 pg_dump，频率每天凌晨 3:00（cron），保留=日备 30 天 + 周归档永久。本地存储 /opt/nupai-store/backups/，异地复制到 123.57.224.35（职业装1）。脚本 deploy/backup.sh，日志 /opt/nupai-store/backups/backup.log。.env 文件每日同步到异地。 单文件大小：3 月份约 4MB，4 月份增长到约 30MB（业务数据快速增长）。3 月 18 日至今的日备都在（≥30 天保留）。 配套：Docker 日志轮转 /etc/docker/daemon.json 配每容器最多 3 文件×50MB，修改后须 systemctl restart docker 生效（中断容器 20-30 秒，选低峰）。

## 结论
备份脚本可复用到 CRM/Dify：复制 deploy/backup.sh，把 pg_dump 换成对应库的 dump 命令，cron 走同时间，存储路径分目录避免混淆。.env 改了必须 recreate 才重载。

## 影响
Store 数据安全底线、CRM 备份脚本模板、Dify 备份脚本模板、磁盘空间规划（备份增长快）、灾备恢复路径、deploy.sh 部署前 .env 备份。

## 下次注意
盯备份文件磁盘占用，30MB×30 天=900MB+ 周归档将持续增长；新项目复制此模板时把库名/容器名/路径改对再上 cron；备份还原至少季度演练一次；Dify 升级前手动触发一次备份；改 .env 用 recreate。

