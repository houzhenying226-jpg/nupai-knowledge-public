# L12 · Python list[0] vs list[0][0] 单字符 bug 全局排查

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

对 list[str] 取第一个元素，只用 [0]，绝不用 [0][0]。grep -rn "[0][0]" backend/app/ --include="*.py" → 必须 0 行。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
visit_purpose 存单字符根因：req.visit_purposes[0][0] 把字符串第一个字符当成列表元素。

## 内容
visit_purposes[0] = "qualification" ← 正确。visit_purposes[0][0] = "q" ← 错误！取了字符串第一个字符。根因是混淆了 list[str] 索引和字符串索引，[0] 取列表元素、[0][0] 取字符串第一个字符。

## 结论
对 list[str] 取第一个元素，只用 [0]，绝不用 [0][0]。grep -rn "[0][0]" backend/app/ --include="*.py" → 必须 0 行。

## 影响
加守卫脚本防止同类 bug。所有 list[str] 索引操作纳入代码审查重点检查项。

## 下次注意
任何 list[str] 只取 [0]，禁止 [0][0]；代码审查重点检查多级索引。

