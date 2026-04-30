# CRM 移动端从未真机测试导致大量 UI 问题

> 类型：故障复盘 | 项目：nupai-crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

IT3 测试发现 B18/B19 移动端布局混乱：选项卡超出屏幕无横向滚动、双列布局在小屏幕直接溢出、操作按钮重叠。这些问题在 PC 端完全不存在，是移动端从未经历真机测试的集中爆发。

## 正文

## 背景

IT3 测试发现 B18/B19 移动端布局混乱：选项卡超出屏幕无横向滚动、双列布局在小屏幕直接溢出、操作按钮重叠。这些问题在 PC 端完全不存在，是移动端从未经历真机测试的集中爆发。

## 根因分析

整个系统设计以 PC 为主，移动端是附带适配：
- Tab 超过 6 个未改为下拉或分组
- 表格未包裹 `overflow-x-auto`
- grid 布局使用固定 `grid-cols-3` 未设响应式 `md:grid-cols-3`
- 按钮触摸区域小于 44px
- Form 未设置 `layout={isMobile ? 'vertical' : 'horizontal'}`

## 处理方式

按移动端铁律逐项修复：
- 所有 Card 宽度改为 `width: 100%`
- Drawer 改为 `width="100%"`
- Modal 改为 `width={isMobile ? '95vw' : 600}`
- 所有 Table 包裹 `overflow-x-auto`
- 使用 `useMobile` hook 判断设备类型

检查命令：`grep -rn "grid-cols-[2-9]" frontend/components/ --include="*.tsx" | grep -v "md:grid-cols"`

## 经验教训

- 每个新增组件必须在开发时同步做移动端适配，不能「先 PC 后移动」，等集中适配时已积累大量问题。
- 移动端测试必须用真机（而非浏览器缩放），浏览器缩放看不到触摸区域大小和手势交互问题。
- 已记录在 `.claude/rules/mobile-first.md` 移动端铁律中，新 PR 合并前自动检查。
