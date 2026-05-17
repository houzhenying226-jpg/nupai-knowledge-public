# deploy.yml 健康检查假通过 Bug

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-17T03:40:24Z

## 摘要

CRM 未部署也能通过健康检查，因为 Store API 也监听 8001 端口

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_health_check_false_positive.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因

`deploy.yml` verify 步骤 SSH 到 DEPLOY_HOST（[REDACTED_IP]）检查：
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/health
```

**问题**：这两台服务器的 8001 端口都被占用：
- [REDACTED_IP]：CRM API（正确的）
- [REDACTED_IP]：Store API（`nupai-store-api-1` 映射了 `[REDACTED_IP]:8001->8000/tcp`）

过去曾误以为 DEPLOY_HOST=[REDACTED_IP]，所有 CRM 部署"验证通过"实际验证的是 Store API。

## 影响

GitHub Actions 显示 Deploy ✅、健康检查 ✅，但 CRM 可能根本没有部署成功。
这个 bug 导致定位问题时在错误服务器上查了很久。

## 正确诊断方式

排查 CRM 问题必须先确认服务器 IP：
```bash
# 1. 找对服务器
ssh root@[REDACTED_IP] "docker ps | grep nupai-fastapi"

# 2. CRM 容器内健康检查（不依赖端口暴露）
ssh root@[REDACTED_IP] "docker exec nupai-fastapi curl -s http://localhost:8001/health"
```

## 修复建议（待 Issue）

deploy.yml 的 verify 步骤应改为容器内检查，避免端口冲突导致假通过：
```bash
docker exec nupai-fastapi curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/health
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

