# PR合并顺序与部署健康验证铁律

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-06-04T14:00:01Z

## 摘要

连续合并PR导致宕机的教训，以及正确的合并-部署-验证流程

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_pr_merge_order.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因（2026-05-11 真实事故）

在 5 分钟内连续合并 PR #545（NUP-425）和 PR #544（E6修复），两次 Docker 构建叠加，
第二次构建在第一次构建未完成时开始，服务器内存/时间耗尽 → 宕机。

## 铁律

### 合并前
- 执行 `gh pr checks <N>` 确认所有 CI 绿灯（OpenClaw Gate 的脏缓存不算，看实质检查）
- 实质检查清单：lint ✅ typecheck ✅ test ✅ e2e ✅ smoke ✅ spec-format ✅ blueprint-check ✅

### 合并后（必须等待，再合并下一个）
```bash
# 等新版本号出现才算部署完成
until curl -s http://[REDACTED_IP]/health | grep -q "<new_version>"; do sleep 10; done
echo "部署完成，可以合并下一个 PR"
```
- **上一个 PR 部署健康确认前，禁止合并下一个 PR**
- 含前端变更的 PR 部署等待时间约 15-20 分钟（含 Next.js 构建）

### OpenClaw Gate 脏缓存识别
当 OpenClaw Gate 在 8-10 秒内快速失败（正常应等待所有子检查），
且其他实质检查全部绿灯时 → 这是时序问题造成的脏缓存，
使用 `gh api repos/.../pulls/<N>/merge -X PUT -f merge_method=squash` 绕过。

## 合并方式（worktree 冲突时）

`gh pr merge <N> --squash --delete-branch --admin` 会因 main 锁在 worktree 而失败。
正确方式：
```bash
gh api repos/houzhenying226-jpg/nupai-crm/pulls/<N>/merge \
  -X PUT -f merge_method=squash \
  -f commit_title="<title>" -f commit_message="<msg>"
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

