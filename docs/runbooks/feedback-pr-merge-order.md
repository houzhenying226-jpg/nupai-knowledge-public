# PR合并顺序与部署健康验证铁律

> 类型：操作规程 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-11T10:30:00Z

## 摘要

连续合并 PR 前必须确认 CI 绿灯和上一个部署健康完成，避免 Docker 构建叠加导致宕机。

## 正文

> 记录日期：2026-05-11  风险等级：L4

## 背景
2026-05-11 真实事故：5 分钟内连续合并 PR #545（NUP-425）和 PR #544（E6修复），两次 Docker 构建叠加，第二次构建在第一次未完成时开始，服务器内存/时间耗尽导致宕机。

## 内容
合并前执行 gh pr checks <N>，确认 lint/typecheck/test/e2e/smoke/spec-format/blueprint-check 等实质检查绿灯。合并后必须等待健康检查返回新版本号才允许合并下一个 PR。含前端变更的 PR 部署等待约 15-20 分钟。OpenClaw Gate 若 8-10 秒快速失败而实质检查全绿，可能是时序脏缓存，可用 GitHub API squash merge 绕过。

## 结论
上一个 PR 部署健康确认前，禁止合并下一个 PR。worktree 冲突时使用 gh api repos/.../pulls/<N>/merge -X PUT -f merge_method=squash，而不是 gh pr merge --delete-branch。

## 影响
避免部署流水线重叠、服务器资源耗尽、生产 504。保证每次合并都能独立定位问题。

## 下次注意
每次 merge 前看实质 CI；每次 merge 后等 /health 新版本；遇到 OpenClaw Gate 脏缓存只在实质检查全绿时绕过。

