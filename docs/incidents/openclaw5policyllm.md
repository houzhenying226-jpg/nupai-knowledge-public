# OpenClaw工具5处policy配置才能LLM可见

> 类型：故障复盘 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:00Z

## 摘要

新增 OpenClaw 插件工具的 SOP：① 插件代码 registerTool ② 插件清单 contracts.tools ③ 主 agent profile \+ alsoAllow ④ sandbox.tools.alsoAllow ⑤ 根级 tools.allow。5 处任一遗漏 → LLM 看不见工具 → 报"no registered tools matched"。

## 正文

> 记录日期：2026-05-05  风险等级：L4

## 背景
nupai-oc-router 通过 api.registerTool 注册了 dispatch_worker / query_task_status / confirm_task_plan 三个工具，plugin runtime 显示注册成功，但 GPT-5.5 在飞书对话中报"没有工具/不能真派 worker"，gateway 日志显示 no registered tools matched。

## 内容
工具注册成功 ≠ LLM 可调用。OpenClaw 有多层工具策略过滤，只配一层时插件工具仍会被过滤掉。5 处必须同步配置（缺一即工具对 LLM 不可见）：插件代码 api.registerTool\('tool_name', \{...\}\) — 注册到 plugin runtime插件清单 openclaw.plugin.json 的 contracts.tools 数组 — 声明插件提供哪些工具主 agent agents.list[0].tools.profile 设为 "coding" — agent 第一层 policy主 agent agents.list[0].tools.alsoAllow 数组追加工具名 — agent 白名单扩展sandbox agents.list[0].tools.sandbox.tools.alsoAllow 数组追加工具名 — sandbox 子层 policy（agents.defaults.sandbox.mode = "all" 触发）根级 tools.allow 也要包含这三个工具名，保证全局 allowlist 不再排斥。5 处具体配置（来自实际生效的 openclaw.json）：```PLAIN_TEXT
"tools.allow": ["dispatch_worker", "query_task_status", "confirm_task_plan"]

"agents.list[0].tools": {
  "profile": "coding",
  "alsoAllow": ["dispatch_worker", "query_task_status", "confirm_task_plan"],
  "sandbox": {
    "tools": {
      "alsoAllow": ["dispatch_worker", "query_task_status", "confirm_task_plan"]
    }
  }
}

"openclaw.plugin.json": {
  "contracts.tools": ["dispatch_worker", "query_task_status", "confirm_task_plan", "memory_recall"]
}
```排查命令：```PLAIN_TEXT
openclaw plugins inspect nupai-oc-router --runtime --json | python3 -m json.tool
```如果 runtime 显示工具但 LLM 调不到，依次核 5 处配置。

## 结论
新增 OpenClaw 插件工具的 SOP：① 插件代码 registerTool ② 插件清单 contracts.tools ③ 主 agent profile \+ alsoAllow ④ sandbox.tools.alsoAllow ⑤ 根级 tools.allow。5 处任一遗漏 → LLM 看不见工具 → 报"no registered tools matched"。

## 影响
所有 nupai-oc-router 工具注册流程；未来任何 OpenClaw 插件开发；agents.list 多 agent 配置时每个 agent 都要单独同步 alsoAllow；sandbox.mode = "all" 必须配套 sandbox.tools.alsoAllow；根级 tools.allow 是全局白名单不能漏。

## 下次注意
新增工具时拿这 5 处当 checklist 逐项核；报"no registered tools matched"立即查这 5 处；plugin inspect runtime 显示工具不代表 LLM 可见，要看 policy 链路完整；记住 sandbox 是独立 scope 不会自动继承上层 policy。

