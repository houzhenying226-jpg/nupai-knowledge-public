# Store监控容器3个清单

> 类型：操作规程 | 项目：nupai-store | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:00:00Z

## 摘要

Grafana 3000 + Prometheus 9090 当前公网开放，存在信息泄露风险，建议改 127.0.0.1: 监听或加认证。Alertmanager 仅内部端口安全。

## 正文

> 记录日期：2026-04-17  风险等级：L2

## 背景
Store 服务器除业务容器外还跑 3 个监控容器（Grafana/Prometheus/Alertmanager），排查指标、告警时需要知道容器名和访问方式。

## 内容
nupai-store-grafana-1 → grafana/grafana:latest，0.0.0.0:3000→3000，监控面板，⚠️ 监听 0.0.0.0 公网暴露（基础设施清单 §15 P1 风险）。 nupai-store-prometheus-1 → prom/prometheus:latest，0.0.0.0:9090→9090，指标采集，⚠️ 同样公网暴露。 nupai-store-alertmanager-1 → prom/alertmanager:v0.27.0，9093 内部端口，报警管理。

## 结论
Grafana 3000 + Prometheus 9090 当前公网开放，存在信息泄露风险，建议改 127.0.0.1: 监听或加认证。Alertmanager 仅内部端口安全。

## 影响
监控告警链路、Grafana 面板访问、Prometheus 数据采集、安全审计、未来阿里云安全组规则收紧。

## 下次注意
访问 Grafana 必须立刻评估改 127.0.0.1: + Tailscale 访问或加认证；Prometheus 同步收紧；Alertmanager 已经只内部，不动。

