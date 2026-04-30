# OpenClaw Workspace 49 文件 8 层结构

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

v1 部署完成后，OpenClaw 的定位从「部署助手」升级为「公司级 AI 操作系统底座」。振英拍板「全局思考前瞻意识，一次执行不分阶段」，在 Sprint NUPOPENCLAWWS1 中一次性建立 49 个文件的完整 workspace 结构。

## 正文

## 背景

v1 部署完成后，OpenClaw 的定位从「部署助手」升级为「公司级 AI 操作系统底座」。振英拍板「全局思考前瞻意识，一次执行不分阶段」，在 Sprint NUP-OPENCLAW-WS-1 中一次性建立 49 个文件的完整 workspace 结构。

## 架构设计

```
~/.openclaw/workspace/
├── 身份层：IDENTITY / USER / SOUL / AGENTS / PERMISSIONS
├── 工具层：TOOLS / COMMANDS / INTEGRATIONS / MODEL_ROUTING / AGENT_TEAM
├── 记忆层：MEMORY / memory/ / projects/ (CRM + 门店，预留扩展位)
├── 巡检层：BOOT / HEARTBEAT / INCIDENTS / ESCALATION
├── 决策层：decisions/ (4 个文件，沉淀所有真知识)
├── Runbook 层：runbooks/ (5 个故障手册)
├── Skills 层：skills/ (6 个 NuPai 专属 skill，workspace 优先级最高)
└── 审核+Ops：REVIEW_POLICY / ops/ (4 个，不写明文 secret，只索引位置)
```

文件长度纪律（workspace 注入 LLM context，大文件浪费 token）：AGENTS.md ≤200 行 / SOUL.md ≤80 行 / HEARTBEAT.md ≤50 行 / 单个 SKILL.md <100 行 / decisions/runbooks 100-150 行上限。

## 经验教训

- workspace 是 OpenClaw 自己的家，绝不和项目代码混在一起，独立路径 ~/.openclaw/workspace/。
- projects/ 是多项目记忆中枢，未来添加新产品线只需复制结构，零改动。
- 禁止 Claude Code 批量覆盖 workspace（cp -r 覆盖整个目录是危险操作）——更新必须振英手动单文件 cp，已写入 PERMISSIONS.md 绝对禁止清单。
- WS-2 补丁发现 5 条问题（断链/路径硬编码/不安全 JSON 写入/命令稳健性/自相矛盾规则），说明一次性建完后仍需第三方审计。
