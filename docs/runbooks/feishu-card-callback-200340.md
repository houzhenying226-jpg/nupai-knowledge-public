# 飞书卡片按钮点击 200340 审批无响应无效操作

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T03:40:02Z

## 摘要

飞书部署审批卡片点击后出现 200340、无效操作或审批无响应时的诊断、根因与修复 runbook。

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
飞书部署审批卡片点击后可能出现 200340、无效操作、已处理但未触发 GitHub Actions 等问题。

## 内容
# Runbook：飞书卡片按钮点击 — 200340 / 审批无响应 / 无效操作

## 现象

- 用户点击飞书审批卡片「批准部署」后，出现以下任一情况：
  - 「出错了，请稍后重试 code: 200340」
  - 「无效的操作」toast
  - 「此审批已处理」但 GitHub Actions 未触发
- pending-deploys.json 中该 cmd_id 的 status 一直是 `pending`
- webhook server 日志无对应的 `/feishu-callback` 请求记录

## 诊断

```bash
# 1. 确认 webhook server 进程存在
ps aux | grep webhook_server | grep -v grep

# 2. 确认本地端口 18790 可达
curl -s http://localhost:18790/health

# 3. 确认公网链路可达（从本机模拟 Lark challenge）
curl -i -X POST https://openclaw.hzy1979.com/feishu-callback \
  -H "Content-Type: application/json" \
  -d '{"challenge":"test","token":"x","type":"url_verification"}'
# 期望：HTTP 200，Content-Type: application/json，body = {"challenge":"test"}
# 如果返回 502 → webhook server 未运行（方案B）
# 如果返回 HTML → Cloudflare 拦截（方案D）

# 4. 查 auth proxy 访问日志，确认 Lark 回调是否到达
tail -50 ~/Library/Logs/com.openclaw.auth-proxy.out.log | grep feishu-callback
# 有 "proxy-feishu-callback-bypass" 条目 → 回调到达，看 stderr 查解析错误
# 无条目 → Lark 未配置回调 URL（方案A）

# 5. 查 webhook server stderr，确认回调 payload 格式
tail -30 ~/.openclaw/k2-webhook.stderr.log | grep -E "feishu_card|feishu_callback|payload"

# 6. 查 cmd_id 在 pending-deploys.json 的状态
python3 -c "
import json; d=json.load(open('/Users/james/.openclaw/feishu/pending-deploys.json'))
for k,v in sorted(d.items(), key=lambda x: x[1].get('created_at',''))[-5:]:
    print(k[:8], v.get('status'), v.get('plan'), v.get('created_at'))
"
```

## 已知根因

| 根因 | 现象 | 诊断特征 | 修复方案 |
|------|------|---------|---------|
| **Lark 开发者后台未配置卡片回调 URL** | 200340 立刻弹出 | auth proxy 日志无任何 Lark 来源的 `/feishu-callback` 请求 | 方案A |
| **Card Kit 2.0 回调 payload 格式未适配** | 「无效的操作」toast | stderr 有 `payload keys=['schema','header','event']`；`action_val={}` | 方案E |
| **webhook server 未运行** | 公网返回 502 | `curl localhost:18790/health` 无响应 | 方案B |
| **launchd plist disabled，kill 后不自动重启** | server 宕机后无法自愈 | `ls ~/Library/LaunchAgents/ \| grep k2-webhook` 显示 `.disabled` | 方案B |
| **deploy.yml 缺少 workflow_dispatch 触发器** | 批准成功但 GitHub 返回 HTTP 422 | stderr 有 "Workflow does not have 'workflow_dispatch' trigger" | 方案C |
| **Cloudflare 拦截** | 公网返回 HTML，Lark challenge 失败 | `curl -i` 返回 HTML 而非 JSON | 方案D |

## 修复步骤

### 方案A：配置 Lark 卡片回调 URL（新项目必做的一次性配置）

> **前提**：执行前确认 webhook server 在线（`curl -s http://localhost:18790/health` 返回 ok）

1. 进入 [Lark 开发者后台](https://open.larksuite.com/app) → 找到 OpenClaw bot app
2. 左侧菜单 →「事件与回调」→「回调配置」
3. 「请求地址（订阅方式：将回调发送至开发者服务器）」填：
   ```
   https://openclaw.hzy1979.com/feishu-callback
   ```
4. 点「保存」— Lark 发 challenge，server 实时响应后保存成功
5. 点「添加回调」→ 选「**卡片回传交互**」（Card Kit 2.0，非旧版）
6. 发布新版本

> 如果保存时报「返回数据不是合法的JSON格式」，说明 server 不在线，先执行方案B。

### 方案B：重启 webhook server

```bash
# 检查 launchd plist 状态（plist 是 .disabled 则不会自动重启）
ls ~/Library/LaunchAgents/ | grep k2-webhook

# 手动启动（env 变量来自 plist backup）
PYTHONPATH=/Users/james/nupai-crm-main/dispatcher \
NUPAI_REPO_ROOT=/Users/james/nupai-crm-main \
PYTHONUNBUFFERED=1 \
HOME=/Users/james \
/opt/homebrew/bin/python3 /Users/james/nupai-crm-main/dispatcher/webhook_server.py \
  >> ~/.openclaw/k2-webhook.stdout.log \
  2>> ~/.openclaw/k2-webhook.stderr.log &

sleep 3 && curl -s http://localhost:18790/health
```

> **警告**：plist 是 `.disabled` 时，`kill` 掉 server 后不会自动重启。不要随意 kill，需要重启时用上面命令重新拉起。

### 方案C：deploy.yml 补充 workflow_dispatch 触发器

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:          # OpenClaw 通过 gh api dispatches 调用，必须有此触发器
    inputs:
      environment:
        required: false
        default: "prod"
      cmd_id:
        required: false
        default: ""

jobs:
  deploy:
    if: github.ref == 'refs/heads/main'   # branch guard，防止非 main 分支被 dispatch
    steps:
      - name: Log dispatch info
        if: github.event_name == 'workflow_dispatch'
        run: echo "env=${{ inputs.environment }} cmd_id=${{ inputs.cmd_id }}"
```

store 已通过 PR #153（commit 6117d00）合并。CRM 的 deploy.yml 已有此配置。

### 方案D：Cloudflare 拦截排查

```bash
curl -i -X POST https://openclaw.hzy1979.com/feishu-callback \
  -H "Content-Type: application/json" \
  -d '{"challenge":"test","token":"x","type":"url_verification"}'
```

如果返回 HTML：
- Cloudflare Dashboard → Security → Bot Fight Mode → 关闭或加白名单
- Speed → Optimization → 关闭 Rocket Loader / Auto Minify
- SSL/TLS 模式确认为 Full (strict)

### 方案E：适配 Card Kit 2.0 回调 payload 格式

Lark 订阅「卡片回传交互」后，回调 payload 格式为：

```json
{
  "schema": "2.0",
  "header": {"event_type": "card.action.trigger", "token": "..."},
  "event": {
    "operator": {"open_id": "ou_..."},
    "action": {"value": {"action": "approve", "cmd_id": "..."}}
  }
}
```

旧版 payload（直接读顶层字段）：
```json
{"operator": {"open_id": "..."}, "action": {"value": {...}}, "token": ""}
```

`feishu_card.py` 的 `handle_card_callback` 函数开头必须有适配逻辑：

```python
# Lark Card Kit 2.0 new-style callback wraps everything under payload["event"]
if payload.get("schema") == "2.0" and isinstance(payload.get("event"), dict):
    event = payload["event"]
    payload = {
        "operator": event.get("operator", {}),
        "open_id": (event.get("operator") or {}).get("open_id", ""),
        "action": event.get("action", {}),
        "token": (payload.get("header") or {}).get("token", ""),
    }
```

已修复并重启 server（2026-05-13）。

## 验证成功

```bash
# 1. 公网 challenge 返回 JSON
curl -s -X POST https://openclaw.hzy1979.com/feishu-callback \
  -H "Content-Type: application/json" \
  -d '{"challenge":"verify","token":"x","type":"url_verification"}'
# 期望：{"challenge":"verify"}

# 2. 点击飞书卡片「批准部署」→ 显示"部署已批准，正在触发 GitHub Actions"
# 3. GitHub Actions Deploy workflow 触发（workflow_dispatch）
gh run list --repo houzhenying226-jpg/-nupai-store --limit 3

# 4. pending-deploys.json 中 cmd_id 状态变为 approved
python3 -c "
import json; d=json.load(open('/Users/james/.openclaw/feishu/pending-deploys.json'))
last = sorted(d.items(), key=lambda x: x[1].get('created_at',''))[-1]
print(last[0][:8], last[1].get('status'))
"
```

## 新项目接入 OpenClaw 部署审批 Checklist

- [ ] deploy.yml 包含 `workflow_dispatch` 触发器 + branch guard
- [ ] Lark 开发者后台「事件与回调 → 回调配置」已配置请求地址并通过 challenge
- [ ] 已添加「卡片回传交互」回调订阅并发布版本
- [ ] 点卡片「批准部署」测试全链路（approved → GitHub Actions triggered）

## 事故记录

| 日期 | 持续 | 根因组合 | 教训 |
|------|------|---------|------|
| 2026-05-12 ~ 2026-05-13 | ~20h | ① store deploy.yml 缺 workflow_dispatch → HTTP 422；② Lark 未配置卡片回调 URL → 200340；③ Card Kit 2.0 新 payload 格式未适配 → 无效操作；④ launchd plist disabled → kill 后不自动重启 | 新项目必须走「接入 Checklist」；Card Kit 2.0 的回调格式与旧版不同，必须在 server 端做格式适配 |

## 结论
按 runbook 依次确认 Lark 回调配置、webhook server、Card Kit 2.0 payload 适配、deploy.yml workflow_dispatch 和公网链路。

## 影响
新项目接入部署审批必须完成回调 URL、卡片回传交互订阅、workflow_dispatch 与端到端批准验证。

## 下次注意
不要只看卡片状态；必须同时验 pending-deploys.json、webhook 日志和 GitHub Actions workflow_dispatch。

