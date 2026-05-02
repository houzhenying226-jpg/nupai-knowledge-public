# 本地挂载vs镜像打包更新铁律

> 类型：架构决策 | 项目：_global | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

铁律：镜像打包的容器必须在 deploy.sh 的 rebuild 列表里，否则代码白写。镜像打包 service 部署命令：docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build <service>。Store deploy.sh v3 已加 dingtalk-bot + nginx + 新鲜度检

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
NuPai 容器有两种更新方式，混用会让代码白写。2026-04-17 排查发现 Store dingtalk-bot 27 天未 rebuild、CRM cortex 37 天未 rebuild，全部因为开发者用了只对本地挂载有效的 docker compose restart。

## 内容
（1）本地挂载（restart 即生效）：Store api → /opt/nupai-store/backend/app → /code/app；Store scheduler → 同上；Store event-consumer → 同上；Store nginx → /opt/nupai-store/dist → /usr/share/nginx/html；CRM fastapi → /opt/nupai-crm/backend/app → /app/app；CRM nginx → /opt/nupai-crm/nginx/nginx.conf → /etc/nginx/nginx.conf。 （2）镜像打包（必须 rebuild）：Store dingtalk-bot → dingtalk-bot/Dockerfile（27 天未 rebuild 已修，2026-04-17）；CRM nextjs → 仓库根目录 Dockerfile；CRM cortex → cortex/Dockerfile（37 天未 rebuild 已修，2026-04-17）。 镜像打包 service 全部在 docker-compose.prod.yml。

## 结论
铁律：镜像打包的容器必须在 deploy.sh 的 rebuild 列表里，否则代码白写。镜像打包 service 部署命令：docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build <service>。Store deploy.sh v3 已加 dingtalk-bot + nginx + 新鲜度检查（PR #56）；CRM deploy.sh 已加 cortex + 回滚消息矩阵（PR #257）。

## 影响
deploy.sh 服务列表正确性；所有涉及 dingtalk-bot/nextjs/cortex 改动的部署；新增有独立 Dockerfile 的 service 必须同步 ① 放 docker-compose.prod.yml ② 加进 deploy.sh rebuild 列表 ③ 同步基础设施清单 §4。

## 下次注意
改容器代码前查基础设施清单 §4 更新方式表；动 dingtalk-bot/nextjs/cortex 必须 rebuild；新增容器若有独立 Dockerfile 一定放 prod.yml；deploy.sh 加新 service 后做“改一行注释 → 部署 → 看容器创建时间”演练，确认 rebuild 列表生效。

