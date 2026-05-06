# nupai-oc-router部署后9步真账验证清单

> 类型：操作规程 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:00Z

## 摘要

任何 nupai-oc-router 配置/代码改动后必须按此 9 步顺序跑完才算真闭环。Step 1-7 是只读验证可放心跑；Step 8/9 是真派工/真审批，执行前确认副作用。"已生效"必须配 9 步真账，少一步即假账。

## 正文

> 记录日期：2026-05-05  风险等级：L4

## 背景
OpenClaw 插件部署后"看起来过了"不等于真闭环。配置加载成功 ≠ gateway 真重启 ≠ 工具真注册 ≠ LLM 真可见 ≠ 业务真派工。需要 9 步真账验证才算闭环（对应"6 天 50 小时假完结"铁律的具体落地）。

## 内容
9 步真账验证清单（按顺序执行，前序失败立即停下，不进下一步）：Step 1 — 配置合法性：```PLAIN_TEXT
openclaw config validate
```预期：exit 0，无 schema 错误。Step 2 — gateway 真重启：```PLAIN_TEXT
/bin/zsh -lc "launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway"
```预期：launchctl 不报错。Step 3 — gateway 真存活 \+ health：```PLAIN_TEXT
lsof -nP -iTCP:18789 -sTCP:LISTEN && curl -sS -i http://127.0.0.1:18789/health
```预期：18789 LISTEN \+ curl 返回 200。Step 4 — 插件 runtime 真注册：```PLAIN_TEXT
openclaw plugins inspect nupai-oc-router --runtime --json | python3 -m json.tool
```预期：3 个工具（dispatch_worker / query_task_status / confirm_task_plan）\+ /oc command alias 真出现。Step 5 — 只读工具真可调（query_task_status）：```PLAIN_TEXT
openclaw agent --message "请调用 query_task_status 查一下当前任务" --session-id smoke-tool-test --json --timeout 90
```预期：返回任务列表，不报"no registered tools matched"。Step 6 — 自然语言进度汇报真可路由：```PLAIN_TEXT
openclaw agent --message "汇报一下每个worker的工作进度" --session-id smoke-status-nl --json --timeout 90
```预期：GPT-5.5 选择 query_task_status 工具，返回任务汇总。Step 7 — 心跳 schema 真复核：```PLAIN_TEXT
openclaw config schema | python3 -c 'import sys,json; s=json.load(sys.stdin); print(json.dumps(s["properties"]["channels"]["properties"]["feishu"]["properties"]["heartbeat"], indent=2, ensure_ascii=False))'
```预期：只看到 visibility \+ intervalMs 两个字段。Step 8（危险，会真派 worker）— 真实派工验证：```PLAIN_TEXT
openclaw agent --message "派 worker 修 STO-186" --session-id smoke-dispatch-worker --json --timeout 90
```预期：返回 cmd_id \+ log_path，飞书战报真到群。⚠️ 这条会真实派 worker，执行前确认你要派。Step 9（危险，会真发飞书审批卡）— 部署审批验证：```PLAIN_TEXT
curl -sS -X POST http://127.0.0.1:18790/command -H 'Content-Type: application/json' --data-raw '{"text":"部署 CRM 到 prod","open_id":"ou_177cf294199865ce1ec0888693cedb05","chat_id":"ou_177cf294199865ce1ec0888693cedb05","event_id":"smoke-deploy-crm-prod"}' | python3 -m json.tool
```预期：返回审批卡片消息，⚠️ 飞书会真发卡，但不会直接部署 prod。确认计划验证（条件性，仅当有 pending plan 时）：```PLAIN_TEXT
echo '{"text":"确认","open_id":"ou_177cf294199865ce1ec0888693cedb05","chat_id":"ou_177cf294199865ce1ec0888693cedb05","event_id":"test-confirm"}' | curl -sS -X POST http://127.0.0.1:18790/command -H 'Content-Type: application/json' -d @- | python3 -m json.tool
```⚠️ 当前有 pending plan 时这条会真实确认派发，执行前确认就是要派。关键端口与 ID：18789 — gateway18790 — dispatcherchat_id ou_177cf294199865ce1ec0888693cedb05 — 测试对话标识

## 结论
任何 nupai-oc-router 配置/代码改动后必须按此 9 步顺序跑完才算真闭环。Step 1-7 是只读验证可放心跑；Step 8/9 是真派工/真审批，执行前确认副作用。"已生效"必须配 9 步真账，少一步即假账。

## 影响
所有 nupai-oc-router 部署后验证流程；未来其他 OpenClaw 插件参考此模式建立各自验证清单；CC 提交"已部署"PR 时审核标准；避免"6 天 50 小时假完结"重演。

## 下次注意
9 步顺序不能跳——前序失败立即停；Step 1 不过不要 kickstart；Step 4 工具不真注册不要 smoke 测；Step 5/6 报"no registered tools matched"立即查 5 处 policy 配置；Step 8/9 是写操作执行前看清 chat_id / issue_id 是不是测试值；OpenClaw 升级版本后这 9 步都要重跑一遍。

