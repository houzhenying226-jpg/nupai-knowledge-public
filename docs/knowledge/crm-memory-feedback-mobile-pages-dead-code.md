# frontend/mobile-pages/ 是孤立死代码，不需要维护

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T09:41:50Z

## 摘要

mobile-pages/ 下的 .tsx 文件在代码库中无任何 import 引用，修 bug 时不需要同步修改这些文件

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_mobile_pages_dead_code.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 背景

调查提醒中心乱码时发现 `/Users/james/nupai-crm/frontend/mobile-pages/` 目录下有一批独立的移动端页面文件（如 `reminders_page_mobile.tsx`），与主页面逻辑存在差异（缺少 JSON 解析），担心需要同步修复。

## 结论

```bash
grep -rn "reminders_page_mobile\|mobile-pages" /Users/james/nupai-crm/frontend --include="*.tsx" --include="*.ts"
# 输出：（空）
```

`mobile-pages/` 下的所有文件**无任何 import 引用**，不在任何路由或组件树中。是孤立的未使用文件（历史遗留或草稿）。

## 实际移动端适配方式

项目用的是**同一个文件 + useMobile hook** 的模式：
```tsx
// frontend/app/reminders/page.tsx
const isMobile = useMobile();
// 根据 isMobile 条件渲染
```

不是通过 mobile-pages/ 路由。

## 操作规则

- Bug 修复时：**只改 `frontend/app/` 下的主文件**，不需要同步 mobile-pages/
- 如果 mobile-pages/ 某文件将来要激活，需先建 import 路由，再同步修 bug
- 可以考虑整体删除 mobile-pages/（但需振英确认）

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

