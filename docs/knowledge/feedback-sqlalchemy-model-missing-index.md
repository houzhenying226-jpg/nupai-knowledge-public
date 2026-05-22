# L13 · SQLAlchemy 模型必须为所有高频查询维度定义 Index

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

新建模型时，逐一列出所有 WHERE 维度，全部加 Index。所有外键出现在查询条件中的列 → Alembic 迁移显式 create_index。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
NUP-588，crm_activities 缺 account_id 索引，客户详情页活动查询全表扫描。

## 内容
account_id 是最高频查询维度，建模时没加 Index。revision ID 必须 ≤32 字符（VARCHAR(32) 约束）。AI 不能写 alembic/versions/，必须用户手动 SCP from 服务器。

## 结论
新建模型时，逐一列出所有 WHERE 维度，全部加 Index。所有外键出现在查询条件中的列 → Alembic 迁移显式 create_index。

## 影响
消除全表扫描，客户详情页活动查询速度大幅提升。

## 下次注意
建模时列出所有查询维度 → 全部加 Index；alembic 迁移文件必须从服务器 SCP。

