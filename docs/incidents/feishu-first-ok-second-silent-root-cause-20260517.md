# 飞书"第一条有反应第二条静默"根因仍未定论——勿删 channels.feishu

> 类型：故障复盘 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-17T08:41:52Z

## 摘要

channels.feishu 是 Feishu 连接凭证，删除它会立即断联。"第一条有反应第二条静默"的真实根因未查清，不是
  channels.feishu 字段本身。

  ---

## 正文

> 记录日期：2026-05-17  风险等级：L1

## 背景
2026-05-17 排查飞书静默问题时，Claude 误判 channels.feishu 字段导致 gateway 崩溃循环，将其从 openclaw.json
  删除，结果 gateway 无法连接飞书，飞书完全死亡约 15 分钟。

## 内容
channels.feishu 包含的字段：enabled / appId / appSecret / connectionMode / domain / groupPolicy / dmPolicy /
  allowFrom 等。这是 gateway 连接飞书的完整凭证，绝对不可删除。

  "第一条有反应第二条静默"的已排除原因：
  - channels.feishu 字段本身（删掉只会更糟，不是根因）

  "第一条有反应第二条静默"的可能原因（待验证）：
  - gateway agent session 上下文在第一条后进入等待/锁定状态
  - 飞书 WS 连接不稳定导致消息偶发丢失
  - gateway 内存或 token 限制触发的静默降级

  正确排查命令：
  tail -f ~/.openclaw/logs/gateway.log | grep -E "received|dispatch complete|error|crash|restart"
  观察 dispatch complete replies=0 出现的场景，结合前后日志找真正根因。

## 结论
channels.feishu 是 Feishu 凭证，绝对不能删。修改 openclaw.json 前必须先确认字段含义，有 appId/appSecret
  的字段是凭证不能动。

## 影响
误删导致飞书完全失联约 15 分钟，所有飞书消息处理中断。

## 下次注意
修改 openclaw.json 之前，先执行 python3 -c "import json;
  print(list(json.load(open('/Users/james/.openclaw/openclaw.json'))['channels']['feishu'].keys()))" 确认字段含义。看到
  appId/appSecret 立刻停手。

