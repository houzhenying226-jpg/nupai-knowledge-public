# 镜像打包容器长期未rebuild

> 类型：故障复盘 | 项目：_global | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

镜像打包类容器一律必须在 deploy.sh 的 rebuild 列表里。新增有独立 Dockerfile 的服务一律放 docker-compose.prod.yml + 加 deploy.sh + 同步基础设施清单 §4。手动 rebuild 命令模板：docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d -

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
2026-04-17 整理基础设施时审计容器创建时间，发现 Store dingtalk-bot 27 天未 rebuild、CRM cortex 37 天未 rebuild。代码已经合并 N 次，生产跑的还是旧镜像，钉钉通知和 cortex 12 个 scanner 的逻辑都没生效。

## 内容
事故根因：dingtalk-bot 和 cortex 是镜像打包类容器（独立 Dockerfile，分别在 dingtalk-bot/Dockerfile 和 cortex/Dockerfile），代码烧进镜像。但 deploy.sh 旧版的 rebuild 列表里没有它们，开发者也用 docker compose restart 这种只对本地挂载有效的命令。具体修复（2026-04-17）：Store deploy.sh v3 加 dingtalk-bot + nginx + 新鲜度检查（PR #56）→ 振英手动 rebuild dingtalk-bot 容器；CRM deploy.sh 加 cortex + 回滚消息矩阵（PR #257）→ 振英手动 rebuild cortex 容器；同步生成基础设施清单 §4 容器更新方式表（本地挂载 vs 镜像打包）；同步建立“compose 双 -f 加载”铁律（dingtalk-bot/cortex 都在 prod.yml）。

## 结论
镜像打包类容器一律必须在 deploy.sh 的 rebuild 列表里。新增有独立 Dockerfile 的服务一律放 docker-compose.prod.yml + 加 deploy.sh + 同步基础设施清单 §4。手动 rebuild 命令模板：docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build <service>。

## 影响
deploy.sh 服务列表正确性；所有镜像打包类容器（Store dingtalk-bot / CRM nextjs / CRM cortex）的部署；新增 service 流程；CR 检查项（独立 Dockerfile = 必须加 deploy.sh rebuild）；新鲜度检查脚本设计。

## 下次注意
新增 service 有独立 Dockerfile 必走三件套：① 放 docker-compose.prod.yml ② 加进 deploy.sh rebuild 列表 ③ 同步基础设施清单 §4；deploy.sh 加新 service 后做“改一行注释 → 部署 → 看容器创建时间是否更新”演练；新鲜度检查脚本要检测“超过 X 天未 rebuild”主动告警（防止再发生 27/37 天事故）。

