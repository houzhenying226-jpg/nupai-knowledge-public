# L15 · 前端列表页 page_size 默认值 + 骨架屏两个必做项

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

每个列表页的标配：page_size 限制 + 骨架屏组件 + Playwright E2E 验证骨架屏。

## 正文

> 记录日期：2026-05-22  风险等级：L2

## 背景
page_size=200 首屏慢；无骨架屏=白屏；每个列表页缺少统一的 loading 体验。

## 内容
page_size Kanban ≤50，Calendar ≤100。loading 状态必须用 <Skeleton active>，不能用文字。Playwright 验证骨架屏：先 goto /today（卸载组件），再路由拦截 + 等1200ms 截图。

## 结论
每个列表页的标配：page_size 限制 + 骨架屏组件 + Playwright E2E 验证骨架屏。

## 影响
前端所有列表页统一 page_size 规范 + Skeleton 组件 + E2E 验证，首屏加载体验明显改善。

## 下次注意
新建列表页必须同时配 page_size 上限 + <Skeleton active> + Playwright 骨架屏测试。

