---
type: runbooks
title: dispatcher P2P DM allowlist 配置规则
date: 2026-05-02
risk_level: L4
project: openclaw
tags:
  - dispatcher
  - feishu
  - allowlist
---

# dispatcher P2P DM allowlist 配置规则

## 背景

2026-05-02 Phase 5 真测翻车。振英在飞书直接 DM OpenClaw 机器人发 `/oc 修 NUP-321` 被 Layer 2 拦截：

```text
[OpenClaw] 命令被拒绝: chat_id 'ou_177cf...' not in allowlist (group chats blocked)
```

根因：飞书 P2P DM 的 `chat_id` = 发送方的 `open_id`，不是 group chat ID。

## 飞书 chat_id 真规则

```text
群聊 chat_id: oc_xxxxx
P2P DM chat_id: ou_xxxxx = 发送方 open_id
```

当前白名单值（2026-05-02）：

```text
oc_e401a77d9e684c23e2effe35ee34b6b1  # command-bot 群聊
ou_177cf294199865ce1ec0888693cedb05  # 振英 P2P DM
```

## 修复规则

PR #429（NUP-372）：

```text
旧：ALLOWED_CHAT_ID: str = "oc_e401a77d9e684c23e2effe35ee34b6b1"
新：ALLOWED_CHAT_IDS: frozenset[str] = _load_allowed_chat_ids()
```

`_load_allowed_chat_ids()`：硬编码群聊保底 + `~/.openclaw/openclaw.json` 的 `channels.feishu.allowFrom[]`。

Layer 2 校验：

```python
if chat_id not in ALLOWED_CHAT_IDS:
    reject
```

## 新增授权用户 SOP

1. 编辑 `~/.openclaw/openclaw.json`。
2. 在 `channels.feishu.allowFrom[]` 添加 open_id。
3. 重启 dispatcher：

```bash
launchctl kickstart -k gui/$(id -u)/com.nupai.dispatcher
```

4. curl 真测：

```bash
curl -X POST http://127.0.0.1:18790/command \
  -d '{"text":"/oc 状态","open_id":"<新 open_id>","chat_id":"<新 open_id>"}'
```

真返回 `verb=status` 才算生效。

## 下次注意

飞书 P2P DM 不暴露原生 group chat_id，直接用对方 open_id 当 chat_id。
allowlist 必须包含所有授权用户的 open_id，不能只放 group chat_id。
