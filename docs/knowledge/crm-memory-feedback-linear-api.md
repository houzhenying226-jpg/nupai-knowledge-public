# Linear API 必须用 curl

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-18T12:50:39Z

## 摘要

禁止用 MCP 连接 Linear，必须用 curl 调 GraphQL API，MCP 连接不稳定

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_linear_api.md`，本次按侯爷要求同步到飞书知识库。

## 内容
禁止使用 MCP 连接 Linear，必须用 curl 调 Linear GraphQL API。

**Why:** MCP 连接 Linear 不稳定，经常失败。

**How to apply:** 所有 Linear 操作（读 Issue、建 Issue、更新状态、加评论）一律用 curl：
```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: [REDACTED_AUTH] \
  -d '{"query":"..."}'
```
绝不调用 mcp__linear__* 工具。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

