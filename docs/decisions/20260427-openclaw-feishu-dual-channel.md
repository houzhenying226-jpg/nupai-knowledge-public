# 飞书双向通路架构 K2 + K7 + K12

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

振英的核心诉求是从「命令行搬运工」变为「飞书指挥官」。飞书在 OpenClaw 中承担双角色：命令机器人是指挥层（振英发命令），心跳告警是监控层（所有人看进度）。

## 正文

## 背景

振英的核心诉求是从「命令行搬运工」变为「飞书指挥官」。飞书在 OpenClaw 中承担双角色：命令机器人是指挥层（振英发命令），心跳告警是监控层（所有人看进度）。

## 架构设计

| 通道 | 类型 | 用途 | 实现 |
|---|---|---|---|
| 命令机器人 DM（K2） | 双向 WebSocket | 振英发命令 + 收战报 + D3 审批 | App ID cli_a93a858dd3389e19 / port 18790 |
| 心跳告警群（K4） | 单向 webhook | 进度通知 / 异常告警 | larksuite open-apis/bot/v2/hook/ |
| D3 审批卡片（K7） | 交互卡片 | 高风险操作人工审批 | GitHub Deployments SoT |
| 群聊 @OpenClaw（K12） | 批量派工 | 批量派多个 Issue 给不同 LLM | 群 chat_id 白名单 |

命令机器人 DM 支持的命令：`/oc status`、`/oc 修 NUP-XXX`、`/oc deploy plan/approve/rollback`、`/oc 审计 sprint-12`，以及 K12 批量派工如 `派 NUP-321/322/323`。

权限三层校验：open_id 白名单（仅振英+James）+ chat_id 作用域 + verb 白名单。陌生人发命令直接忽略。

## 经验教训

- 飞书消息开头必须含「OpenClaw」关键字，否则触发 19024 错误（飞书关键词过滤）。feishu_send() 已加自动前缀重试机制。
- P2P DM 的 chat_id 不是标准飞书 chat_id，而是对方 open_id（ou_xxx 格式），需要 frozenset 双格式白名单。
- 双通道设计（主动 DM + 被动群通知）使振英无需主动查询状态，异常会自动推送。
