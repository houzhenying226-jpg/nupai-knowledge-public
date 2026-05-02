# CRM服务器Skyvern容器

> 类型：操作规程 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

9222 必须立即关闭（远程调试任意网页可被劫持）。8000、6080 也要评估收紧。skyvern-ui 已停可考虑彻底移除。

## 正文

> 记录日期：2026-04-17  风险等级：L2

## 背景
Skyvern（浏览器自动化）和 CRM 同台服务器但独立项目，3 个容器，端口 9222 是高危公网 Chrome DevTools。

## 内容
skyvern-skyvern-1（public.ecr.aws/skyvern/skyvern:latest）→ 0.0.0.0:6080,8000,9222，浏览器自动化，状态 Up，⚠️ 9222 是 Chrome DevTools 远程调试端口对公网开放（基础设施清单 §15 P0 高危）。 skyvern-skyvern-ui-1（public.ecr.aws/skyvern/skyvern-ui:latest）→ 无端口映射，状态 Exited（已停止）。 skyvern-postgres-1（postgres:14-alpine）→ 5432 内部，Skyvern 数据库，状态 Up。

## 结论
9222 必须立即关闭（远程调试任意网页可被劫持）。8000、6080 也要评估收紧。skyvern-ui 已停可考虑彻底移除。

## 影响
CRM 服务器安全基线；Skyvern 浏览器自动化对外服务；阿里云安全组规则；CRM ufw 规则。

## 下次注意
9222 立即从公网移除（改 127.0.0.1: 或安全组黑名单）；6080/8000 收紧白名单；skyvern-ui 长期 Exited 评估是否清理；Skyvern 升级镜像前先备份 skyvern-postgres-1 数据。

