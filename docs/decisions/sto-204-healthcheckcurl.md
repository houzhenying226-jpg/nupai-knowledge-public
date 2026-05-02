# STO-204 HEALTHCHECK禁用curl

> 类型：架构决策 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

HEALTHCHECK 一律用容器内必备工具（Python→python3 urllib，Node→fetch），不要为了健康检查多装二进制。HEALTHCHECK 定义放 docker-compose.yml/prod.yml 不放 Dockerfile（compose 覆盖唯一来源避免双源冲突）。

## 正文

> 记录日期：2026-04-20  风险等级：L3

## 背景
STO-204 + STO-167 复盘：HEALTHCHECK 命令依赖 curl 在 NuPai 出过事故——容器内没装 curl 或 build 未完成时 curl 缺失，导致 HEALTHCHECK 永久失败、Docker 把容器标 unhealthy。基础设施清单 §15 + §18.3 写死规则。

## 内容
禁止 HEALTHCHECK 用 curl：HEALTHCHECK CMD curl -f http://localhost:8000/api/v1/health。必须用容器内保证存在的工具：Python 容器（NuPai backend api/scheduler/event-consumer/cortex）→ python3 urllib；Node 容器（CRM nextjs）→ node fetch。当前 Store api 落地（基础设施清单 §15 STO-204 更新）：定义位置 docker-compose.prod.yml api service healthcheck（不在 Dockerfile）；命令：python3 -c 'import urllib.request; urllib.request.urlopen("http://localhost:8000/api/v1/health", timeout=5)'；backend/Dockerfile.prod 不再装 curl，HEALTHCHECK 指令也从 Dockerfile 移除（compose 覆盖为唯一来源）。历史备注：PR#65 加 curl 到 backend/Dockerfile（dev 版）不影响生产 prod 镜像，本 PR 彻底改方案。

## 结论
HEALTHCHECK 一律用容器内必备工具（Python→python3 urllib，Node→fetch），不要为了健康检查多装二进制。HEALTHCHECK 定义放 docker-compose.yml/prod.yml 不放 Dockerfile（compose 覆盖唯一来源避免双源冲突）。

## 影响
所有自建容器的 Dockerfile 和 compose 配置；docker ps 健康状态准确性；deploy.sh 部署后健康检查；监控容器 NOT RUNNING/UNHEALTHY 告警准确率；新建服务的 HEALTHCHECK 模板。

## 下次注意
新建容器 HEALTHCHECK 一律选容器内必备工具不依赖 curl；HEALTHCHECK 定义放 compose 不放 Dockerfile；CR Dockerfile 改动检查 HEALTHCHECK 是否引入 curl；任何“装 curl 只为健康检查”的 PR 直接打回。

