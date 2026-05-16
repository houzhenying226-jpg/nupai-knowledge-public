# 两种乱码根因不同，处理路径完全不同

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-16T14:36:15Z

## 摘要

JSON原文泄漏（渲染bug）和DB字段mojibake（数据bug）表现相似但修法截然不同，必须分开诊断

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_two_types_of_garbled_text.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

提醒中心同时出现两种"乱码"，容易被当成同一个问题处理导致漏修。

## 两种乱码对比

| | 类型A：JSON 原文泄漏 | 类型B：DB mojibake |
|---|---|---|
| **表现** | 内容区显示 `[{"style":"formal"...}]` | 标题显示 `åŠ³åŠ¨èŠ‚` 而非 `劳动节` |
| **根因** | 前端未解析 JSON，直接渲染原始字符串 | 数据库写入时 charset 错误，UTF-8 被当 latin1 存储 |
| **修复位置** | **前端代码** | **数据库数据** |
| **修复方式** | 添加 `tryParseGreeting()` 解析函数 | SQL `CONVERT(CAST(CONVERT(field USING latin1) AS BINARY) USING utf8mb4)` |
| **是否需要重新部署** | ✅ 需要 | ❌ 不需要（直接改数据） |
| **影响新数据** | ✅ 修复后新数据也正常 | ❌ SQL 只修历史数据，需排查插入时 charset |

## mojibake 识别特征

UTF-8 中文被 latin1 解码后的典型字符：`å æ è ç é Å Æ È Ç É`

```sql
-- 识别 mojibake 行
SELECT * FROM sys_holidays WHERE holiday_name REGEXP '[åæèçéÅÆÈÇÉ]';

-- 修复
UPDATE sys_holidays
SET holiday_name = CONVERT(CAST(CONVERT(holiday_name USING latin1) AS BINARY) USING utf8mb4)
WHERE CHAR_LENGTH(holiday_name) != LENGTH(holiday_name)
  AND holiday_name REGEXP '[åæèçéÅÆÈÇÉ]';
```

## 诊断流程

遇到"乱码"时先问：
1. 内容区显示的是 `[{` 或 `{` 开头的文本？→ 类型A，改前端
2. 标题/名称字段显示 `å æ è ç é`？→ 类型B，改数据库
3. 两个同时有？→ 分开处理，不要混在一个 PR 里

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

