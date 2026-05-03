---
type: incidents
title: OpenClaw 7天首次真账闭环 STO-202
date: 2026-05-03
risk_level: L4
project: openclaw
tags:
  - openclaw
  - true-ledger
  - worker
  - sto-202
---

# OpenClaw 7天首次真账闭环 STO-202

## 背景

OpenClaw 4-26 上线到 5-3，7 天 50+ 小时，worker 真自动出 PR commit author=非 Claude 真账=0 次。
30+ PR 全是 Claude Code 在终端手工 `gh pr create` 开的。

5-3 上午振英在飞书 DM OpenClaw 发 `/oc 修 STO-202` 真测，触发首次真账闭环。

## PR #87 真原文证据

```text
repo: houzhenying226-jpg/-nupai-store
state: OPEN
headRefName: openclaw/STO-202-e39c23b7
createdAt: 2026-05-03T11:05:34Z
author: houzhenying226-jpg
commits[0]: oid=e486ef4, authors=["OpenClaw"], message="fix: STO-202 (OpenClaw auto-commit, task=e39c23b7-01c5-4dcf-b6b1-9c03…"
commits[1]: oid=3e8e842, authors=["James", "Claude Sonnet 4.6"], message="fix(docker): STO-202 也替换 security.debian.org 为阿里云镜像"
```

STO-202 真账内容：Dockerfile apt 源改用阿里云镜像（`deb.debian.org` → `mirrors.aliyun.com`）。

## 真账时刻达到的 4 个标志

1. `commits[0].authors[0] = "OpenClaw"`，不是 `Claude Code` / `Claude Sonnet 4.6`。
2. `headRefName` 含 `openclaw/` 前缀，证明走 worker.sh 5 步管道。
3. commit message 含 `OpenClaw auto-commit, task=<uuid>`。
4. `createdAt` 在飞书发命令后 5-15 分钟内。

## 结论

OpenClaw 真账闭环判定标准（Phase 5）：

- commit author = OpenClaw / Codex / GPT-5.5（非 Claude Code / 非 Claude Sonnet）
- headRefName 以 `openclaw/` 开头
- commit message 含 `OpenClaw auto-commit, task=<uuid>`

任一不符 = 假闭环，继续在 Claude Code 手开 PR。

## 影响

- 验收 OpenClaw 真闭环唯一权威标准就是 GitHub PR commit author。
- 飞书战报 / dispatcher `/health` 200 / curl 测试 / cmd_id 回执都不算真闭环证据。
- 真补债文档 v1/v2/v3 三份完结报告全部夸大，不能引用为真账证据。

## 下次注意

任何“OpenClaw 真闭环 / 真上线 / 真完结”主张前必查：

```bash
gh pr view <号> --json commits
```

看 `authors` 字段。author 含 Claude 字眼 = 假闭环，无论飞书战报多漂亮。
