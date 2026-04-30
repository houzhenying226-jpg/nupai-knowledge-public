# OpenClaw 7 层架构最终版

> 类型：架构决策 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 是 NuPai 内部自主开发的多 LLM 协作平台，核心命题：让多 LLM Worker 在严格闸门和反馈机制下自主完成开发全流程，任何高风险环节自动升级人工。经 Sprint12 + 批次 1/2，7 层架构于 20260427 完整落地。

## 正文

## 背景

OpenClaw 是 NuPai 内部自主开发的多 LLM 协作平台，核心命题：让多 LLM Worker 在严格闸门和反馈机制下自主完成开发全流程，任何高风险环节自动升级人工。经 Sprint-12 + 批次 1/2，7 层架构于 2026-04-27 完整落地。

## 架构决策

| 层 | 职责 | 实现 |
|---|---|---|
| L1 入口 | 飞书命令机器人 DM + GitHub Issue label + Sentry webhook | App ID cli_a93a858dd3389e19 |
| L2 事实源 | GitHub PR / Checks 唯一真相 | 禁止 LLM 口头宣布完成 |
| L3 调度 | dispatcher.sh launchd 守护，60s polling + webhook 事件驱动 | dispatch_handler.py 880 行，纯 Python 无 LLM |
| L4 执行 | ClawTeam tmux + 多 LLM worker，K8 路由器自动选 | Codex / Qwen / DeepSeek / Gemini |
| L5 审查 | Spec 双 AI 互审 + PR Review | DeepSeek+Qwen（spec）/ Codex+Gemini（PR） |
| L6 闸门 | branch protection 11 checks + AI Review JSON Gate | lint/test/typecheck/e2e/blueprint-check/smoke |
| L7 修复闭环 | Gate fail → Fix Worker，T16 上限 3 轮 | round 计数器 + escalated label + 飞书 @ 振英 |

端到端数据流：入口 → dispatcher 60s polling / 事件驱动 → risk_classifier.py D0-D3 分级 → 派 worker / 推 spec / 推审批卡片 → ClawTeam LLM → 11 checks + JSON Gate → Codex+Gemini review → 全绿 APPROVED → P1-2 auto-merge → deploy.yml → 飞书双通道战报。

## 经验教训

- L2 事实源（GitHub Checks 唯一真相）是对抗 LLM 幻觉的核心机制，不依赖任何层的口头声明。
- L3 dispatcher 刻意设计为「纯 Python 无 LLM」，保证调度层确定性，不受 LLM 概率影响。
- L7 固定 3 轮上限是防 LLM 错误路径无限循环的铁律，超出自动升级人工。
