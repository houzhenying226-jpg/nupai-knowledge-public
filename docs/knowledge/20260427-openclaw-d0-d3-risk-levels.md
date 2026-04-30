# OpenClaw D0-D3 风险分级体系

> 类型：知识条目 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw dispatcher 在接收任务时，通过 risk_classifier.py 对每个任务分配 D0D3 风险等级，决定是自动执行还是推升人工审批。这是铁律 2（LLM 决策必须有可证伪硬闸门）的具体落地。

## 正文

## 背景

OpenClaw dispatcher 在接收任务时，通过 risk_classifier.py 对每个任务分配 D0-D3 风险等级，决定是自动执行还是推升人工审批。这是铁律 2（LLM 决策必须有可证伪硬闸门）的具体落地。

## 四级分类

| 等级 | 描述 | dispatcher 行为 | 典型场景 |
|---|---|---|---|
| D0 | 低风险，无业务影响 | 直接 spawn worker，auto-merge | 文档修改、测试修复、UI 微调 |
| D1 | 中低风险，有业务影响但可回滚 | spawn worker，auto-merge | 普通 bug fix、新增 API 字段 |
| D2 | 中高风险，需要 Spec 双 AI 互审 | spawn_spec_worker（DeepSeek+Qwen 互审）→ DEV | 架构改动、数据库迁移、权限变更 |
| D3 | 高风险，必须人工审批 | 推飞书审批卡片 → 振英 [Approve] | 生产部署、数据删除、密钥轮换 |

## 判断规则（risk_classifier.py 核心逻辑）

```python
# D3 直接触发条件（T13）
D3_PATTERNS = [
    r'deploy.*prod',
    r'drop\s+table',
    r'delete.*database',
    r'rollback',
    r'secret.*rotate',
]
# D3 任务必须移除 ai-fix label + 加 needs-human label
```

D3 判定后：dispatcher 推飞书 D3 审批卡片 → 振英点 [Approve] → GitHub Environment Required Reviewer 审批 → deploy.yml 执行。

## 经验教训

- D3 是「最小人工参与」原则的体现：只有真正高风险的操作才需要振英，其余全部自动化。
- D0/D1 auto-merge 的前提是 11 checks + AI Review JSON Gate 全绿，不是「机器直接合并」。
- 重复触发 D3 任务的问题（T13 修复）：D3 任务处理时移除 ai-fix label + 加 needs-human label，避免 dispatcher 重复扫描到同一任务。凭据：dispatch_handler.py:155。
