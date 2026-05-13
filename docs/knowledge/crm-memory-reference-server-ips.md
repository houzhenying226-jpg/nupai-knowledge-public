# 三台服务器 IP 与职责

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T19:45:00Z

## 摘要

CLAUDE.md 写错 CRM IP，查障时浪费大量时间的教训

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `reference_server_ips.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 正确映射（以 OPS-COMMANDS.md 为准）

| 服务器 | IP | 职责 |
|--------|-----|------|
| CRM 生产 | **[REDACTED_IP]** | nupai-fastapi / nupai-nextjs / nupai-cortex / nupai-mysql / nupai-redis |
| Store 生产 | **[REDACTED_IP]** | nupai-store-api-1 / scheduler / event-consumer / postgres / redis |
| 异地备份 | **[REDACTED_IP]** | /opt/backups/crm/ + /opt/backups/store/ 两个系统的备份存储 |

## CLAUDE.md 的错误

CLAUDE.md 写的是 `root@[REDACTED_IP]`（实际是 Store 服务器），CRM 不在那里。
下次修 CLAUDE.md 时需要同步修正。

## 端口映射

| 服务 | 服务器 | 宿主机端口 |
|------|--------|-----------|
| CRM API | [REDACTED_IP] | 8001 |
| Store API | [REDACTED_IP] | 8001（与 CRM 端口号相同，但不同机器！）|

## 诊断命令（直接可用）

```bash
# CRM 健康
ssh root@[REDACTED_IP] "curl -s -o /dev/null -w '%{http_code}' http://localhost:8001/health"

# Store 健康
ssh root@[REDACTED_IP] "curl -s -o /dev/null -w '%{http_code}' http://localhost:8001/api/v1/health"

# CRM 容器状态
ssh root@[REDACTED_IP] "docker ps | grep nupai-fastapi"
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

