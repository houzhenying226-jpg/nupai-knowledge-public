# L14 · SQLAlchemy 列表查询必须用 selectinload 消除 N+1

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

167ms 返回 3375 条（vs 修复前 >5000ms），N+1 完全消除。所有列表接口的 SQLAlchemy 查询必须加 selectinload。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
BF-7，Deals Kanban 查 200 条触发 400+ 次额外 SQL，resolve_names() 逐条查 account/owner。

## 内容
修复前 200条×2 = 400+ SQL。修复方案：stmt = select(CrmDeal).options(selectinload(CrmDeal.account), selectinload(CrmDeal.owner))。

## 结论
167ms 返回 3375 条（vs 修复前 >5000ms），N+1 完全消除。所有列表接口的 SQLAlchemy 查询必须加 selectinload。

## 影响
Deals Kanban 响应时间从 >5s 降至 <200ms，横向适用于所有列表关联查询。

## 下次注意
列表查询凡是涉及关联字段渲染 → 一律加 selectinload，严禁逐条 resolve。

