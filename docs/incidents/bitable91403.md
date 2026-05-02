# 飞书Bitable写入91403两层权限

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

飞书 OpenAPI 写多维表/Wiki 文档需要两层权限，缺一即 91403：① 开放平台 scope + 发布版本（应用级权限）② Bitable/Wiki 内执行"添加文档应用"（资源级权限）。第二层入口在 Bitable 右上角 ··· → 添加文档应用 → 选应用 → 可编辑。"添加协作者"对应用 OpenAPI 写操作无效，必须走"添加文档应用"。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
批 1 用 OpenAPI 自动建飞书 NuPai-AI-底座管理台账的 3 张多维表，调 PATCH/POST 时一直返回 91403 Forbidden，排查 1 小时未果。

## 内容
已确认开放平台 scope（bitable:* 全勾）已勾选并发布版本 1.1.5。Bitable 协作者列表也加了 cli_a93a858dd3389e19 权限"可编辑"。但 PATCH 改表名 / POST 建表 / 写记录全部 91403。app_token=AUssbTyUlaMboks3gsKjMmvqp5d。

## 结论
飞书 OpenAPI 写多维表/Wiki 文档需要两层权限，缺一即 91403：① 开放平台 scope + 发布版本（应用级权限）② Bitable/Wiki 内执行"添加文档应用"（资源级权限）。第二层入口在 Bitable 右上角 ··· → 添加文档应用 → 选应用 → 可编辑。"添加协作者"对应用 OpenAPI 写操作无效，必须走"添加文档应用"。

## 影响
所有用 OpenAPI 操作飞书多维表 / Wiki 文档的脚本、cron、skill。NuPai 项目下涉及 C1 feishu-wiki-sync、knowledge-writer、bitable 初始化等。

## 下次注意
对接飞书 OpenAPI 写文档前必查两层权限：第一层飞书开放平台"权限管理"勾完后必须发版本；第二层去文档本身右上角"添加文档应用"，权限选"可编辑"。仅勾 scope 不"添加文档应用"会报 91403。

