# 必须通过PR流程

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-19T14:00:00Z

## 摘要

所有改动必须创建PR，禁止直接push到main分支

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_pr_workflow.md`，本次按侯爷要求同步到飞书知识库。

## 内容
所有代码改动必须通过 PR 流程提交，不能直接推送 main 分支。

**Why:** 用户明确要求，直接push main不符合团队工作流规范。

**How to apply:** 每次修改代码后，创建feature分支→push分支→gh pr create，绝不直接commit到main再push。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

