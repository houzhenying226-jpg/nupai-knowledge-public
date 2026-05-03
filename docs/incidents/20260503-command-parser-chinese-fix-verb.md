---
type: incidents
title: command_parser 中文 verb 修 缺失踩坑
date: 2026-05-03
risk_level: L3
project: openclaw
tags:
  - dispatcher
  - command-parser
  - feishu
---

# command_parser 中文 verb 修 缺失踩坑

## 背景

2026-05-03 09:30 PR #431 真合 main 后，振英真发 `/oc 修 NUP-321`，飞书机器人返回 `[OpenClaw 状态]`，不是 worker 启动。

Python 模拟：

```python
parse('/oc 修 NUP-321')
```

真返回 `verb='UNKNOWN'`。

## 真根因

`dispatcher/command_parser.py` 的 `ALLOWED_VERBS` 缺中文 `修`：

```python
ALLOWED_VERBS = frozenset({"fix", "status", "状态", "进度", "list", "列表", "audit", "deploy"})
```

导致 `/oc 修` 解析为 `verb=UNKNOWN`，fallthrough 到 status 默认处理。

真补债 §8.1 已警告此坑：中文 verb `修` 不在英文白名单 → fall-through brain。

## 修复

PR #440：`fix(dispatcher): NUP-376 /oc 修 alias`

- `ALLOWED_VERBS` 加 `修`
- 路由判断从 `verb == "fix"` 改成 `verb in ("fix", "修")`
- 加回归测试：
  - `/oc 修 NUP-321` → `verb='fix' issue='NUP-321'`
  - `/oc 修 STO-176` → `verb='fix' issue='STO-176'`

## 合并后真验证

```bash
git -C ~/nupai-crm-main fetch origin
git -C ~/nupai-crm-main reset --hard origin/main
launchctl kickstart -k gui/$(id -u)/com.nupai.dispatcher
sleep 8
python3 -c "import sys; sys.path.insert(0,'/Users/james/nupai-crm-main'); from dispatcher.command_parser import parse; print(parse('/oc 修 NUP-321',open_id='ou_177cf...',chat_id='ou_177cf...'))"
```

期望：

```text
verb='fix' issue='NUP-321' project='crm'
```

## 结论

中文/英文 verb 必须双向真覆盖。

当前 `ALLOWED_VERBS` 真规则：

```python
{"fix", "修", "status", "状态", "进度", "list", "列表", "audit", "deploy"}
```

## 下次注意

新增 verb 真规则 4 步：

1. `ALLOWED_VERBS` 加新 verb 别名，中英文都加。
2. `grep -rn 'verb == "<英文>"' dispatcher/` 全部改成 `verb in ("<英文>", "<中文>")`。
3. tests 加双语测试用例。
4. 真发飞书命令验 `verb=fix`，不是 `verb=UNKNOWN`。
