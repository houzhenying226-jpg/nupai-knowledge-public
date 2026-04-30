# before_dispatch hook 发现历程：7 轮排查终找真可用 hook

> 类型：故障复盘 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 命令机器人 DM 所有消息（包括 /oc 状态）被 brain LLM 拦截，返回固定 5 段格式，真实业务命令完全无法到达 dispatcher。排查历经 7+ 轮，最终找到真可用 hook。

## 正文

## 背景

OpenClaw 命令机器人 DM 所有消息（包括 /oc 状态）被 brain LLM 拦截，返回固定 5 段格式，真实业务命令完全无法到达 dispatcher。排查历经 7+ 轮，最终找到真可用 hook。

## 7 轮失败尝试

| 尝试 | 操作 | 结果 | 原因 |
|---|---|---|---|
| 1 | 改 nupai-claude-ai-brain SKILL.md | 失败 | brain 不读这个文件 |
| 2 | 改 PR #335 SKILL.md（覆盖目录错） | 失败 | PR 内容未部署到运行时 |
| 3 | 改 AGENTS.md | 失败 | 不是 bootstrap 文件 |
| 4 | 禁用 active-memory.promptAppend | 失败 | 还有其他注入点 |
| 5 | 砍掉 OpenClaw gateway | 未执行 | 振英拒绝：会失去 6 个内置插件 |
| 6 | 用 inbound_claim hook | 失败 | stub，只对 plugin-owned binding 触发 |
| 7 | 用 before_agent_reply hook | 失败 | 文档说能 short-circuit，实际不触发 |

## 真根因与修法

Claude Code 自挖 OpenClaw plugin SDK 类型定义，找到真可用 hook：`before_dispatch`（OpenClaw 2026.4.22 官方 hook）。

```typescript
api.on("before_dispatch", async (event, ctx) => {
  if (/^\/oc(\s|$)/.test(event.content)) {
    // 正则匹配，POST 到 dispatcher
    return { handled: true, reply: "..." }; // short-circuit LLM
  }
});
```

三模型互审（GPT-5.5 + Claude Opus 4.7 + Gemini 3.1 Pro）一致确认：OpenClaw 2026.4.22 真有官方 short-circuit LLM 机制。

## 经验教训

- before_prompt_build 不能 cancel LLM（只能改 prompt）；before_agent_reply 文档说能 short-circuit 但实际不可靠；**before_dispatch 是唯一真正可在 LLM 处理前拦截的 hook**。
- 「砍腿思维」（砍掉 gateway）看似快，实际失去 6 个内置插件。有官方 hook 就用官方 hook，不要砍基础设施。
- Claude Code 自挖 SDK 类型定义找到真相，比读文档更可靠——文档可能落后于实现。
- 见 `decisions/brain-route-hijack-root-cause.md` 了解架构层决策，本文件记录排查过程。
