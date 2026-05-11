# Playwright 测试用 innerText 不用 textContent

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-11T10:30:00Z

## 摘要

Playwright 检查用户可见内容必须用 innerText，避免 textContent 读取隐藏 Tab DOM 导致误判。

## 正文

> 记录日期：2026-05-11  风险等级：L2

## 背景
E6.1 情报 Tab JSON 检测持续失败。测试使用 page.textContent('body') 检测情报 Tab 是否显示原始 JSON，但 Ant Design 非激活 Tab pane 用 display:none 隐藏，textContent 仍会读取隐藏 DOM 文本。

## 内容
AI分析Tab / AI攻关Tab 中有 JSON 内容，即使情报Tab显示正确，textContent 仍可能读到隐藏 JSON 导致测试失败。已在 PR #544 将 E6 测试改为 page.evaluate(() => document.body.innerText)。renderIntelContent 的 JSON 兜底禁止 JSON.stringify(parsed, null, 2)，应提取字段值为纯文本。

## 结论
检测用户可见内容必须用 innerText，不用 textContent；更精准时只检测 .ant-tabs-tabpane-active。

## 影响
降低前端 E2E 因隐藏 DOM 产生的假失败，避免误判用户界面仍显示 JSON。

## 下次注意
所有“用户可见内容”断言使用 document.body.innerText 或 active tab locator；JSON 渲染兜底只输出字段值，不输出 JSON 语法。

