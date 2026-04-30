# OpenClaw Workspace 更新 SOP

> 类型：操作规程 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

Workspace 文件直接注入 OpenClaw LLM context，错误更新会立即影响所有 agent 行为。PERMISSIONS.md 明确禁止批量覆盖，本 SOP 是唯一允许的更新方式。

## 正文

## 背景

Workspace 文件直接注入 OpenClaw LLM context，错误更新会立即影响所有 agent 行为。PERMISSIONS.md 明确禁止批量覆盖，本 SOP 是唯一允许的更新方式。

## 禁止操作（PERMISSIONS.md 铁律）

```bash
# 以下操作绝对禁止
cp -r /tmp/new-workspace/ ~/.openclaw/workspace/   # 批量覆盖
rm -rf ~/.openclaw/workspace/                        # 删除整个 workspace
claude code "更新 workspace 中的所有文件"           # 让 AI 批量修改
```

## 允许的更新方式

**单文件更新（振英手动执行）：**
```bash
# 只更新一个文件
cp /tmp/new-SOUL.md ~/.openclaw/workspace/SOUL.md
# 验证
cat ~/.openclaw/workspace/SOUL.md | head -20
```

**新增文件：**
```bash
# 新建单个文件
cat > ~/.openclaw/workspace/decisions/new-decision.md << 'EOF'
内容
EOF
```

## 更新验证步骤

1. 更新文件后，重启 OpenClaw gateway 让新文件生效：`openclaw reload`
2. 发一条测试消息到飞书命令机器人 DM，确认行为符合预期
3. 检查 `~/.openclaw/logs/gateway.err.log` 无新报错

## decisions/ 和 runbooks/ 更新规则

- 每条决策/runbook 是独立文件，不修改现有文件内容（历史不改写）
- 新增内容 = 新建文件，命名格式：`YYYYMMDD-描述.md`
- 文件长度 100-150 行，不超过 200 行

## 经验教训

- workspace 文件被 OpenClaw 注入到每次 LLM 请求中，大文件会显著增加 token 消耗。保持文件简洁是运营成本控制的一部分。
- Sprint 中间检查点（25%/50%/100%）要求在每个检查点验证 workspace 状态，避免逐步偏离。
