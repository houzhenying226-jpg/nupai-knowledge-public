# Dify附属PostgreSQL数据库

> 类型：操作规程 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

需要单独配 pg_dump 定时备份 docker-db_postgres-1 的 dify 库 + weaviate 数据卷归档，加进 cron。Dify 升级版本前必须先备份这两份。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
Dify 部署在 Store 服务器但是独立项目，自带 PostgreSQL 数据库存放工作流定义、对话历史、知识库等。备份归属不清，需要单独说明。

## 内容
引擎 PostgreSQL 15，容器 docker-db_postgres-1，端口 5432 内部（不对外），数据目录 /usr/local/applications/dify/docker/volumes/db/data。 ❓ 备份现状：不在 Store /opt/nupai-store/backups/ 备份范围（Store backup.sh 只备份 nupai-store-postgres-1 的 nupai 库）。基础设施清单 §15 标 P1 待办。 关联容器：docker-api-1、docker-worker-1、docker-worker_beat-1 都依赖此 PG。weaviate 向量库（docker-weaviate-1）数据卷也在 /usr/local/applications/dify/docker/volumes/。

## 结论
需要单独配 pg_dump 定时备份 docker-db_postgres-1 的 dify 库 + weaviate 数据卷归档，加进 cron。Dify 升级版本前必须先备份这两份。

## 影响
Dify 工作流定义安全（13 个 Workflow + Master + 教练/简报）；NuPai AI 矩阵能否在 Dify 出问题时恢复；Store 服务器整体备份完整度。

## 下次注意
立即给 docker-db_postgres-1 加 pg_dump cron（参考 Store backup.sh 复制改库连接即可）；weaviate 向量库数据卷打 tar 归档；Dify 升级前先备份；备份产物归档目录和 Store 主备份隔开避免混淆。

