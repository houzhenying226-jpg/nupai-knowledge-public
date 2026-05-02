# 飞书Wiki多维表协作者无写权限

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

新建独立 Bitable（不在 Wiki 下），用 /base/ 路径而非 /wiki/。独立 Bitable 的"添加文档应用"对应用 OpenAPI 写操作真有效。批 1 实际方案：建独立 Bitable APP_TOKEN=AUssbTyUlaMboks3gsKjMmvqp5d，把表 A/B/C 全建在这里。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
批 1 第一次建飞书表，把表 A 放在 Wiki 知识库下面（NuPai 知识库 / NuPai-AI-底座管理台账）。调 PATCH/POST 全部 91403，即使加了协作者+发了版本+添加文档应用全套都试过。

## 内容
飞书 Wiki 节点下的多维表权限模型与独立 Bitable 不同：协作者权限（哪怕"可编辑"）只对人类用户有效，对应用 OpenAPI 写操作无效。Wiki 空间的"添加协作者"入口走的是通讯录系统，不支持加应用。要给 Wiki 下的多维表加应用授权，必须逐篇文档手动"添加文档应用"，无法批量自动化。

## 结论
新建独立 Bitable（不在 Wiki 下），用 /base/ 路径而非 /wiki/。独立 Bitable 的"添加文档应用"对应用 OpenAPI 写操作真有效。批 1 实际方案：建独立 Bitable APP_TOKEN=AUssbTyUlaMboks3gsKjMmvqp5d，把表 A/B/C 全建在这里。

## 影响
NuPai 系统所有需要应用 OpenAPI 写入的多维表，都不能放 Wiki 下。Wiki 角色降级为人类阅读层，不作为数据真源。

## 下次注意
任何要让 OpenClaw cron / skill 自动写入的飞书多维表，必须新建独立 Bitable（飞书"+ 新建" → "多维表格"，URL 含 /base/），不要建在 Wiki 知识库下面。

