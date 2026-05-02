# CRM业务容器6个清单

> 类型：操作规程 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

本地挂载 2 个（fastapi + nginx，restart 即可）；镜像打包 2 个（nextjs + cortex 必须 rebuild）；官方镜像 2 个。所有 compose 命令必须双 -f：docker compose -f docker-compose.yml -f docker-compose.prod.yml [命令]。CRM 容器名是历史遗留没 -1 后缀。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
CRM 服务器跑 6 个 Nupai 业务容器，每次部署/排障需要知道容器名、镜像、端口、更新方式。CRM 容器命名比 Store 简短（无 nupai-crm- 前缀，是历史遗留），新增容器必须遵循新规范。

## 内容
nupai-fastapi（service=fastapi，自建镜像 nupai-crm-fastapi）→ 127.0.0.1:8001→8001，FastAPI 后端，本地挂载 /opt/nupai-crm/backend/app→/app/app，restart 即生效。 nupai-nextjs（service=nextjs，自建镜像 nupai-crm-nextjs）→ 127.0.0.1:3000→3000，Next.js 前端，有独立 Dockerfile（仓库根目录），必须 rebuild。 nupai-cortex（service=cortex，自建镜像 nupai-crm-cortex）→ 无端口，AI 感知引擎（scanner + scheduler + event_bus），有独立 Dockerfile（cortex/Dockerfile），必须 rebuild。 nupai-nginx（service=nginx，nginx:1.25-alpine）→ 0.0.0.0:8090→80, 0.0.0.0:8493→443，反代，本地挂载 /opt/nupai-crm/nginx/nginx.conf。 nupai-mysql（mysql:8.0）→ 127.0.0.1:3306→3306，主库，数据本地挂载 /opt/nupai-crm/data/mysql。 nupai-redis（redis:7-alpine）→ 127.0.0.1:6379→6379，缓存。

## 结论
本地挂载 2 个（fastapi + nginx，restart 即可）；镜像打包 2 个（nextjs + cortex 必须 rebuild）；官方镜像 2 个。所有 compose 命令必须双 -f：docker compose -f docker-compose.yml -f docker-compose.prod.yml [命令]。CRM 容器名是历史遗留没 -1 后缀。

## 影响
CRM 部署、deploy.sh 服务列表、热更新策略、cortex 12 个 scanner 的运行依赖、新增 service 的命名（必须用新规范 nupai-crm-{service}-1）。

## 下次注意
改 nextjs/cortex 必须 rebuild；改 .env 用 recreate；Build CRM 前先 rm -rf .next/ 防白屏（CRM CLAUDE.md 铁律）；新增容器必须按新命名规范 nupai-crm-{service}-1，不要再用裸名。

