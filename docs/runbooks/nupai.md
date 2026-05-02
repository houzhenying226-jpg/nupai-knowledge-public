# NuPai第三方服务依赖清单

> 类型：操作规程 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

任一外部服务挂掉的影响范围：Dify→AI 工作流全停；DeepSeek→PR 摘要 + AI 分析停；钉钉→Store 通知停；OSS→Store 文件上传停；GitHub→部署链路全停；飞书→团队通知 + 知识入库停。SalesEase 是 CRM 数据同步命脉。Skyvern 是 CRM 自动化命脉。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
NuPai 强依赖 13 个外部服务，任一挂掉都影响业务/部署/AI/通知。需要一卡看清全部外部依赖、用途、关键配置。

## 内容
AI 类：Dify（Store 服务器自部署，80/443）→ AI 编排（教练/简报/WF1-13），关键配置 DIFY_API_KEY + 13 个 WF Key。DeepSeek（外部 API）→ PR 中文摘要 + AI 分析，关键 DEEPSEEK_API_KEY。阿里百炼（外部 API，仅 Store）→ AI 补充，关键 BAILIAN_API_KEY。 集成类：SalesEase（外部 API，仅 CRM）→ CRM 数据同步，关键 SALESEASE_* 6 件套。钉钉（仅 Store，dingtalk-bot 容器对接）→ 客户通知，关键 DINGTALK_* 5 件套。阿里云 OSS（仅 Store）→ 文件存储，关键 OSS_* 5 件套。阿里云短信（仅 Store）→ 短信发送，关键 ALIYUN_SMS_* 4 件套。Skyvern（CRM 服务器自部署，6080/8000/9222）→ 浏览器自动化，独立项目。 监控/通知/CI 类：Sentry（外部 SaaS）→ 错误监控，关键 SENTRY_DSN（Store）+ SENTRY_DSN_FRONTEND（CRM）。GitHub（外部 SaaS）→ 代码托管 + CI/CD，关键 deploy_key + Secrets。Linear（外部 SaaS）→ Issue 管理，API 通过 Claude Code。飞书（外部 SaaS）→ 团队通知 + OpenClaw 知识入库，关键 FEISHU_WEBHOOK，域名用 open.larksuite.com。CodeRabbit（GitHub App）→ AI 代码审查，自动触发。

## 结论
任一外部服务挂掉的影响范围：Dify→AI 工作流全停；DeepSeek→PR 摘要 + AI 分析停；钉钉→Store 通知停；OSS→Store 文件上传停；GitHub→部署链路全停；飞书→团队通知 + 知识入库停。SalesEase 是 CRM 数据同步命脉。Skyvern 是 CRM 自动化命脉。

## 影响
所有 .env Key、deploy.sh、CI/CD 流程（GitHub Required Checks）、AI 矩阵、通知链路、备份异地（阿里云）、新人 onboarding 凭据列表。

## 下次注意
新增第三方服务必须更新本卡；外部 API Key 泄露立即在对应平台轮换；GitHub Secrets 变更同步基础设施清单 §13；Dify 升级前先备份 db_postgres + weaviate；飞书 webhook 域名是 open.larksuite.com 不是 feishu.cn。

