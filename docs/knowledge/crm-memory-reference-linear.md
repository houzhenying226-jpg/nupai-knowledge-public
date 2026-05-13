# Linear API 配置

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T14:43:26Z

## 摘要

Linear API key 和 Nupai 团队/项目 ID，用于查询和管理 Issues

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `reference_linear.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## Linear API

- API Key: `[REDACTED_LINEAR_API_KEY]`
- Endpoint: `https://api.linear.app/graphql`
- Auth Header: `Authorization: [REDACTED_AUTH]

## Nupai 团队

- Team ID: `49f4f125-2624-4248-8df1-270094f32a8f`
- Team Key: `NUP`

## Project

- nupai-crm v1.0 ID: `2f7a295c-66b4-46eb-9e39-0f20c37d881a`

## Labels

- Bug: `3bca0493-e0a2-416b-bc05-48fd228ff8e7`
- Feature: `35cfa815-d777-45d5-a41a-df578170a205`
- Improvement: `c29a0c0e-a313-4197-8f54-873606fb8c3c`
- Patch: `d0077bf9-15a8-4273-8fcf-844363476b15`

## Workflow States

- Backlog: `0bdc9ea3-553f-4936-979e-ea71ef6a3df9`
- Todo: `4b6056cc-f7ad-4ea9-93db-30cdb8ed8ac8`
- In Progress: `09139a4a-dff2-4854-ae1a-bce6f22c11a9`
- In Review: `6e340b1b-21c5-45c2-a2de-a70497c10719`
- Done: `c8ef7403-a6e6-4a25-bb13-3e91ffadea6a`
- Canceled: `9f49ae75-f673-4467-952f-af77c21dc3ae`
- Duplicate: `bbba159a-2e24-4aa1-b38c-7551736fecc5`

## Priority Values

- Urgent: 1, High: 2, Medium: 3, Low: 4, No priority: 0

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

