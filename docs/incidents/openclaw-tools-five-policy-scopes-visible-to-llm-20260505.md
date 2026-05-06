# OpenClaw工具5处policy配置才能LLM可见

> 类型：故障复盘 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

OpenClaw 插件工具 registerTool 成功不等于 LLM 可调用，必须同步插件代码、插件清单、主 agent policy、sandbox policy 和根级 allowlist。

## 正文

> 记录日期：2026-05-05  风险等级：L4

## 背景
nupai-oc-router 通过 api.registerTool 注册 dispatch_worker、query_task_status、confirm_task_plan 三个工具，plugin runtime 显示注册成功，但 GPT-5.5 在飞书对话中报“没有工具/不能真派 worker”，gateway 日志显示 no registered tools matched。

## 内容
工具注册成功不等于 LLM 可调用。OpenClaw 有多层工具策略过滤，只配一层时插件工具仍会被过滤。

5 处必须同步配置：
1. 插件代码 api.registerTool('tool_name', {...}) 注册到 plugin runtime。
2. 插件清单 openclaw.plugin.json 的 contracts.tools 数组声明插件提供哪些工具。
3. 主 agent agents.list[0].tools.profile 设为 coding。
4. 主 agent agents.list[0].tools.alsoAllow 数组追加工具名。
5. sandbox agents.list[0].tools.sandbox.tools.alsoAllow 数组追加工具名。

根级 tools.allow 也要包含这些工具名，保证全局 allowlist 不排斥。排查命令：openclaw plugins inspect nupai-oc-router --runtime --json | python3 -m json.tool。如果 runtime 显示工具但 LLM 调不到，依次核 5 处配置。

## 结论
新增 OpenClaw 插件工具 SOP：插件代码 registerTool、插件清单 contracts.tools、主 agent profile + alsoAllow、sandbox.tools.alsoAllow、根级 tools.allow。任一遗漏都会导致 LLM 看不见工具。

## 影响
影响所有 nupai-oc-router 工具注册流程、未来 OpenClaw 插件开发、多 agent 配置、sandbox.mode = all 场景和根级 tools.allow 全局白名单。

## 下次注意
新增工具时拿这 5 处当 checklist；报 no registered tools matched 立即查 5 处；plugin inspect runtime 显示工具不代表 LLM 可见；sandbox 是独立 scope，不自动继承上层 policy。

