# Store服务器Dify容器11个

> 类型：操作规程 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

Dify Web 入口走 docker-nginx-1 的 80/443。db_postgres 当前 ❓ 不在 Store 备份范围内（P1 风险）。plugin_daemon 5003 公网开放需评估收紧。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
Dify 部署在 Store 服务器但是独立项目（独立 docker compose），是 NuPai 多个 AI 工作流的运行时（教练/简报/WF1-13）。排障时需要清楚 11 个容器各自分工。

## 内容
docker-api-1（langgenius/dify-api:1.13.0）→ 5001 内部，Dify API。 docker-web-1（langgenius/dify-web:1.13.0）→ 3000 内部，Dify Web UI。 docker-worker-1（同 dify-api 镜像）→ 5001 内部，Worker。 docker-worker_beat-1（同 dify-api 镜像）→ 5001 内部，定时调度。 docker-nginx-1（nginx:latest）→ 0.0.0.0:80→80, 0.0.0.0:443→443，Dify 反代（这是 Store 唯一占用 80/443 的服务）。 docker-db_postgres-1（postgres:15-alpine）→ 5432 内部，Dify 数据库（数据目录 /usr/local/applications/dify/docker/volumes/db/data）。 docker-redis-1（redis:6-alpine）→ 6379 内部，Dify Redis。 docker-sandbox-1（langgenius/dify-sandbox:0.2.12）→ 无端口，Dify 沙箱。 docker-ssrf_proxy-1（ubuntu/squid:latest）→ 3128 内部，SSRF 代理。 docker-weaviate-1（semitechnologies/weaviate:1.27.0）→ 无端口，向量数据库。 docker-plugin_daemon-1（langgenius/dify-plugin-daemon:0.5.1-local）→ 0.0.0.0:5003→5003，⚠️ 公网暴露（基础设施清单 §15 P1）。

## 结论
Dify Web 入口走 docker-nginx-1 的 80/443。db_postgres 当前 ❓ 不在 Store 备份范围内（P1 风险）。plugin_daemon 5003 公网开放需评估收紧。

## 影响
所有 NuPai AI 工作流（教练/简报/WF1-13/Master）的运行依赖；Dify 数据安全；Store 80/443 SSL 证书归 Dify nginx 管理；DIFY_BASE_URL/DIFY_API_KEY 系列环境变量配置。

## 下次注意
排查 AI 工作流先 docker logs docker-api-1；Dify 数据库要单独配 pg_dump 备份；plugin_daemon 5003 收紧到 127.0.0.1: 或安全组白名单；Dify 升级版本前先备份 db_postgres + weaviate 数据卷。

