---
type: decisions
title: 三方AI交叉验证铁律
date: 2026-05-03
risk_level: L4
project: 全局
tags:
  - cross-check
  - ai-review
  - incident
---

# 三方AI交叉验证铁律

## 背景

2026-05-02 23:00 → 5-3 12:00，13 小时真账翻车 6 次：

- Phase 1 plist 真账作假
- Phase 5 allowlist 拒绝
- dispatcher 反复 SIGTERM 错追根因
- PR #431 假以为生效
- verb 中文 `修` 缺失
- worker.sh dead code

每次靠单家 AI 凭直觉猜根因都跑偏；每次让 GPT-5.5 + Gemini 3.1 + Claude Opus 4.7 三家深读 GitHub 共识才定真根因。

## 必上三方 AI 的场景

- 真账翻车 ≥ 1 次（同一会话内同类问题）。
- 涉及多层系统：launchd + Python + Git + 飞书 + GitHub Actions。
- 交互涉及“已生效”假账嫌疑：PR merge / health 200 / 200 OK。
- 涉及静默失败：没回执 / 没 log / 沉默无响应。
- 配置数据迁移后某层断掉。

## 三家 AI 选择

- GPT-5.5 Thinking：Codex 同源，看代码细节最仔细，擅长 bash 子 shell / 类型注解陷阱。
- Gemini 3.1 Pro Thinking：大上下文，擅长跨文件追全链路 + 识别 fail-fast 模式。
- Claude Opus 4.7 Thinking：擅长架构层根因，经常给“先取证再判断”路径。

## 被否决方案：凭单家 AI 直觉

单家结论可能 stylistic 跑偏：Gemini 偏激进 kill，GPT 偏短路径，Claude 偏证据保守。
跨家共识才是真根因，分歧点只作为次要决策。

## 提问模板

1. 30 秒项目背景：项目目标 + 真账闭环目标。
2. 真账历史：以前翻车 N 次 + 修过什么。
3. 当前真账症状 + 真原文证据：命令 + 输出。
4. 假说候选 A/B/C，让它判优先级。
5. 要求深读 GitHub 链接，具体到文件 + 行号 + Spec。
6. 要求“只改一处”最短真账修复路径。
7. 禁止凭推断，必须源码引用。

## 结论

真账翻车 ≥ 1 次后，凭“看像”猜的真账权重归零。
三方 AI 深读 GitHub 共识 = 唯一真账可靠源。

今晚救场 ≥ 4 次：dispatcher 反复 SIGTERM 真根因 / PR #431 磁盘没 pull / verb 缺中文 / worker.sh 空 PR 号根因。

## 下次注意

真账翻车 ≥ 1 次 → 立刻三方 AI，不再凭经验给指令。
给三方 AI 的提问必须要求源码原文 + 行号 + 具体 diff，禁止“先看 log 再说”推迟答案。
