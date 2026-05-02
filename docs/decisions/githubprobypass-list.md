# GitHubPro是bypass-list前置条件

> 类型：架构决策 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

v2.1 版本治理子系统的硬前置条件是 GitHub Pro+ 计划。如果 GitHub Free 计划，必须先升级 Pro，或改用其他不依赖 bypass list 的方案（比如手动审批每个 docs-update PR，但失去自动化价值）。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
v2.1 版本治理方案 Step 6 要求 Branch Protection bypass list 加 openclaw-bot 账号，否则 post-deploy 的 gh pr merge --auto 会因等待 approval 永久挂起。但 bypass list 是 GitHub Pro/Team/Enterprise 才有的功能。

## 内容
GitHub Free 计划没有 "Allow specified actors to bypass required pull request reviews" 这个选项，整套 v2.1 自动化链路无法工作。GitHub Pro 计划（含个人账号）支持。落地前必须先确认 GitHub 计划。振英已确认 GitHub Pro 个人账号，仓库在个人账号下（houzhenying226-jpg/nupai-crm），bypass list 功能可用。

## 结论
v2.1 版本治理子系统的硬前置条件是 GitHub Pro+ 计划。如果 GitHub Free 计划，必须先升级 Pro，或改用其他不依赖 bypass list 的方案（比如手动审批每个 docs-update PR，但失去自动化价值）。

## 影响
未来给其他 GitHub 账号 / 组织部署版本治理 v2.1，落地前必查 Settings → Billing & plans 确认计划。免费计划项目要么升 Pro，要么放弃自动化部分。

## 下次注意
任何依赖 GitHub 高级功能的方案（bypass list / GitHub Apps / GraphQL API 部分功能 / required reviewers 等），落地前必查 GitHub 计划。从"现状摸底"步骤就要列出"所需 GitHub 计划级别"作为前置条件，避免实施一半发现功能没有要返工。

