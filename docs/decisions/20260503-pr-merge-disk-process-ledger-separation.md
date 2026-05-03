---
type: decisions
title: PR合并vs磁盘vs进程三层真账分离
date: 2026-05-03
risk_level: L4
project: 全局
tags:
  - deployment
  - true-ledger
  - process
---

# PR合并vs磁盘vs进程三层真账分离

## 背景

2026-05-02 23:00 → 2026-05-03 00:30 真账翻车：PR #431 NUP-372 P2P DM allowlist 真合 main，但振英真发 `/oc 修 NUP-321` 仍然静默 REJECT。

三方 AI 深读 GitHub 共识：cloud 上代码是对的，但本地磁盘 PID 38124 一直跑从未更新的老版本 `ALLOWED_CHAT_ID` 单值。

## 三层真账

### 第 1 层：GitHub cloud

`gh pr view <N> --state merged` 只代表 cloud，不代表磁盘。
merge commit SHA 在 `origin/main`，本地仓库可能没拉。

### 第 2 层：磁盘工作树

`git status: clean` 不等于 up to date。clean 只说没未提交改动。
真账要看：

```bash
git fetch origin
git reset --hard origin/main
git log --oneline -3
```

### 第 3 层：进程内存

Python import 模块级常量不会热重载。
改 `.py` 文件后必须杀进程重启：

```bash
launchctl kickstart -k gui/$(id -u)/<service>
```

重启后必须验证：进程启动时间 ≥ 文件 mtime ≥ git pull 时间。

## 强制 3 道闸门

### 闸门 1：磁盘真 pull

```bash
PR_SHA=$(gh pr view <N> --json mergeCommit -q .mergeCommit.oid)
git -C <repo> fetch origin
git -C <repo> reset --hard origin/main
[ "$PR_SHA" = "$(git -C <repo> rev-parse HEAD)" ] || STOP
```

### 闸门 2：进程真重启

```bash
launchctl kickstart -k gui/$(id -u)/<service>
sleep 8
NEW_PID=$(lsof -nP -iTCP:<port> -sTCP:LISTEN -t | head -1)
ps -o lstart= -p $NEW_PID
```

### 闸门 3：真规则 curl 真测

用符合新规则的 payload curl 真测，真返回符合新规则才算过。
禁止让用户真发当真测。

## 4 真账核 4 件套

任何“已生效”主张必须配：

```bash
ps aux | grep <进程关键词>
lsof -nP -iTCP:<端口>
launchctl print gui/$(id -u)/<service>
tail -100 <stderr.log>
```

缺一 → 不许真测，不许下一步。
