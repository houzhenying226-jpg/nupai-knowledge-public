# Feishu direct session 验证不能用默认 main session 代替

> 类型：知识条目 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

验证飞书回复格式时，必须用 sessions.json 中真实 Feishu direct session id，默认 main session 可能被旧上下文污染。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
修改 AGENTS.md、HEARTBEAT.md 和 heartbeat prompt 后，默认 CLI session 仍输出旧段落格式，容易误判为未生效。但真实 Feishu direct session 后来验证可输出 Markdown 表格。

## 内容
真实验证方法：
1. 查 `~/.openclaw/agents/main/sessions/sessions.json`。
2. 找到 `agent:main:feishu:direct:<open_id>` 对应 session id。
3. 用真实 session id 发受控测试。
4. 判断输出是否符合 Markdown 表格规则。

## 结论
默认 `agent:main:main` 不能代表飞书真实会话。OpenClaw 多 session 环境中，格式验证必须绑定目标 channel/session。

## 影响
以后用户问“生效了吗”，必须给真实链路验证，不接受默认 session smoke test 代替飞书 direct session。

## 下次注意
如果默认 session 和 Feishu direct session 行为不同，以真实 Feishu direct session 为准。

