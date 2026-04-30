# 枚举值前后端约定不一致导致英文直显

> 类型：故障复盘 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

IT3 测试多处发现英文枚举值直接展示给用户：客户「规模」显示「large」、拜访类型显示「onsite」、情绪显示「neutral」。竞品档案的删除按钮显示 删除（汉字「删除」被多转义一次）。

## 正文

## 背景

IT3 测试多处发现英文枚举值直接展示给用户：客户「规模」显示「large」、拜访类型显示「onsite」、情绪显示「neutral」。竞品档案的删除按钮显示 `删除`（汉字「删除」被多转义一次）。

## 根因分析

后端存储英文枚举是正确设计（标准化、利于排序过滤），问题在前端：AI 生成代码时未添加中文映射层，直接将后端原始值 `{customer.size}` 渲染到 UI。

Unicode 转义 `删除` 的根因：AI 生成代码后从未在浏览器真实渲染验证，JSON.stringify 的输出被错误地当成了显示文本。

## 修复方式

```typescript
// 在组件中加枚举映射
const SIZE_MAP: Record<string, string> = {
  small: '小型', medium: '中型', large: '大型', enterprise: '超大型'
};
const VISIT_TYPE_MAP: Record<string, string> = {
  onsite: '上门拜访', phone: '电话', online: '线上会议'
};
// 渲染时：{SIZE_MAP[customer.size] ?? customer.size}
```

## 经验教训

- 后端枚举 → 前端显示必须有翻译层，这是不可省略的前端责任。建议在 `constants/enums.ts` 集中定义所有映射。
- AI 生成的显示代码必须在真实浏览器渲染后验证，截图中出现英文 or Unicode 转义码 = 翻译层缺失的信号。
- Unicode 转义字符显示（`删除`）几乎都是 JSON 字符串被错误渲染为文本，不是编码问题，检查代码中是否有 `JSON.stringify` 的结果直接用于显示。
