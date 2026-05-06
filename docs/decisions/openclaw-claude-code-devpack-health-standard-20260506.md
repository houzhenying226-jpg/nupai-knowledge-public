# Claude Code 到 OpenClaw 自动开发包与健康收口标准

> 类型：架构决策 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-06T02:34:48Z

## 摘要

Claude Code 会员侧只产出蓝图、Spec、Task 和 devpack；OpenClaw 作为无人值守工厂，必须先通过模型路由、skill 冲突、gateway、任务审计和安全门验证后再接自动开发。

## 正文

> 记录日期：2026-05-06  风险等级：L2

## 背景
用户使用 Claude Code 会员做系统开发规划，但 Claude Code 与 OpenClaw 隔离，不能依赖 Claude API 或 Claude Code 被 OpenClaw 自动调用。用户目标是让 OpenClaw 承担自动开发、PR、CI、部署和回报闭环，但此前存在冲突 skill、错误模型路由、active-memory 401/超时、旧格式 prompt 注入、以及任务状态容易误判的问题。

## 内容
本次修复将 OpenClaw 默认模型切到 openai-codex/gpt-5.5，fallback 为 openai-codex/gpt-5.4、Qwen、DeepSeek；普通 OpenAI API provider 不再进入自动路由。禁用 active-memory 残留配置，避免错误 embeddings key 导致 401 和 before_prompt_build 超时。禁用 nupai-claude-ai-brain，使旧五段式回复 skill 不再对模型可见，保留 nupai-ai-brain。新增 openclaw-devpack-ingest skill，并在 OpenClaw workspace 写入 Claude Code 到 OpenClaw devpack 标准。

devpack 标准要求 Claude Code 每次交付 `.openclaw/devpacks/<WORK_ID>/`，至少包含 `task.json`、`SPEC.md`、`TASKS.md`、`ACCEPTANCE.md`、`RISKS.md`。OpenClaw 只能从 devpack 派发 worker，不从模糊聊天猜需求。风险等级分 D0-D3：repo-only 文档、测试、代码改动可自动推进；生产部署、数据库迁移、回滚、云权限、密钥和不可逆数据操作属于 D3，必须生成审批卡，不得自动执行。

健康验收采用真账：`openclaw config validate` 必须通过；`openclaw models` 默认必须为 Codex OAuth 路由；冲突 skill 必须不可见；`openclaw gateway probe` 必须 reachable 且 read probe ok；`openclaw tasks audit --json` finding 必须为 0；`openclaw status --deep` 不得有 critical，安全 warning 必须被解释为策略选择。status 中历史 failed task 不等于当前队列故障，必须同时查看 queued/running/lost/timed_out 和 audit findings。

## 结论
Claude Code 负责蓝图、Spec、Task、验收标准和 devpack 产物；OpenClaw 负责接包、验证、派 worker、开 PR、跑 CI、处理 review、低风险部署和可见回报。自动工厂启动前必须先跑健康门，不允许在 active-memory 报错、模型 fallback 异常、skill 冲突或任务审计不清时继续自动开发。

## 影响
以后 Claude Code 的输出不再是聊天文本，而是机器可消费的 devpack。OpenClaw 自动化从“靠提示词约束”升级为“文件协议 + skill 接包 + 风险门 + 真账验收”。这样既保留 Claude Code 会员在规划和规格整理上的优势，又避免 OpenClaw 依赖昂贵或不可用的 Claude API。

## 下次注意
1. Claude Code 产物必须按 devpack 标准落盘，缺 `task.json`、`SPEC.md` 或 `ACCEPTANCE.md` 时 OpenClaw 应拒绝派发。
2. 每次改 OpenClaw 配置后必须重启 gateway，并确认 PID 更新、gateway probe 正常、config validate 通过。
3. 看到 `3 issues` 这类状态摘要时，先查 `openclaw tasks audit --json` 和 failed/lost/timed_out 分类，不要把历史失败当作当前队列故障。
4. `exec security=full` 是无人值守能力与安全之间的策略选择；生产、DB、密钥、回滚必须继续走 D3 审批门。
5. 不要把 API key、token、真实凭据或敏感地址写入知识卡正文；验收只写结论和命令类型，不贴秘密原文。

