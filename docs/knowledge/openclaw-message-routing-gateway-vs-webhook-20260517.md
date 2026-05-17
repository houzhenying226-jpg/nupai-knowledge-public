# OpenClaw 消息路径：自然语言走 Gateway，/oc 命令走 webhook_server

> 类型：知识条目 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-17T08:41:52Z

## 摘要

飞书自然语言消息由 Gateway AI agent 直接处理，不经过 webhook_server.py；只有 /oc 命令才由 gateway 转发到
  webhook_server:18790。

  ---

## 正文

> 记录日期：2026-05-17  风险等级：L2

## 背景
2026-05-17 故障排查中，Claude 错误认为自然语言消息经过 feishu_bot.py → webhook_server.py → UNKNOWN verb
  路径，花大量时间给 webhook_server 加 NLP fallback，实际上该路径对自然语言根本不适用。

## 内容
实际消息流：

  飞书自然语言（"你好"、"三方AI对比"）：
  飞书 → Gateway（日志在 ~/.openclaw/logs/gateway.log）→ agent:main AI 直接处理 → 回复飞书

  飞书 /oc 命令（"/oc status"、"修 NUP-123"）：
  飞书 → Gateway → webhook_server:18790/command → command_parser → _dispatch → 回复飞书

  KNOWLEDGE_CARD 块： 
  飞书 → Gateway → webhook_server:18790 → _dispatch_inner 拦截 → knowledge_writer.py → 入库

  验证命令：tail -f ~/.openclaw/logs/gateway.log | grep -E "received message|dispatch complete"
  replies=0 才是真正无回复；replies=1 说明 gateway 已回复，问题在飞书消息发送层。

## 结论
自然语言静默问题查 gateway.log，不查 webhook_server.py。webhook_server 只负责 /oc 命令和 KNOWLEDGE_CARD 入库。

## 影响
误判路径导致修了不相关的 webhook_server UNKNOWN 路由，耽误约 1 小时。

## 下次注意
飞书 bot 不回复自然语言 → 先 tail gateway.log 确认消息是否到达并被 dispatch，再看 replies 是否为
  0。不要直接跳去看 webhook_server.py。

