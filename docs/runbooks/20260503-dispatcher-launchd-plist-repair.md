---
type: runbooks
title: dispatcher launchd plist 真修复全流程
date: 2026-05-03
risk_level: L4
project: openclaw
tags:
  - dispatcher
  - launchd
  - repair
---

# dispatcher launchd plist 真修复全流程

## 背景

2026-05-02 23:00 → 5-3 00:30 真账翻车。dispatcher 反复 SIGTERM 8 小时无 worker 真启动。

三方 AI 交叉验证定性：launchd 入口 plist `ProgramArguments` 仍指向老路径 `dispatcher.sh` wrapper，三个时代 service 同时跑，孤儿进程真占 18790，launchd 每 60 秒拉新 `webhook_server` 撞端口崩。

## 真修复 8 步

1. `cat dispatcher.sh` 取证：看 wrapper 内部真启什么 + 有无 venv/env 注入。
2. 备份 plist：

```bash
cp /Users/james/Library/LaunchAgents/com.nupai.dispatcher.plist \
  /Users/james/Library/LaunchAgents/com.nupai.dispatcher.plist.bak.$(date +%Y%m%d-%H%M%S)
```

3. bootout 旧 job 停 launchd 自动拉起：

```bash
launchctl bootout gui/$(id -u)/com.nupai.dispatcher
launchctl bootout gui/$(id -u)/com.nupai.k2-webhook
launchctl bootout gui/$(id -u)/com.nupai.webhook
sleep 2
```

4. kill 三时代孤儿：

```bash
kill -KILL <PID...>
sleep 2
ps aux | grep webhook_server | grep -v grep
```

期望 0 行。

5. 验证 18790 真空：

```bash
lsof -nP -iTCP:18790 -sTCP:LISTEN
```

期望无输出。

6. 写新 plist，`ProgramArguments` 直启 python，绕过 wrapper：

```text
/opt/homebrew/bin/python3 -u /Users/james/nupai-crm-main/dispatcher/webhook_server.py
WorkingDirectory=/Users/james/nupai-crm-main
PYTHONPATH=/Users/james/nupai-crm-main
NUPAI_REPO_ROOT=/Users/james/nupai-crm-main
PYTHONUNBUFFERED=1
KeepAlive=true
```

验证：

```bash
plutil -lint <plist>
```

7. bootstrap + kickstart：

```bash
launchctl bootstrap gui/$(id -u) <plist>
launchctl kickstart -k gui/$(id -u)/com.nupai.dispatcher
sleep 10
```

8. 6 项铁验闭环：

- `launchd_pid == port_pid`
- command 含 `webhook_server.py` + `nupai-crm-main`
- cwd = `/Users/james/nupai-crm-main`
- `/health` 真返回 `{"status":"ok"}`
- 70 秒后 port_pid 不变
- stderr.log 无新 `OSError Address already in use`

## 结论

launchctl bootout 旧 job → kill 孤儿 → 验 18790 空 → 写新 plist 直启 python → bootstrap + kickstart → 6 项铁验。

70 秒铁验仅算“上线成功”，8 小时无 SIGTERM 才算“可托管”。
