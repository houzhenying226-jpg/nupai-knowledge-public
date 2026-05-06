# channels.feishu.heartbeat仅含visibility与intervalMs

> 类型：故障复盘 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

OpenClaw 2026.5.3-1 的 channels.feishu.heartbeat schema 只允许 visibility 和 intervalMs，showOk 等字段会导致 config validate 失败。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
nupai-oc-router 部署时尝试在 channels.feishu.heartbeat 下配置 showOk 字段，期望控制 OK 状态显示。openclaw config validate 报错 channels.feishu.heartbeat: invalid config: must NOT have additional properties。

## 内容
OpenClaw 版本 2026.5.3-1 的 schema 中 channels.feishu.heartbeat 只有两个合法字段：visibility（visible / hidden）和 intervalMs（毫秒整数）。不存在 showOk、showError、template 等字段，任何额外字段都会被 schema 拒绝。

正确配置是 channels.feishu.heartbeat.visibility + intervalMs。如果想实现“只在出错时显示”或“OK 时静默”，不能在 channels.feishu.heartbeat 实现，要去 agents.defaults.heartbeat 层配置；那一层有 prompt、suppressToolErrorWarnings 等行为字段。

schema 复核命令：openclaw config schema 后查看 properties.channels.properties.feishu.properties.heartbeat。agent heartbeat schema 复核命令：查看 properties.agents.properties.defaults.properties.heartbeat。

## 结论
心跳配置分两层：channels.feishu.heartbeat 只管可见性和间隔；agents.defaults.heartbeat 管心跳行为。报错信息控制在 agent 层，不在 channel 层。

## 影响
影响所有飞书心跳行为配置。OpenClaw 升级后 schema 可能新增字段，需要重跑 schema 复核命令。agents.defaults.heartbeat 是真正的行为控制层。

## 下次注意
配置任何 OpenClaw 字段前先用 schema 命令查合法字段；报 additional properties 立即查 schema；分清显示问题是 channel 层，行为问题是 agent 层；记录 OpenClaw 版本号。

