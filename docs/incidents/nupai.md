# NuPai公网暴露高危端口清单

> 类型：故障复盘 | 项目：_global | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

P0 立即处置：CRM 9222 改监听 127.0.0.1: 或在阿里云安全组里黑名单。P1 顺次：Grafana/Prometheus 改本机 + Tailscale 访问；plugin 5003 评估必要性；CRM nginx server_name 改真实域名。P2 评估是否启 ufw/firewalld（注意 Store Docker 用 firewalld 会破坏网络）。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
2026-04-17 整理基础设施清单时审计两台服务器端口，发现多个本应内网的服务对公网开放，存在数据/控制面泄露风险。

## 内容
🔴 P0 高危：CRM 9222 → Skyvern Chrome DevTools，远程调试任意网页可被劫持，必须立即关。 🟠 P1 风险：Store 3000 → Grafana 监控面板对公网（建议改 127.0.0.1: 或加认证）；Store 9090 → Prometheus 指标对公网（建议改 127.0.0.1:）；Store 5003 → Dify plugin_daemon 对公网（评估必要性）；CRM 6080 → Skyvern UI/控制台对公网；CRM 8000 → Skyvern API 对公网；CRM nginx server_name=localhost（不是真实域名，HTTPS 证书匹配会失败）。 🟡 P2：Store 防火墙无额外规则（iptables INPUT ACCEPT），全靠阿里云安全组兜底，安全基线不够。

## 结论
P0 立即处置：CRM 9222 改监听 127.0.0.1: 或在阿里云安全组里黑名单。P1 顺次：Grafana/Prometheus 改本机 + Tailscale 访问；plugin 5003 评估必要性；CRM nginx server_name 改真实域名。P2 评估是否启 ufw/firewalld（注意 Store Docker 用 firewalld 会破坏网络）。

## 影响
基础设施清单 §15 待办、阿里云安全组规则、Docker 端口映射改写、ufw 规则、SSL 证书匹配、安全审计报告。

## 下次注意
所有新服务上线前默认 127.0.0.1: 监听，对外端口必须走阿里云安全组白名单；Skyvern 升级前先确认 9222 是否仍开放；监控类（Grafana/Prometheus）一律不对公网；CRM 配真实域名后再签 SSL。

