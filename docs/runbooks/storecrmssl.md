# Store/CRM域名SSL现状

> 类型：操作规程 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

P1 待办两件：① 确认 store.fesuntailor.net 的 SSL 是否齐全（Let's Encrypt 续签？过期？）。② CRM nginx 配置真实域名（备案、DNS 解析、Let's Encrypt 申请、server_name 改 + reload）。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
基础设施清单 §9 整理两台服务器的域名和 SSL 现状，发现 CRM nginx server_name 还是 localhost，Store 部分域名 SSL 状态未确认。

## 内容
Store：store.fesuntailor.net → Store 前端 nginx，端口 8080，SSL ❓ 需确认。Dify Web → 走 docker-nginx-1 的 80/443，证书 Let's Encrypt（在 nginx 容器挂载）。 CRM：当前 server_name=localhost（基础设施清单 §15 P1），不是真实域名。容器 nupai-nginx 挂载 /etc/letsencrypt，端口 8090(HTTP) / 8493(HTTPS)。⚠️ server_name=localhost 状态下 HTTPS 证书匹配失败，访问会有 SSL 错误。

## 结论
P1 待办两件：① 确认 store.fesuntailor.net 的 SSL 是否齐全（Let's Encrypt 续签？过期？）。② CRM nginx 配置真实域名（备案、DNS 解析、Let's Encrypt 申请、server_name 改 + reload）。

## 影响
公网访问可达性、SSL 证书续签、阿里云域名备案、新接入终端的 https 信任、Sentry 前端 DSN 来源域校验。

## 下次注意
SSL 续签前 30 天告警必须配（Let's Encrypt 90 天有效期）；新增子域名一律先备案再签证书；修改 server_name 后 nginx -t 验证再 reload；CRM 真实域名上线后回头更新基础设施清单 §9 + 本卡。

