# channels.feishu.heartbeat仅含visibility与intervalMs

> 类型：故障复盘 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:00Z

## 摘要

心跳配置分两层：channels.feishu.heartbeat 只管"可见性 \+ 间隔"两个简单字段；agents.defaults.heartbeat 管心跳行为（prompt / 报错处理 / 上下文 / 超时等）。报错信息控制在 agent 层，不在 channel 层。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
nupai-oc-router 部署时尝试在 channels.feishu.heartbeat 下配置 showOk 字段，期望控制 OK 状态显示。openclaw config validate 报错 channels.feishu.heartbeat: invalid config: must NOT have additional properties。

## 内容
OpenClaw 版本 2026.5.3-1 的 schema 中 channels.feishu.heartbeat 只有两个合法字段：   字段 类型 含义    visibility "visible" / "hidden" 心跳可见性  intervalMs 整数（毫秒） 心跳间隔不存在 showOk / showError / template 等字段。任何额外字段会被 schema 拒绝。正确配置（来自实际生效的 openclaw.json）：```PLAIN_TEXT
"channels.feishu": {
  "streaming": true,
  "heartbeat": {
    "visibility": "visible",
    "intervalMs": 60000
  }
}
```如果想要"只在出错时显示"或"OK 时静默"等行为，不能在 channels.feishu.heartbeat 实现，要去 agents.defaults.heartbeat 层配置（那一层有 prompt / suppressToolErrorWarnings 等丰富字段）。schema 复核命令：```PLAIN_TEXT
openclaw config schema | python3 -c 'import sys,json; s=json.load(sys.stdin); print(json.dumps(s["properties"]["channels"]["properties"]["feishu"]["properties"]["heartbeat"], indent=2, ensure_ascii=False))'
```agent heartbeat schema 复核命令：```PLAIN_TEXT
openclaw config schema | python3 -c 'import sys,json; s=json.load(sys.stdin); print(json.dumps(s["properties"]["agents"]["properties"]["defaults"]["properties"]["heartbeat"], indent=2, ensure_ascii=False))'
```agents.defaults.heartbeat 实际生效配置（5 分钟主动汇报，报错不静默，忙碌也不跳过）：```PLAIN_TEXT
"agents.defaults.heartbeat": {
  "every": "5m",
  "target": "last",
  "directPolicy": "allow",
  "includeReasoning": true,
  "includeSystemPromptSection": true,
  "ackMaxChars": 1200,
  "suppressToolErrorWarnings": false,
  "timeoutSeconds": 180,
  "isolatedSession": false,
  "lightContext": false,
  "skipWhenBusy": false,
  "prompt": "读 HEARTBEAT.md 并严格按其执行..."
}
```

## 结论
心跳配置分两层：channels.feishu.heartbeat 只管"可见性 \+ 间隔"两个简单字段；agents.defaults.heartbeat 管心跳行为（prompt / 报错处理 / 上下文 / 超时等）。报错信息控制在 agent 层，不在 channel 层。

## 影响
所有飞书心跳行为配置；未来 OpenClaw 升级时 schema 字段可能新增（要重跑 schema 复核命令）；agents.defaults.heartbeat 是真正的行为控制层。

## 下次注意
配置任何 OpenClaw 字段前先用 schema 命令查合法字段再写；报 additional properties 立即查 schema 不要硬试；心跳功能需求分清是"显示"问题（channel 层）还是"行为"问题（agent 层）；OpenClaw 版本号要记下，schema 可能随版本变。

