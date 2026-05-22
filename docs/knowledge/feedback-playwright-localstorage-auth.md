# L17 · Playwright E2E 测试 CRM：认证在 localStorage 不在 Cookie

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

Playwright CRM 认证必须通过 localStorage Bearer token + page.evaluate(fetch) 模式，不能用 page.request.get()。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
page.request.get() 返回 401，因为 Next.js SPA 认证 token 存在 localStorage 而非 Cookie。

## 内容
正确做法是 page.evaluate(fetch) + Bearer token。登录用 waitUntil: 'load'，不用 'networkidle'（Next.js SPA 永远不 idle）。生产测试账号：13911222699/NuPai2026（CEO 完整权限）。

## 结论
Playwright CRM 认证必须通过 localStorage Bearer token + page.evaluate(fetch) 模式，不能用 page.request.get()。

## 影响
E2E 测试稳定化，不再因为认证方式错误导致 401 失败。

## 下次注意
CRM E2E 测试统一用 page.evaluate(fetch) 模式；生产账号固定使用 CEO 账号。

