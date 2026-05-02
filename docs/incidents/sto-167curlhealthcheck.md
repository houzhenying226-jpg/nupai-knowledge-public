# STO-167容器无curl导致HEALTHCHECK失败

> 类型：故障复盘 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:00:00Z

## 摘要

任何依赖额外 apt 安装的二进制做 HEALTHCHECK 都不可靠——build 失败、apt 源超时、镜像层错位都会让二进制缺失。HEALTHCHECK 一律用容器内必备工具：Python 容器→python3 urllib；Node 容器→fetch；不要为健康检查多装 curl/wget。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
NuPai 历史上 Store backend api 容器配置了 HEALTHCHECK 用 curl，但 Dockerfile.prod build 未完成时容器内没装上 curl，导致 HEALTHCHECK 命令找不到 curl 二进制、永久返回失败、docker ps 一直标 unhealthy（即使 API 进程在正常跑）。这是 STO-204 决策"HEALTHCHECK 禁用 curl 改 python3 urllib"的因。

## 内容
事故链路（基础设施清单 §18.3 简述）：backend/Dockerfile.prod 设计是 build 阶段 apt-get install curlbuild 流程因为 apt 网络/源问题未完成 → 镜像里没 curl 二进制容器启动后 HEALTHCHECK CMD curl -f http://localhost:8000/api/v1/health 执行找不到 curl，返回非零退出码Docker 按 HEALTHCHECK 协议把容器标 unhealthy实际 API 进程在跑，但健康状态错误，触发误告警 + 阻塞依赖容器健康的下游 后续：STO-204 全面切换 HEALTHCHECK 用 python3 urllib，删除对 curl 的依赖；backend/Dockerfile.prod 不再装 curl，HEALTHCHECK 指令也从 Dockerfile 移到 docker-compose.prod.yml（compose 覆盖唯一来源）。

## 结论
任何依赖额外 apt 安装的二进制做 HEALTHCHECK 都不可靠——build 失败、apt 源超时、镜像层错位都会让二进制缺失。HEALTHCHECK 一律用容器内必备工具：Python 容器→python3 urllib；Node 容器→fetch；不要为健康检查多装 curl/wget。

## 影响
所有自建容器的 HEALTHCHECK 设计；docker ps 健康状态准确性；deploy.sh 部署后健康检查链路；监控告警准确率；STO-204 决策的根因；新建容器服务的 HEALTHCHECK 模板（详见"STO-204 HEALTHCHECK禁用curl"卡）。

## 下次注意
HEALTHCHECK 选工具优先选容器内必备（Python→python3 urllib，Node→fetch）；写 Dockerfile 不要为 HEALTHCHECK 多装二进制；HEALTHCHECK 定义放 compose 不放 Dockerfile（避免双源冲突）；遇到 docker ps 标 unhealthy 但 API 实际可达，先怀疑 HEALTHCHECK 命令本身不是服务挂了。

