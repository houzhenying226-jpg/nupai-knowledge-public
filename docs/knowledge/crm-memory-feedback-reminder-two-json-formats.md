# 提醒中心两种 JSON 内容格式需要不同解析器

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-14T18:22:07Z

## 摘要

tryParseEmpowerment 只处理 {_empowerment_version} 格式，节日/生日 [{style,content}] 数组格式必须用独立的 tryParseGreeting 解析

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_reminder_two_json_formats.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

NUP-383 修复后仍然出现 JSON 原文泄漏，原因是系统有**两种不同的 content JSON 格式**被混为一谈：

## 两种格式

| 来源 | 格式 | 起始字符 | 解析器 |
|------|------|----------|--------|
| NUP-157 赋能型（cortex/handlers/*.py） | `{"_empowerment_version":"v1","what_happened":...}` | `{` | `tryParseEmpowerment()` |
| NUP-383 节日/生日祝福词（birthday_holiday.py） | `[{"style":"formal","content":"..."},{"style":"friendly",...},{"style":"brief",...}]` | `[` | `tryParseGreeting()` ← **这个之前没有** |

## 根因

`tryParseEmpowerment()` 第一行检查：
```ts
if (!text.startsWith('{')) return null;  // 遇到 [ 开头直接返回 null
```
所以节日/生日 content 完全绕过解析，fallback 到 `{r.content}` 原文渲染。

## 修复模式

```ts
function tryParseGreeting(raw?: string | null): string | null {
  if (!raw?.trim().startsWith('[')) return null;
  try {
    const arr = JSON.parse(raw);
    const formal = arr.find((x: any) => x.style === 'formal') || arr[0];
    const c = formal?.content;
    return typeof c === 'string' && c.trim() ? c.trim() : null;
  } catch { return null; }
}
```

渲染优先级：`empowerment > greetingText > 普通文本（非[开头）> 隐藏`

## 检查铁律

新增 content JSON 格式时，必须问：
1. 起始字符是 `{` 还是 `[`？
2. 是否有现有解析器覆盖？
3. fallback 是否会泄漏原文？

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

