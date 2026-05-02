# Store业务容器7个清单

> 类型：操作规程 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:00:00Z

## 摘要

本地挂载 5 个（restart 即可）；镜像打包 1 个（dingtalk-bot 必须 rebuild）；官方镜像 2 个（一般不动）。所有 compose 操作必须双 -f：docker compose -f docker-compose.yml -f docker-compose.prod.yml [命令]。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
Store 服务器跑 7 个 Nupai 业务容器，每次重启/排障需要知道容器名、镜像、端口、更新方式，避免误操作。

## 内容
nupai-store-api-1（service=api，自建镜像 nupai-store-api）→ 0.0.0.0:8001→8000，FastAPI 后端，本地挂载 /opt/nupai-store/backend/app→/code/app，restart 即生效。 nupai-store-scheduler-1（service=scheduler，同 api 镜像）→ 8000 内部，APScheduler 定时任务，挂载同上，restart 即生效。 nupai-store-event-consumer-1（service=event-consumer，同 api 镜像）→ 8000 内部，事件消费者，挂载同上，restart 即生效。 nupai-store-dingtalk-bot-1（service=dingtalk-bot，自建镜像 nupai-store-dingtalk-bot）→ 无端口映射，钉钉机器人通知，有独立 Dockerfile（dingtalk-bot/Dockerfile），必须 rebuild。 nupai-store-nginx-1（service=nginx，nginx:alpine）→ 0.0.0.0:8080→80，前端静态 + 反代，本地挂载 /opt/nupai-store/dist→/usr/share/nginx/html，restart 即生效。 nupai-store-postgres-1（postgres:16-alpine）→ 127.0.0.1:5433→5432，主库，官方镜像。 nupai-store-redis-1（redis:7-alpine）→ 127.0.0.1:6380→6379，缓存，官方镜像。

## 结论
本地挂载 5 个（restart 即可）；镜像打包 1 个（dingtalk-bot 必须 rebuild）；官方镜像 2 个（一般不动）。所有 compose 操作必须双 -f：docker compose -f docker-compose.yml -f docker-compose.prod.yml [命令]。

## 影响
Store 部署、deploy.sh 服务列表、热更新策略、新增 service 时是否要独立 Dockerfile 的判断。

## 下次注意
改 dingtalk-bot 必须 rebuild 不能 restart；改 api/scheduler/event-consumer 三者共用挂载，重启一个会影响其他；新增 service 有独立 Dockerfile 一律放 docker-compose.prod.yml + 加进 deploy.sh。

