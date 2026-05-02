# 飞书OpenAPIscope必须tenant非user

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

飞书开放平台权限管理界面里，同名权限可能有 user 和 tenant 两个版本（用户身份 vs 应用身份），勾选时必须看"权限类型"列。OpenClaw cron / skill 用 tenant_access_token，所以必须勾 tenant 版本。重新勾 wiki:wiki:readonly 的 tenant 版本并发布 1.1.6 即解决。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
批 2 PoC B1 调飞书 wiki API 报 99991672"应用尚未开通所需的应用身份权限"。当时已勾 wiki:wiki:readonly 等 8 个 wiki scope 并发布版本，按理不该报错。

## 内容
通过 OpenAPI 查 openclaw 应用当前已生效 scope 列表，发现 8 个 wiki scope 的 token_types 字段全是 ['user']，但 PoC 用的是 tenant_access_token（应用身份）。错误链接 token_type=tenant 明确指向需要 tenant 权限。飞书后端拒绝 user scope 的 token 调要求 tenant 的 API。

## 结论
飞书开放平台权限管理界面里，同名权限可能有 user 和 tenant 两个版本（用户身份 vs 应用身份），勾选时必须看"权限类型"列。OpenClaw cron / skill 用 tenant_access_token，所以必须勾 tenant 版本。重新勾 wiki:wiki:readonly 的 tenant 版本并发布 1.1.6 即解决。

## 影响
任何对接飞书 OpenAPI 的脚本，如果用应用身份（tenant_access_token），必须确保所有 scope 都是 tenant 版本。

## 下次注意
飞书开放平台勾 scope 时，必看"权限类型"列。tenant_access_token 的应用必须勾"应用身份"版本。同名权限有 user/tenant 两版时不要勾错。可以用 OpenAPI 查 application/v6/applications/{app_id}/scopes 确认当前生效 scope 的 token_types 字段。

