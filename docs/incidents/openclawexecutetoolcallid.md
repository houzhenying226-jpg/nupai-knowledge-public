# OpenClaw工具execute第一参数是toolCallId

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:00Z

## 摘要

OpenClaw 工具开发统一签名：execute: async \(_toolCallId, params\) =\> \{...\}，从 params 解构入参。parameters 必须完整 JSON Schema \+ additionalProperties: false。

## 正文

> 记录日期：2026-05-05  风险等级：L3

## 背景
nupai-oc-router 三个工具被 GPT-5.5 选择后，参数解析错位或执行失败。从用户输入"派 worker 修 STO-186"到工具内拿到的 issue_id 不正确。

## 内容
OpenClaw tool execute 函数的第一个参数是 OpenClaw 内部的 tool call id，不是用户传入的工具参数。错误写法：```PLAIN_TEXT
execute: async ({ issue_id }) => {
  // ...
}
```后果：把 _toolCallId 当成 params 解构，issue_id = undefined，函数失败。正确写法：```PLAIN_TEXT
execute: async (_toolCallId, params) => {
  const { issue_id, repo } = params;
  // ...
}
```完整工具定义模板（来自实际生效代码）：```PLAIN_TEXT
api.registerTool('dispatch_worker', {
  description: '派 worker 修 issue。当用户说"派 worker / 修 STO-xxx"时调用',
  parameters: {
    type: 'object',
    properties: {
      issue_id: { type: 'string', description: 'issue ID 如 STO-186' },
      repo: { type: 'string', description: '可选，仓库名' }
    },
    required: ['issue_id'],
    additionalProperties: false
  },
  execute: async (_toolCallId, params) => {
    const { issue_id, repo } = params;
    // POST 到 dispatcher /command
  }
});
```关键约束：parameters 必须是完整 JSON Schema，含 type / properties / requiredadditionalProperties 必须设为 false，否则 GPT-5.5 可能传额外字段导致后端报错_toolCallId 通常用不到，但参数位置不能省当前 nupai-oc-router/index.mjs 中 dispatch_worker / query_task_status / confirm_task_plan 三个工具均已使用此正确签名。

## 结论
OpenClaw 工具开发统一签名：execute: async \(_toolCallId, params\) =\> \{...\}，从 params 解构入参。parameters 必须完整 JSON Schema \+ additionalProperties: false。

## 影响
所有 nupai-oc-router 工具实现；未来任何 OpenClaw 插件工具开发；GPT-5.5 调用工具时的参数验证；额外字段被拒绝的兼容性。

## 下次注意
写新工具时复制上面的模板，不要简写参数解构；execute 第一个参数永远是 _toolCallId 加下划线提示不用；parameters 必须 JSON Schema 不能用 TypeScript 类型；additionalProperties: false 必加。

