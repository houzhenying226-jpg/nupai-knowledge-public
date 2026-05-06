# nupai-oc-router部署后9步真账验证清单

> 类型：操作规程 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

nupai-oc-router 配置或代码改动后必须按 9 步真账验证，从配置合法、gateway 存活到工具可见、自然语言路由和危险写操作确认。

## 正文

> 记录日期：2026-05-05  风险等级：L4

## 背景
OpenClaw 插件部署后“看起来过了”不等于真闭环。配置加载成功不等于 gateway 真重启，不等于工具真注册，不等于 LLM 真可见，不等于业务真派工。需要 9 步真账验证才算闭环。

## 内容
9 步真账验证清单：
1. 配置合法性：openclaw config validate，预期 exit 0。
2. gateway 真重启：launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway，预期不报错。
3. gateway 真存活 + health：lsof 18789 LISTEN 且 curl /health 返回 200。
4. 插件 runtime 真注册：openclaw plugins inspect nupai-oc-router --runtime --json，预期 3 个工具和 /oc command alias 出现。
5. 只读工具真可调：openclaw agent 调用 query_task_status，预期返回任务列表，不报 no registered tools matched。
6. 自然语言进度汇报真可路由：openclaw agent 消息“汇报一下每个worker的工作进度”，预期 GPT-5.5 选择 query_task_status 工具。
7. 心跳 schema 真复核：openclaw config schema 查看 channels.feishu.heartbeat，预期只看到 visibility + intervalMs。
8. 真实派工验证：openclaw agent 消息“派 worker 修 STO-186”，会真实派 worker，执行前确认。
9. 部署审批验证：POST dispatcher /command 部署 CRM 到 prod，会真发飞书审批卡但不会直接部署 prod。

确认计划验证只在有 pending plan 时执行，且会真实确认派发，执行前必须确认。

关键端口与 ID：18789 是 gateway，18790 是 dispatcher，chat_id ou_177cf294199865ce1ec0888693cedb05 是测试对话标识。

## 结论
任何 nupai-oc-router 配置或代码改动后必须按 9 步顺序跑完才算真闭环。Step 1-7 是只读验证；Step 8/9 是真派工/真审批，执行前确认副作用。“已生效”必须配 9 步真账，少一步即假账。

## 影响
影响所有 nupai-oc-router 部署后验证流程；未来其他 OpenClaw 插件可参考此模式建立验证清单；CC 提交“已部署”PR 时按此审核；避免“6 天 50 小时假完结”重演。

## 下次注意
9 步顺序不能跳，前序失败立即停；Step 1 不过不要 kickstart；Step 4 工具不真注册不要 smoke 测；Step 5/6 报 no registered tools matched 立即查 5 处 policy 配置；Step 8/9 是写操作，执行前确认 chat_id / issue_id 是不是测试值；OpenClaw 升级后 9 步都要重跑。

