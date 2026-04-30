# OpenClaw 三铁律

> 类型：架构决策 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 多 LLM 平台需要系统性机制对抗三类根本风险：LLM 幻觉声称完成、LLM 自由发挥超出边界、LLM 错误路径无限循环。三铁律是整个系统的架构原则，Sprint12 完整落地。

## 正文

## 背景

OpenClaw 多 LLM 平台需要系统性机制对抗三类根本风险：LLM 幻觉声称完成、LLM 自由发挥超出边界、LLM 错误路径无限循环。三铁律是整个系统的架构原则，Sprint-12 完整落地。

## 三铁律

**铁律 1：GitHub PR/Checks 是事实源 — 对抗 LLM 幻觉**

任何 LLM 不得口头宣布"任务完成"，唯一判断依据是 GitHub PR 状态和 CI Checks 结果。LLM 说完成 ≠ 完成，绿灯才算完成。凭据：branch protection 11 checks。

**铁律 2：任何 LLM 决策必须有可证伪硬闸门 — 对抗 LLM 自由发挥**

所有决策节点必须有机器可验证标准，不依赖 LLM 概率推断。AI Review Gate 要求 JSON Schema 输出而非自然语言描述；before_dispatch hook 用正则前缀匹配而非 LLM 判断。

**铁律 3：反馈循环必有上限（默认 3）— 对抗 LLM 错误路径无限循环**

任何 fix-review 循环最多 3 次（T16 机制），超出自动打 escalated label 并飞书 @ 振英人工介入。凭据：dispatch_handler.py:1266-1349。

## 经验教训

- 三铁律是架构原则而非规则清单：每一层都必须独立可证伪，不依赖上层"口头保证"。
- before_dispatch 用正则（可证伪）代替 LLM 判断，是铁律 2 的直接体现。
- 没有铁律 3 时，Claude Code 曾重复检查同一文件 8 轮无法停止，证明上限机制不可省略。
