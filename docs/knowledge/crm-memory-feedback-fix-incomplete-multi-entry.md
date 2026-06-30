# Bug 修复不彻底的系统性根因：新增 content 格式时未更新所有消费入口

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-06-30T13:30:00Z

## 摘要

birthday_holiday.py 新增了 [{style,content}] 格式但前端没有同步解析，类似情况今后必须先列出所有消费入口再修

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_fix_incomplete_multi_entry.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

NUP-383 的修复被认为"完成了"，但节日/生日 JSON 泄漏问题在生产环境持续出现。

## 根因模式

```
后端新增格式 → 前端有解析器A → 解析器A不覆盖新格式 → fallback 原文渲染
```

具体路径：
1. `birthday_holiday.py` 生成 `[{style,content}]` 格式存入 content 字段
2. 前端有 `tryParseEmpowerment()` 但它只处理 `{_empowerment_version}` 格式
3. 新格式无法被识别 → fallback 到 `{r.content}` → 原文泄漏

## 预防铁律

**后端新增 content 存储格式时必须问**：
- [ ] 该字段的所有前端消费入口是哪些？（grep `r.content` / `reminder.content`）
- [ ] 现有解析器（`tryParseEmpowerment`、`tryParseGreeting` 等）是否覆盖新格式？
- [ ] fallback 渲染是否安全（不会泄漏 JSON 原文）？

**前端 content 渲染必须有最后一道防线**：
```tsx
// ❌ 错误：直接渲染，JSON 原文会泄漏
{r.content && <div>{r.content}</div>}

// ✅ 正确：非 JSON 才渲染
{r.content && !r.content.trim().startsWith('[') && !r.content.trim().startsWith('{') && (
  <div>{r.content}</div>
)}
```

## 当前已知的 content 格式清单

| 格式 | 来源 | 解析器 |
|------|------|--------|
| `{"_empowerment_version":"v1",...}` | cortex/handlers/*.py | `tryParseEmpowerment()` |
| `[{"style":"formal","content":"..."},...]` | birthday_holiday.py | `tryParseGreeting()` |
| 普通字符串 | 手动/其他 | 直接渲染（需非 `[`/`{` 开头） |

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

