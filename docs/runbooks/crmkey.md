# CRM环境变量Key清单

> 类型：操作规程 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

13 个 Workflow Key 是 NuPai AI 矩阵的入口；缺一个对应 WF 全失效。SalesEase 6 个 Key 是 CRM 数据同步命脉，泄露需立即在 SalesEase 平台轮换。.env 改了用 recreate 不是 restart。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
CRM 业务容器读取 /opt/nupai-crm/.env，关键是 13 个 Dify Workflow Key + SalesEase 集成。排查 AI 工作流或 SalesEase 同步失败时先看 Key。

## 内容
数据库/Redis（敏感）：MYSQL_HOST、MYSQL_PORT、MYSQL_DATABASE（值=nupaidb 历史命名）、MYSQL_USER、MYSQL_PASSWORD、REDIS_URL。 应用密钥（敏感）：JWT_SECRET。 AI 通用（敏感）：DEEPSEEK_API_KEY、DIFY_API_BASE、DIFY_API_KEY、DIFY_ASSIST_KEY、DIFY_SORTER_KEY。 Dify Workflow Key（敏感）：DIFY_WF1_KEY ~ DIFY_WF13_KEY（共 13 个）+ DIFY_WFM_KEY（Master）。 SalesEase CRM 集成（敏感）：SALESEASE_API_BASE、SALESEASE_CLIENT_ID、SALESEASE_CLIENT_SECRET、SALESEASE_USERNAME、SALESEASE_PASSWORD、SALESEASE_SECURITY_TOKEN。 非敏感：APP_ENV、SENTRY_DSN_FRONTEND。

## 结论
13 个 Workflow Key 是 NuPai AI 矩阵的入口；缺一个对应 WF 全失效。SalesEase 6 个 Key 是 CRM 数据同步命脉，泄露需立即在 SalesEase 平台轮换。.env 改了用 recreate 不是 restart。

## 影响
所有 AI 工作流（WF1-13 + Master + Assist + Sorter）；SalesEase 数据同步 cron；CRM cortex 12 个 scanner 部分依赖 Dify；Sentry 前端错误监控。

## 下次注意
新增 Workflow Key 按 DIFY_WF{N}_KEY 命名连续；SalesEase 凭据 6 件套必须配齐；改 .env 用 recreate；Key 命名按 {服务}_{用途} 规范；新增 AI 工作流前先在 .env 加好 Key 再上代码。

