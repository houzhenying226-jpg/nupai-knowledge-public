# PR Gate rerun 后需查 attempt/2，check-runs 显示旧结果

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-14T13:20:33Z

## 摘要

gh run rerun --failed 创建 attempt 2，不生成新 run_id；check-runs API 可能仍显示旧结论，需指定 attempt/2 查看新结果

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_pr_gate_rerun_attempt2.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

执行 `gh run rerun 25712054759 --failed` 后，`gh pr view 572` 的 statusCheckRollup 仍然显示旧的 failure，不知道 rerun 是否生效。

## 结论

`gh run rerun <run-id> --failed` 的行为：
- **不生成新 run_id** — workflow_runs 列表中不会出现新条目
- **创建 attempt 2** — 在原 run_id 下新增一次尝试
- **check-runs API 可能显示旧结果** — 因为聚合的是最新 check_run 记录，rerun 后记录更新需要时间

## 正确的 rerun 结果查看方式

```bash
# ❌ 错误：看这个可能还是旧结果
gh pr view 572 --json statusCheckRollup

# ✅ 正确：直接查 attempt 2 的 jobs
gh api "repos/houzhenying226-jpg/nupai-crm/actions/runs/25712054759/attempts/2/jobs" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for job in data.get('jobs', []):
    print(job['name'], '|', job['status'], '|', job['conclusion'] or 'running')
"
```

## 失败 job 的日志查看

```bash
# 1. 从 attempt/2/jobs 找到 job id
# 2. 用 job id 查日志
gh api repos/houzhenying226-jpg/nupai-crm/actions/jobs/<job_id>/logs | tail -40
```

## 完整 rerun 流程

```bash
# 重跑 PR Gate 失败的 job
gh run rerun 25712054759 --failed

# 重跑 OpenClaw Gate 失败的 job  
gh run rerun 25712054779 --failed

# 30-60秒后查看 attempt/2 结果
gh api "repos/houzhenying226-jpg/nupai-crm/actions/runs/25712054759/attempts/2/jobs" 2>&1 | python3 -c "..."
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

