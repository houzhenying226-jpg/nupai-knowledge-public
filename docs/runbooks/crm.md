# CRM端口映射全表

> 类型：操作规程 | 项目：nupai-crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-04T02:05:57Z

## 摘要

9222 必须立即关。6080/8000 评估收紧。ufw 规则需要和实际端口对齐（8090/8493 不在 ufw 列表）。CRM nginx server_name 当前是 localhost（基础设施清单 §15 P1），需要配真实域名。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
CRM 服务器除业务容器还跑 Skyvern 浏览器自动化，9222 端口是高危公网 Chrome DevTools。需要一卡看清。

## 内容
✅ 公网开放（监听 0.0.0.0）：8090 → CRM nginx HTTP；8493 → CRM nginx HTTPS；6080 → Skyvern ⚠️（P1）；8000 → Skyvern API ⚠️（P1）；9222 → Skyvern Chrome DevTools ⚠️ P0 高危远程调试。 ❌ 内部/仅本机：8001 → CRM FastAPI（127.0.0.1）；3000 → CRM Next.js（127.0.0.1）；3306 → MySQL（127.0.0.1）；6379 → Redis（127.0.0.1）。 端口规则：CRM 基数 8000 → API 8001、nginx HTTP 8090、HTTPS 8493。 ufw 当前 ALLOW：22 / 80 / 443 / 8080 / 8443 / 9000（注意这里 ufw 列了 80/443/8080/8443/9000 但实际业务端口是 8090/8493，配置和现实有偏差）。

## 结论
9222 必须立即关。6080/8000 评估收紧。ufw 规则需要和实际端口对齐（8090/8493 不在 ufw 列表）。CRM nginx server_name 当前是 localhost（基础设施清单 §15 P1），需要配真实域名。

## 影响
nginx 反代配置、ufw 规则、阿里云安全组、Skyvern 安全风险、SSL 证书配置（/etc/letsencrypt 已挂载但 server_name=localhost 不生效）。

## 下次注意
9222 立即从公网移除；ufw 规则按实际业务端口重写（8090/8493）；CRM nginx server_name 改成真实域名；新增端口前查本卡和基础设施清单 §5。

