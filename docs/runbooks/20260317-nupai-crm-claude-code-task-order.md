# CRM Bug 修复 Claude Code 三文件任务执行 SOP

> 类型：操作规程 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

IT 测试修复时，Claude Code 多任务执行顺序错误（直接跑 CC2 跳过 CC1）导致修复基于错误假设，修完又返工。正确的三文件顺序是固定的，不可跳步。

## 正文

## 背景

IT 测试修复时，Claude Code 多任务执行顺序错误（直接跑 CC-2 跳过 CC-1）导致修复基于错误假设，修完又返工。正确的三文件顺序是固定的，不可跳步。

## 正确执行顺序

```
CC-1（诊断）→ CC-2（后端修复）→ CC-3（前端修复）
```

**CC-1（必须先跑）**：连服务器读代码，把关键文件内容带入后续任务
- SSH 到服务器读路由代码、数据库 schema、中间件配置
- 不做任何修改，只读取并输出真实代码状态
- 目的：避免 CC-2 基于错误假设修复

**CC-2（后端修复）**：按每个 Fix 独立修复，改完即测试
```bash
# 每个接口修完用 curl 验证
curl -X POST http://39.106.83.79/api/customers/{id}/claim \
  -H "Authorization: Bearer {token}"
# 必须返回 200 + 正确数据，不是仅路由存在
```

**CC-3（前端修复）**：改完运行构建确认无 TypeScript 错误
```bash
cd /opt/nupai-crm/frontend && npm run build
# 必须成功，不能有报错
```

## 启动命令模板

```bash
caffeinate -s claude --dangerously-skip-permissions
```

粘贴给 Claude Code 的指令中必须包含：服务器 SSH 凭证 + 后端容器名 + 前端路径 + GitHub Token + 执行顺序 CC-1→CC-2→CC-3 + 每个 Fix 完成后 `git commit`。

## 经验教训

- 「CC-3 没执行」这类问题的根因通常是指令中未明确三个任务都要执行，或 Claude Code 错误地认为某任务已在前轮完成。每次必须明确列出所有任务。
- 任务完成报告里「API 正常」的判断必须基于真实 curl 测试，不是代码审查。
