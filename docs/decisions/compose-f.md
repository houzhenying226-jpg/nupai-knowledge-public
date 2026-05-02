# compose双-f加载铁律

> 类型：架构决策 | 项目：_global | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

所有 compose 命令一律双 -f，无例外。命令模板写死进 deploy.sh、OPS-COMMANDS.md 白名单、SOP 文档。新增有独立 Dockerfile 的 service 一律放 prod.yml 而不是 base.yml。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
Store 和 CRM 的 docker-compose 都拆成 base.yml + prod.yml。base 放共享服务（数据库/Redis/nginx/主 API），prod 放有独立 Dockerfile 的 service + 生产覆盖。只用 docker compose xxx 不带 -f 会漏掉 prod 里的 service —— 这正是 dingtalk-bot 27 天未 rebuild、cortex 37 天未 rebuild 的根因之一。

## 内容
Store 文件分工（部署目录 /opt/nupai-store/）：docker-compose.yml（2864B）：api, scheduler, event-consumer, postgres, redis, nginx；docker-compose.prod.yml（3744B）：dingtalk-bot（独立 Dockerfile）+ 生产覆盖。 CRM 文件分工（部署目录 /opt/nupai-crm/）：docker-compose.yml（2177B）：fastapi, mysql, redis, nginx；docker-compose.prod.yml（805B）：nextjs, cortex（独立 Dockerfile）+ 生产覆盖。 正确命令模板：docker compose -f docker-compose.yml -f docker-compose.prod.yml [命令]。错误用法：docker compose [命令] —— 漏掉 dingtalk-bot/nextjs/cortex。

## 结论
所有 compose 命令一律双 -f，无例外。命令模板写死进 deploy.sh、OPS-COMMANDS.md 白名单、SOP 文档。新增有独立 Dockerfile 的 service 一律放 prod.yml 而不是 base.yml。

## 影响
deploy.sh、所有运维 SOP、Claude Code 改容器流程、新增 service 规则、OPS-COMMANDS.md 白名单、bash-firewall 白名单、新人 onboarding。

## 下次注意
打 compose 命令前确认双 -f；OPS-COMMANDS.md 白名单只放双 -f 版本；新 service 有独立 Dockerfile 一律放 prod.yml；改 .env 用 recreate；生产禁 docker build --no-cache；服务器迁移先恢复 daemon.json 镜像加速器才能 build。

