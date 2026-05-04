# NuPai命名规范铁律

> 类型：架构决策 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-04T06:35:58Z

## 摘要

新增容器/数据库/环境变量/分支/端口一律按以上规范，CR 时严卡。历史遗留（CRM 容器名、库名 nupaidb）不改但新项目不能再这样做。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
当前 CRM 容器名（nupai-mysql、nupai-fastapi）和库名（nupaidb）是历史遗留，没有项目前缀。基础设施清单 §16 写死新规范，新增资源必须遵守。

## 内容
容器命名：nupai-{项目}-{服务}-{序号}，例 nupai-store-api-1 / nupai-crm-api-1。禁止：nupai-fastapi（缺项目名）/ api-server（缺 nupai 前缀）。 数据库命名：nupai_{项目}，例 nupai_store / nupai_crm。现状：Store=nupai / CRM=nupaidb（历史遗留，不改但新项目必须遵守）。 环境变量命名：{服务}_{用途}，例 DIFY_API_KEY / MYSQL_PASSWORD / REDIS_URL。敏感变量必须放 .env 不能写在 docker-compose.yml。 compose 文件分工：docker-compose.yml → 所有共享基础服务（db/redis/nginx/主 API）；docker-compose.prod.yml → 有独立 Dockerfile 的服务 + 生产覆盖。新增服务：和 API 共用 Dockerfile → 放 base yml；有独立 Dockerfile → 放 prod yml。 端口规则：{项目基数}+{服务偏移}。Store 基数 8000：API=8001、nginx=8080。CRM 基数 8000：API=8001、nginx HTTP=8090、HTTPS=8493。新端口不能和现有冲突（查端口表）；数据库/Redis 只监听 127.0.0.1。 分支命名：{type}/{issue 前缀}-{N}-{简述}，例 feat/sto-147-wecom-sync / fix/nup-223-visit-nav。Store 前缀 sto-（Linear STO-），CRM 前缀 nup-（Linear NUP-）。

## 结论
新增容器/数据库/环境变量/分支/端口一律按以上规范，CR 时严卡。历史遗留（CRM 容器名、库名 nupaidb）不改但新项目不能再这样做。

## 影响
新建容器/库/字段/分支的命名习惯；CR 模板；hooks（如 enforce-commit-format.sh）规则；OPS-COMMANDS.md 命令模板；新人 onboarding。

## 下次注意
CR 看到 nupai-{服务}（缺项目）的 PR 直接打回；新项目库名一定 nupai_{项目} 不要再用裸名；分支前缀和 Linear 项目对齐（sto-/nup-）；新端口先查端口表防冲突；数据库/Redis 默认 127.0.0.1: 不要 0.0.0.0:。

