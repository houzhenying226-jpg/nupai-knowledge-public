# Store PostgreSQL数据库

> 类型：操作规程 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

连接一律 127.0.0.1:5433，库名 nupai 用户 nupai。改库结构走 alembic 迁移不裸 SQL。备份文件单文件 4MB(3 月)→30MB(4 月)增长明显，盯磁盘。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
Store 主库是 PostgreSQL 16，是门店所有业务数据落地点。排障/迁移/接入 MCP 工具前需要清楚连接参数和表结构概览。

## 内容
引擎 PostgreSQL 16.12，容器 nupai-store-postgres-1，端口 127.0.0.1:5433→5432，用户 nupai，库名 nupai。迁移工具 alembic。数据目录在容器内（非挂载）。表数量 36+ 张。主要表（部分）：contracts、conversation_analysis、conversation_transcript、appointment、approval_flow、audit_log、body_preference、campaign、care_task、competitor_content、content_piece、analytics_events 等。开发机连接：Mac Mini 走 SSH 隧道 localhost:5433 → Store 5433。MCP 包用 mcp-postgres（@modelcontextprotocol/server-postgres 已废弃，不要用）。 ✅ 备份完整：每天 3:00 pg_dump → /opt/nupai-store/backups/，保留日备 30 天 + 周归档永久 + 异地复制到职业装1（123.57.224.35），脚本 deploy/backup.sh，日志 backup.log。

## 结论
连接一律 127.0.0.1:5433，库名 nupai 用户 nupai。改库结构走 alembic 迁移不裸 SQL。备份文件单文件 4MB(3 月)→30MB(4 月)增长明显，盯磁盘。

## 影响
所有 Store 业务读写、API/scheduler/event-consumer 三容器共用、Claude Code MCP 配置、备份链路、新增表/字段流程、磁盘容量规划。

## 下次注意
连库前确认引擎是 PG 不是 MySQL（Store=PG/CRM=MySQL 别串）；改库走 alembic 不裸 SQL；MCP 用 mcp-postgres 别用废弃包；备份盘满前先清旧周归档。

