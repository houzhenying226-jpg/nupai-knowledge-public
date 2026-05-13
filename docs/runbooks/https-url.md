# 企业微信侧边栏配置必须用HTTPS URL

> 类型：操作规程 | 项目：nupai-store | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

企业微信后台 → 应用管理 → 找到侧边栏应用 → 编辑 → 主页 URL 改成 [https://store.ifeishen.com/wecom-sidebar](https://store.ifeishen.com/wecom-sidebar)。

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
企业微信后台配置侧边栏主页时填了 http:// 的地址，虽然服务器有 HTTP→HTTPS 重定向，但 WeCom 内置浏览器行为不一致，建议直接用 HTTPS。

## 内容
正确的企业微信后台配置：
- 可信域名：[store.ifeishen.com](store.ifeishen.com)（不含协议头）
- 侧边栏主页 URL：[https://store.ifeishen.com/wecom-sidebar](https://store.ifeishen.com/wecom-sidebar)（必须 HTTPS）
服务器情况：
- [store.ifeishen.com](store.ifeishen.com) → [39.96.216.33](39.96.216.33)
- 80 端口有 HTTP→HTTPS 301 重定向
- SSL 证书已配置，HTTPS 正常

## 结论
企业微信后台 → 应用管理 → 找到侧边栏应用 → 编辑 → 主页 URL 改成 [https://store.ifeishen.com/wecom-sidebar](https://store.ifeishen.com/wecom-sidebar)。

## 影响
- 企业微信管理后台配置（需人工在后台修改）

## 下次注意
WeCom 配置所有 URL 一律用 HTTPS。HTTP URL 在某些 WeCom 版本会被直接拒绝或不触发 JS-SDK 鉴权。

