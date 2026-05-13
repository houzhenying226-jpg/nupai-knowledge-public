# 新页面必须加在src/不能加在frontend-v2/

> 类型：架构决策 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

[CLAUDE.md](CLAUDE.md) 铁律："线上代码在 src/，frontend-v2/ 暂未上线，绝不修改"。新功能一律在 src/ 下开发，frontend-v2/ 只作参考，不写入。

## 正文

> 记录日期：2026-05-13  风险等级：L4

## 背景
项目有两个前端目录：src/（Vite 构建入口，线上运行）和 frontend-v2/（历史遗留，未接入构建，不上线）。WecomSidebar 组件最初只在 frontend-v2/ 里，导致构建产物不含该路由，线上 404。

## 内容
两个目录的定位：
- src/ — Vite 入口（vite.config.ts root），构建产物进 dist/，线上运行的代码
- frontend-v2/ — 遗留目录，有独立的 tsconfig/api client，不参与主构建，代码不上线
踩坑症状：
- 在 frontend-v2/ 里写组件，线上访问对应路由 → React Router "404 Not Found"
- Playwright Network 里无对应 chunk 文件（如无 WecomSidebar-BjaQQcRQ.js）
- 本地 dist/ 里也无该 chunk（因为根本没被构建进去）
正确路径：
- 页面组件：src/pages/store/ComponentName.tsx
- API 客户端：src/api/feature.api.ts（使用 import client from './client'，default export）
- 路由注册：src/router/index.tsx，用 L\(\(\) =\> import\('@/pages/store/ComponentName'\)\) 懒加载

## 结论
[CLAUDE.md](CLAUDE.md) 铁律："线上代码在 src/，frontend-v2/ 暂未上线，绝不修改"。新功能一律在 src/ 下开发，frontend-v2/ 只作参考，不写入。

## 影响
- src/pages/store/WecomSidebar.tsx（已从 frontend-v2/ 复制并修正）
- src/api/wecom.api.ts（新建，适配 src/ 的 client 导出方式）
- src/router/index.tsx（已加 /wecom-sidebar 路由）

## 下次注意
看到 frontend-v2/ 里有参考实现时，必须手动复制到 src/ 并调整 import 路径（apiClient → client）和 API 文件名（wecom → wecom.api），不能直接引用 frontend-v2/ 的文件。

