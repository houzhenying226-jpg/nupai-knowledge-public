# 【置顶】FESUN Platform 新项目启动工程化标准：Layer -1 到 Gate-0

> 类型：操作规程 | 项目：_global | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-24T09:30:00Z

## 摘要

FESUN 新项目写代码前必须完成的标准化准备流程：GitHub/Linear/Apifox/Specs/安全工具链/RLS/LiteLLM/Gate-0，并固化为 Template Repo、bootstrap.sh、Linear CSV、飞书 Gate Checklist 和 ADR 复盘机制。

## 正文

> 记录日期：2026-05-24  风险等级：L2

## 背景
侯爷基于 Claude Code 生成的计划做了专业评审，确认 8 层 31 项 + Layer -1 的工程化框架整体合理，但必须补充 Apifox 并行初始化、Secret scanning/dependabot、RLS 跨租户测试、LiteLLM fallback/retry、飞书知识库与 GitHub 同步规范。该文档用于沉淀 FESUN Platform 以及未来新项目的写代码前准备工作标准，重要级别高，建议置顶。

## 内容
# 【置顶】FESUN Platform 新项目启动工程化标准：Layer -1 到 Gate-0

## 1. 方案评估结论

Claude Code 的框架整体合理：8 层 31 项 + Layer -1 的结构逻辑严谨，符合大厂标准。
正确顺序是 Layer -1 → Layer 0 → Layer 8：基础设施先行，再进入仓库、质量、DB、API、前端、Agent、监控和冒烟。

优点：
- Layer -1 → Layer 0 → Layer 8 的顺序正确，基础设施先行。
- 每层有明确验收标准，避免“差不多能跑”。
- RLS + 多租户 + Vault 等安全层设计完整。

必须补充/修正：
| 问题 | 影响层 | 建议 |
|---|---|---|
| Layer -1 缺少 Apifox Workspace 初始化顺序说明 | M7 | Apifox 可与 GitHub 并行，不需要等 Specs 入库完成 |
| Layer 2 缺 Secret scanning / Dependabot 配置 | L2 | 大厂标准必须有，否则合规审计过不了 |
| Layer 3 RLS 策略没有测试用例 | L3-L4 | 单独写 pytest 验证跨租户隔离 |
| Layer 6 LiteLLM fallback 策略未提 | L6-1 | 7 别名要配 fallback + retry，否则上线一挂全挂 |
| 缺 Layer -1 的 M8：Notion/飞书知识库同步规范 | 团队协作 | Specs 文档在 GitHub 和飞书之间的同步机制要说清楚 |

## 2. 执行顺序总览

```text
M1-M3 (GitHub) → M4-M6 (Linear) → M7 (Apifox) ← 并行
 ↓
Layer 0 (环境核验)
 ↓
Layer 1 (仓库骨架)
 ↓
Layer 2 (质量工具链)
 ↓
Layer 3 (DB基础) + Layer 4 (API骨架) + Layer 5 (前端骨架) ← 可并行
 ↓
Layer 6 (Agent基础设施)
 ↓
Layer 7 (监控基础)
 ↓
Layer 8 (冒烟测试) → Gate-0 通过 → Sprint 1
```

## 3. Layer -1：项目基础设施（全部完成才开始 L0）

### M1 — GitHub 建 FESUN Org + Monorepo

操作清单：
- 建 GitHub Organization：FESUN。
- 建 Monorepo：`fesun-platform`，初始化 `main` 分支。
- 邀请团队成员并按角色分配权限：SRE=Admin，BE/FE/DBA=Write，QA=Triage。
- 验收：`git clone git@github.com:FESUN/fesun-platform.git` 成功。

### M2 — 分支保护 + CODEOWNERS + Workflows

分支保护规则（main + develop）：
```text
- Require PR before merge
- Require 1 approving review
- Dismiss stale reviews on new push
- Require status checks: ci / lint / typecheck
- Restrict push to admins only
```

CODEOWNERS 示例：
```text
/apps/backend/ @fesun/backend-team
/apps/frontend/ @fesun/frontend-team
/infra/ @fesun/sre-team
/Specs/ @fesun/product-team
```

必须有 4 个 Workflow：
| 文件名 | 触发条件 | 内容 |
|---|---|---|
| ci.yml | PR → main/develop | lint + typecheck + unit test |
| cd-staging.yml | push → develop | 自动部署 staging |
| cd-prod.yml | push → main + manual approve | 生产部署 |
| security.yml | 每日 cron + PR | Dependabot + Secret scanning |

### M3 — /Specs 目录入库

目录结构：
```text
Specs/
├── PRD/              # 产品需求文档
├── API/              # 10 个 OpenAPI yaml
│   ├── auth.yaml
│   ├── customers.yaml
│   ├── measurements.yaml
│   └── ...
├── DB/               # 数据库设计文档 + ERD
├── Architecture/     # 系统架构图
├── Agent/            # Agent 行为规范
└── ADR/              # Architecture Decision Records
```

验收：GitHub 上 `/Specs` 目录可见，所有 yaml 通过 `yamllint`。

### M4 — Linear Workspace + 9 个 Milestone

建立规范：
- Workspace：FESUN Platform。
- 9 个 Milestone：M0~M8，对应 Sprint 周期。
- 每个 Milestone 设置截止日期 + Responsible person。
- Label：bug / feature / infra / blocked / spec-needed。

### M5 — Linear 导入 61 条任务

CSV 字段要求：
```text
Title,Description,Team,Priority,Milestone,Estimate,Labels,Assignee,Dependencies
"L0-1: docker-compose 开发栈起动","验收：PG/Redis/Qdrant/OS/MinIO 全 healthy","SRE",1,"M0",2,"infra","owner",""
```

补充字段：
- Assignee：每条任务必须有 owner，不能留空。
- Estimate：SRE 类 1-2，BE/FE 类 2-5。
- Dependencies：跨层依赖用 Linear Blocking 关系标记。

### M6 — 10 条跨团队依赖 Issue

必须覆盖：
| 依赖 Issue | Blocking | Blocked by |
|---|---|---|
| L0-4 火山引擎 Key → L6-1 LiteLLM | SRE | BE-3 |
| L3-1 TenantMixin → L4-2 JWT+RLS | DBA | BE |
| L1-3 .env.example → L0-4 Vault Key | BE | SRE |
| L4-3 OpenAPI codegen → L5-3 TS types | BE | FE |
| M3 Specs 入库 → L4-3 codegen | BE | BE |

### M7 — Apifox Workspace + 10 个 yaml 导入

操作清单：
- 建 Apifox Project：FESUN Platform。
- 导入 `/Specs/API/` 下 10 个 yaml。
- 开启 Mock Server，Base URL 写入 `.env.example`。
- 配置 Team 权限，所有成员可访问。
- 验收：`curl http://mock.apifox.fesun.internal/health → 200`。

### M8 — 飞书知识库 / GitHub 同步规范

必须明确：
- GitHub `/Specs` 是工程真源。
- 飞书知识库是团队阅读层，承接评审、运营、非技术成员阅读。
- 每次 Specs 变更必须同步到飞书知识库，或在飞书文档里链接 GitHub commit/PR。
- 重要 Runbook/ADR 必须入飞书知识库 + GitHub，并标记“置顶/重要”。
- 文档写入不走代码修复 hook，不需要 Spec/reviewer/冒烟；但必须有入库证据（record_id、URL、GitHub path/commit）。

## 4. Layer 0~8 关键补充项

### 补充 L2：Security 工具链

CI 增加：
```yaml
- name: Secret scanning
  uses: trufflesecurity/trufflehog@main

- name: Dependency audit
  run: pip-audit && pnpm audit
```

仓库 Settings 开启：
- Dependabot alerts。
- Dependabot security updates。
- Secret scanning push protection（阻止含密钥的 commit）。

### 补充 L3-L4：RLS 跨租户隔离测试

DDL 之后立即写 pytest：
```python
# tests/test_rls.py
def test_tenant_isolation(db_tenant_a, db_tenant_b):
    # 用租户 A 的连接写入数据
    # 用租户 B 的连接查询，必须返回空
    assert db_tenant_b.query(Customer).count() == 0
```

### 补充 L6-1：LiteLLM Fallback 策略

```yaml
model_list:
  - model_name: jimeng-primary
    litellm_params:
      model: volcengine/...
      timeout: 30
  - model_name: jimeng-fallback
    litellm_params:
      model: volcengine/... # 备用 endpoint

router_settings:
  num_retries: 3
  retry_after: 5
  fallbacks:
    - {"jimeng-primary": ["jimeng-fallback"]}
```

要求：7 个模型别名都要配 fallback + retry，避免单点故障。

## 5. Gate-0：进入 Sprint 1 的门槛

所有条件必须同时满足：
- Layer -1 全部 M1~M8 验收通过。
- Layer 0~7 所有绿灯。
- L8-1 `GET /health → 200`。
- L8-2 `POST /auth/login → JWT`，JWT 含 2h TTL。
- L8-3 `POST /customers → 201`，带 RLS 落库。
- CI 全绿，至少跑过一次真实 PR。
- 至少 1 名团队成员（非 SRE）跟着文档独立跑通 L0-1。

预计工期：Layer -1 约 1~2 天，Layer 0~8 约 3~5 天，总计 1 周内完成所有前置工作，第 2 周进入 Sprint 1 正式编码。

## 6. Linear 架构配置

Linear 不需要额外“搭建架构”，但必须做配置：
- GitHub 集成：Linear Settings → Integrations → GitHub，关联 FESUN Org；PR 写 `Fixes LIN-42` 自动关闭 Issue。
- Cycle 配置：每个 Cycle 2 周，对应 M0~M8 Milestone。
- Triage View：建立 `Blocked` filter view，每日站会排卡点。
- Webhook：Linear → 飞书/企微通知，让非技术成员也能看到进度。

## 7. 标准化复用：三层固化

| 层级 | 形式 | 作用 |
|---|---|---|
| 模板层 | GitHub Template Repo + Cookiecutter | 一条命令生成骨架 |
| 流程层 | Runbook / Checklist（飞书文档） | 每次开新项目对照执行 |
| 自动化层 | CLI 脚本 / GitHub Actions | 能自动的全部自动 |

### 7.1 建 Template Repo

在 FESUN Org 下建 `fesun-template`，沉淀：
- 完整目录结构：`apps/`、`packages/`、`infra/`、`Specs/`。
- CI workflow、CODEOWNERS、lefthook.yml。
- `.env.example`、`pyproject.toml`、`tsconfig.json`。

GitHub 设置为 Template repository。以后新项目用 `Use this template` 30 秒出骨架，Layer 1 基本免费得到。

### 7.2 写 bootstrap.sh

把 Layer -1 + Layer 0 的手工操作脚本化：
```bash
#!/bin/bash
# bootstrap.sh <project-name>
PROJECT=$1

# 1. 从 template 克隆
gh repo create FESUN/$PROJECT --template FESUN/fesun-template --private

# 2. 设置分支保护
gh api repos/FESUN/$PROJECT/branches/main/protection --method PUT -F ...

# 3. docker-compose up
cd $PROJECT && docker-compose up -d

# 4. vault 写入
vault kv put secret/$PROJECT VOLCANO_KEY=$VOLCANO_KEY

echo "✅ Gate -1 通过，开始 L0 核验"
```

### 7.3 Linear CSV 模板复用

把 61 条任务保存为模板：
- 下次新项目只改 Milestone 日期和 Assignee。
- 任务结构、依赖关系全部复用。
- 存在飞书文档：`/研发标准/Linear任务模板.csv`。

### 7.4 飞书多维表格做 Gate Checklist

把 8 层 31 项做成飞书多维表格：
- 每行一个检查项。
- 字段：状态（未开始/进行中/✅）、负责人、验收时间、证据链接。
- 每次开项目复制模板表。
- 全绿后才进入 Sprint 1。

### 7.5 ADR 记录偏差

每个项目结束在 `/Specs/ADR/` 写一篇 `ADR-000-项目复盘`：
- 哪几项实际跳过，为什么。
- 哪项耗时超预期，下次如何改进。
- 有没有新增检查项。

几个项目跑下来，Checklist 会越来越精准，越来越符合团队实际。

## 8. 长期维护节奏

```text
新项目启动
 → 运行 bootstrap.sh（10分钟）
 → 复制飞书 Gate Checklist（2分钟）
 → Linear 导入 tasks.csv 模板（5分钟）
 → 按层执行，每层完成更新 Checklist 状态
 → Gate-0 全绿 → Sprint 1
 → 项目结束写 ADR 复盘 → 更新模板
```

关键原则：Template Repo 是“源代码”，Runbook 是“操作手册”，两者都要版本控制。每次迭代改模板，而不是每个项目单独维护。目标是 6 个月后新项目启动成本从 1 周压缩到 1 天以内。

## 结论
FESUN 新项目写代码前必须按 Layer -1 到 Gate-0 走完：GitHub/Linear/Apifox/Specs/安全工具链/RLS/LiteLLM/监控/冒烟全部绿灯后，才允许进入 Sprint 1。该流程必须模板化、脚本化、Checklist 化，并通过 ADR 持续迭代。

## 影响
未来 FESUN 及 NuPai 新项目启动不再依赖临场经验；可通过 Template Repo、bootstrap.sh、Linear CSV、飞书 Gate Checklist 将启动周期从约 1 周压缩到 1 天以内，同时满足安全、审计、跨团队协作和可复用要求。

## 下次注意
新项目启动时先跑 Gate -1 / Gate-0，不要直接写业务代码。重要文档必须同时入飞书知识库和 GitHub；飞书知识库负责阅读与置顶，GitHub 负责版本控制。文档写入属于知识库流程，不触发代码修复 Spec/reviewer/冒烟。

