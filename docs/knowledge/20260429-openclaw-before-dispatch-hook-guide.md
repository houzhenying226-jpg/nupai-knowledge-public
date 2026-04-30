# OpenClaw Plugin Hook 对比：before_dispatch 是唯一可靠的 LLM 拦截点

> 类型：知识条目 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 插件系统提供多个 hook，但并非所有 hook 都能实现「LLM 处理前拦截消息」。经 7+ 轮排查验证，以下是各 hook 的真实能力对比。

## 正文

## 背景

OpenClaw 插件系统提供多个 hook，但并非所有 hook 都能实现「LLM 处理前拦截消息」。经 7+ 轮排查验证，以下是各 hook 的真实能力对比。

## Hook 对比表

| Hook | 触发时机 | 能否 cancel LLM | 实测结论 |
|---|---|---|---|
| `before_dispatch` | 消息进入调度前 | ✅ 可 short-circuit | 唯一可靠拦截点 |
| `before_prompt_build` | 构建 LLM prompt 前 | ❌ 只能改 prompt | 无法取消 LLM 调用 |
| `before_agent_reply` | LLM 回复后 | ❌ 实际不可靠 | 文档说能 short-circuit，实际未必触发 |
| `inbound_claim` | 消息分发时 | ❌ stub | 只对 plugin-owned binding 触发，飞书 DM 不走 |
| `active-memory.promptAppend` | 注入 prompt 时 | ❌ 不是 hook | 禁用后仍有其他注入点 |

## before_dispatch 用法

```typescript
api.on("before_dispatch", async (event, ctx) => {
  const text = event.content?.trim() ?? "";
  const senderId = event.senderId;    // open_id
  const chatId = ctx.conversationId; // P2P DM: ou_xxx, 群聊: oc_xxx

  if (/^\/oc(\s|$)/.test(text)) {
    // 可证伪的正则判断（铁律 2）
    const resp = await fetch("http://localhost:18790/command", {
      method: "POST",
      body: JSON.stringify({ text, senderId, chatId })
    });
    return { handled: true, reply: await resp.text() };
  }
  // 不匹配 /oc 的消息走正常 LLM 流程
});
```

## 关键注意事项

- P2P DM 的 `ctx.conversationId` 是对方 open_id（`ou_xxx`），不是标准飞书 chat_id（`oc_xxx`），白名单必须两种格式都包含。
- `return { handled: true }` 是 short-circuit 信号，必须设置，否则消息仍会继续走 LLM 流程。
- 正则判断是可证伪硬闸门，不要改成让 LLM 判断「这是不是 /oc 命令」。
